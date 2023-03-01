
https://williamyaoh.com/posts/2023-02-28-writing-servant-combinators.html


If you’ve used Servant for anything more than basic APIs, you’ve probably run into situations where it seems like Servant should be able to handle boilerplate-y functionality for you. Say, automatically adding HTTP headers to outgoing responses, or validating CSRF tokens. Technically Servant gives you the functionality you’d need for these, since you could add a Header parameter on each endpoint to ingest CSRF tokens, and similarly wrap each response in a Headers to modify the HTTP response. But even if you can factor out the repeated header mentions at the type level, it’s incredibly tedious to return response headers in every single handler body – when the whole point of Servant is to save us a bunch of parsing and validation boilerplate.

Since Servant produces WAI Applications, you could also handle these sorts of needs by applying WAI Middleware at the toplevel of your executable; for instance, the wai-extra package provides prebuilt middleware for things like logging requests and blocking requests from public IP addresses. But this doesn’t feel like a particularly Servant-y solution, since WAI middleware has to be applied across your entire application. What you’d really prefer is to specify the middleware inside your Servant API type, marking which endpoints need the middleware applied and which ones don’t; you can sort of do this with WAI middleware through string prefix matching on the request path, but again, if we wanted to use that sort of stringly-typed, seat-of-your-pants implementation, we wouldn’t be using Servant, would we?

The ideal solution would be something like the following, where the same way we specify our endpoint’s query parameters, request body, and so on inside the API type, we would be able to do the same for any middleware we want to apply below that path.

type API =
  "api" :> "frobwozzle" :> AddExtraHeaders
    :> QueryParam "kerjigger" Bool :> Get '[JSON] [Frobwozzle]
It turns out that these LEGO-like typed API components, also known as Servant combinators, aren’t that difficult to implement ourselves. For instance, here’s a basic combinator that logs “Hello, Servant!” to stdout on every request.

{-# LANGUAGE DataKinds             #-}
{-# LANGUAGE FlexibleInstances     #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE ScopedTypeVariables   #-}
{-# LANGUAGE TypeApplications      #-}
{-# LANGUAGE TypeFamilies          #-}
{-# LANGUAGE TypeOperators         #-}

import Data.Functor ( (<&>) )
import Servant

data HelloServant

instance HasServer api ctx => HasServer (HelloServant :> api) ctx where
  type ServerT (HelloServant :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server = route (Proxy @api) ctx server <&> \app req respK -> do
    putStrLn "Hello, Servant!"
    app req respK

type TestAPI =
       "nolog" :> GetNoContent
  :<|> "log" :> HelloServant :> GetNoContent
Note that I’m going to be assuming you already know how to handle yourself around type-level metaprogramming. While a fairly deep subject, it’s out of scope here since it would make the post far too long. For the purposes of this post we’ll only be focusing on the specifics of working with Servant’s internals. Alexis King has written a good introduction to the topic of type-level metaprogramming if you need a primer.

All the code in this post, along with some simple unit tests, is available on Gitlab if you’d like to follow along. Feel free to use the code as a reference, as a template for your own combinators, or for whatever you want, really.

With that preliminary stuff out of the way, let’s dive into it. We’ll start from the bottom and break down how Servant internally implements these combinators, and work up from there to build our own convenient combinators for accessing the raw request in our server, automatically adding response headers, parsing cookies, and a simple example of request size limiting.

HasServer, routing, and how Servant generates your app
The heart of servant-server is the HasServer typeclass, which contains all the functions needed for Servant to convert your API type + handler functions into a runnable application.

class HasServer api context where
  type ServerT api m

  route
    :: Proxy api
    -> Context context
    -> Delayed env (Server api)
    -> Router env

  hoistServerWithContext
    :: Proxy api
    -> Proxy context
    -> (forall x. m x -> n x)
    -> ServerT api m
    -> ServerT api n
There’s a lot to unpack here. First of all, the two type parameters:

api refers to the API type itself. So an instance like HasServer GetNoContent context means that if you define a type like type API = GetNoContent, you can write a handler for it and Servant will allow you to generate an application from the type + handler. You’ll most often see instances like HasServer api context => HasServer (QueryParam ... :> api) context, making use of the API type’s recursive structure.
context is a type-level list of types. It’s used to pass arguments to combinators at runtime; since combinators aren’t like normal functions, we can’t just pass in different parameters, but we might still want ways to vary their behavior. For instance, your combinator might need some private key data to validate incoming headers, so the context might look like '[PEM]. Don’t worry if this doesn’t make much sense for now, we’ll get into it in more detail later.
What about the things inside the typeclass?

type ServerT is an associated type family. It’s how you define what handler type matches your combinator. For instance, for QueryParam "foo" Int :> api, the type of ServerT would be Maybe Int -> ServerT api m. We’ll see how we can use this in a later section.
route is the function Servant actually calls to generate your application.
hoistServerWithContext is how Servant allows you to write your handlers in different monad stacks. In simple server implementations, you can get away with writing your handlers in Servant’s Handler monad, which is just a thin layer over IO. More complicated servers will need an actual monad stack; at least some kind of ReaderT for storing configuration.1 If you look at the type signature, you can see it’s simply changing the monad the server is running in. We’ll also see how we can use this function to provide extra information in the context.
For now, let’s focus specifically on the route function, since it’s how Servant actually generates your web application, and also where the bulk of the work goes when implementing a combinator.

Given its name, you might reasonably assume that the route function gets called on every incoming request to, well, route it. That isn’t what it does. Instead, route is called exactly once, as part of the startup of your server.2 It takes in the context and the handler for your whole server, and returns a Router. What’s a Router? Well, let’s look at the types:

route
  :: Proxy api
  -> Context context  -- context, i.e. arguments to the combinator
  -> Delayed env (Server api)  -- your handler function, essentially
  -> Router env  -- the generated application

type Router env = Router' env RoutingApplication

data Router' env a
  = StaticRouter  (Map Text (Router' env a)) [env -> a]
  | CaptureRouter [CaptureHint] (Router' (Text, env) a)
  | CaptureAllRouter [CaptureHint] (Router' ([Text], env) a)
  | RawRouter     (env -> a)
  | Choice        (Router' env a) (Router' env a)
  deriving Functor

type RoutingApplication =
     Request
  -> (RouteResult Response -> IO ResponseReceived)
  -> IO ResponseReceived

-- ^^ notice the similarity to a WAI Application
-- type Application =
--   Request -> (Response -> IO ResponseReceived) -> IO ResponseReceived
We’re going to gloss over the specifics of the Router' type, mostly because they’re not that important for what we want to do. More useful for us is the RoutingApplication type; this is the function that actually gets called with each request! You can think of Router as a tree of path information, with RoutingApplications at the leaves which receive the requests for each specific path and produce responses. This leads us to some useful observations:

RoutingApplications are extremely flexible, because they have all the information about the request, and can do whatever they want to modify the response.
When we implement route as part of the HasServer instance for our own combinator, inside the body of our implementation we get back a Router from a recursive call to route, containing the application for our sub-API.
Router' implements Functor; the router we got back contains RoutingApplications, and we can modify them by mapping!
The general outline we’ll follow in our instances, then, is to call route recursively, then augment the returned application with whatever code we need, whether that’s cookie parsing, extra response headers, whatever. Just by mapping over RoutingApplication, you have a lot of power at your fingertips. You have full access to both the request and response. You have access to IO as well, in case you need any side effects. This approach is simple, and handles most of your implementation needs when writing combinators.

Example: Returning a header with a “replay” path
Say we want every response to include an X-Replay-Path header, whose value is the full path and query parameters used to make the request. For instance, perhaps we want to make it easy for our frontend code to repeat search requests; maybe our frontend is running a lot of async requests and needs to know where a specific result came from.

First, let’s define our combinator type and write the simpler parts of our HasServer instance.

{-# LANGUAGE FlexibleInstances         #-}
{-# LANGUAGE MultiParamTypeClasses     #-}
{-# LANGUAGE OverloadedStrings         #-}
{-# LANGUAGE ScopedTypeVariables       #-}
{-# LANGUAGE TypeApplications          #-}
{-# LANGUAGE TypeFamilies              #-}
{-# LANGUAGE TypeOperators             #-}

import Data.Functor ( (<&>) )
import Network.Wai ( rawPathInfo, rawQueryString, mapResponseHeaders )
import Servant

data Replay

instance HasServer api ctx => HasServer (Replay :> api) ctx where
  type ServerT (Replay :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server
This should all look like fairly standard recursive typeclass code. Since we don’t want to pass any extra arguments to any implementing handlers, we define our ServerT type as just being the same type as that of our sub-API. And since the ServerT is the same, we can also define hoistServerWithContext by just calling it for our sub-API as well. Now we just need to implement route and modify the application to add the header.

  route _ ctx server = route (Proxy @api) ctx server <&> \app req respK -> do
    ...
We make our recursive route call to get a Router, then fmap over its contained RoutingApplication. Note that the type of the function we pass to fmap needs to be RoutingApplication -> RoutingApplication, or if we expand the type alias:

   (Request
     -> (RouteResult Response -> IO ResponseReceived)
     -> IO ResponseReceived)
-> Request
-> (RouteResult Response -> IO ResponseReceived)
-> IO ResponseReceived
Hence why our mapping function takes in 3 arguments instead of 1: the sub-API application, the raw request, and the response continuation. You can use the same trick if you ever need to write WAI Middleware. I also prefer using (<&>) (flipped fmap) when writing combinators, since the body of the mapping function tends to be the largest part of the implementation.

Now all we have to do is pull the path and query info from the incoming request and add it to the response. We don’t have the response itself as one of our arguments, but we do have a response continuation, which is just as good; we compose a function to add our header with the continuation, pass the updated continuation into the sub-API app, and off we go. The full HasServer instance follows.

instance HasServer api ctx => HasServer (Replay :> api) ctx where
  type ServerT (Replay :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server = route (Proxy @api) ctx server <&> \app req respK -> do
    let replayPath = rawPathInfo req <> rawQueryString req
    let respK' = respK . fmap (mapResponseHeaders (("X-Replay-Path", replayPath) :))
    app req respK'
For the sake of brevity, I’m going to omit showing the result of making HTTP requests against a server using the example combinators shown in this post. The source code contains ready-to-use test APIs and servers, as well as unit tests showing their usage and expected behavior. Go look there if you’d like to confirm for yourself that these combinators do what I say they do.

ServerT, DelayedIO, and passing extra arguments to your handlers
Augmenting the RoutingApplication like this is very useful, but you might have noticed that when you go to write server handlers, they’re generally a bit more strictly typed than raw Requests and Responses. Servant types your query params, your request bodies, your endpoint returns, and we haven’t mentioned any of that so far. How would we go about implementing something like the QueryParam combinator, where adding it into our API type means our handlers get passed extra information? Say we wanted to automatically parse the Cookie header and pass the cookies into our handlers; how would we go about writing that?

As we saw previously, the magic here is in the ServerT associated type family, which defines what the type of an implementing handler for a given combinator is. A useful trick is that we can ask GHCi to tell us what the “value” of ServerT is for a given API type, using the :kind! command:

λ> :kind! forall m. ServerT (Get '[JSON] [Int]) m
forall m. ServerT (Get '[JSON] [Int]) m :: *
= m [Int]

λ> :kind! forall m. ServerT (QueryParam "foo" String :> Get '[JSON] [Int]) m
forall m. ServerT (QueryParam "foo" String :> Get '[JSON] [Int]) m :: *
= Maybe [Char] -> m [Int]
So, if we want our own combinators to be able to pass information into our handlers, we should start by defining our ServerT appropriately:

data WithCookies

instance HasServer api ctx => HasServer (WithCookies :> api) ctx where
  type ServerT (WithCookies :> api) m = Map Text Text -> ServerT api m
Of course, it’s not that easy, as now both our definitions of hoistServerWithContext and route need to change appropriately to deal with the new argument. hoistServerWithContext is fairly simple, as it’s only concerned with running a natural transformation. route is a bit harder, as it actually needs to pass in the value of the argument. And to do that, we need to talk about a Servant-internal data structure called Delayed.

What is a Delayed? Defined in Servant.Server.Internal.Delayed, it’s essentially a wrapper around some value that also contains a bunch of validations and parsers on the HTTP request.

data Delayed env c where
  Delayed ::
    { capturesD :: env -> DelayedIO captures
    , methodD   :: DelayedIO ()
    , authD     :: DelayedIO auth
    , acceptD   :: DelayedIO ()
    , contentD  :: DelayedIO contentType
    , paramsD   :: DelayedIO params
    , headersD  :: DelayedIO headers
    , bodyD     :: contentType -> DelayedIO body
    , serverD   :: captures
                -> params
                -> headers
                -> auth
                -> body
                -> Request
                -> RouteResult c
    } -> Delayed env c
Don’t worry too much about specifics here, the general idea is more important. Each of the fields other than serverD is essentially an effectful validation on the incoming request. The reason for separating them like this is so that they can get run in a specific order, with the least expensive checks happening first. You can read the details in the documentation of Delayed.

Why is this necessary? Say our endpoint type contains a ReqBody '[JSON] Frobwizzle. This needs to run two specific checks on an incoming request: that the Content-Type header is application/json, and that the body correctly parses as a Frobwizzle. It’s valuable to have the header check happen first, since it requires no parsing and we can potentially reject bad requests much faster. But other combinators might also want to add a header check; if our endpoint type ends in Post '[JSON] [Int], then the Post would also want to check that the Accept header is application/json3 — and we would want to ensure that both the header check from the ReqBody and from the Post happen before the body parse. Having a separated data structure like this ensures each combinator can specify its own independent checks, while still having those checks interleaved properly with the validations performed by other combinators.

How do we actually use this? If we look at the type signature of route again, we can see that the third argument is a Delayed env (Server api), which we know is just our handler function wrapped in a Delayed (since Server api is just an alias for ServerT api Handler):

route :: Proxy api -> Context context -> Delayed env (Server api) -> Router env
Conveniently, the Servant.Server.Internal.Delayed module provides a family of functions for adding checks to a Delayed, like addHeaderCheck:

addHeaderCheck :: Delayed env (a -> b) -> DelayedIO a -> Delayed env b
DelayedIO is an IO that can read the incoming request, and has the ability to signal an error by failing routing. As you can see from the signature of addHeaderCheck, by writing a DelayedIO that produces the value we want, we can get Servant to pass that value to the handler function inside Delayed on each request, meaning we can register not only validations, but parsers that pull data from the incoming request and pass them along. Our definition of route, then, will modify the Delayed that gets passed in, then pass the modified Delayed to the recursive call, letting us provide the desired information to the handler.

Example: Getting the raw request information
It’s often useful to be able to examine the raw request in your Servant handlers, but as far as I know Servant doesn’t provide the ability to do this out-of-the-box.

Up to our definition of ServerT is as expected, where the ServerT type gains an argument for the Request:

{-# LANGUAGE FlexibleInstances     #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE ScopedTypeVariables   #-}
{-# LANGUAGE TypeApplications      #-}
{-# LANGUAGE TypeFamilies          #-}
{-# LANGUAGE TypeOperators         #-}

import Control.Monad.IO.Class ( liftIO )
import Network.Wai ( Request )
import Servant

data WithRawRequest

instance HasServer api ctx => HasServer (WithRawRequest :> api) ctx where
  type ServerT (WithRawRequest :> api) m = Request -> ServerT api m

  ...
hoistServerWithContext is slightly different, since now the type of ServerT for our combinator is different from our sub-API:

  ...

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt . server

  ...
Finally, we can use passToServer to directly pass in the request to the handler that’s inside the Delayed, without adding a validation:

import Servant.Server.Internal.Delayed ( passToServer )

  ...

  route _ ctx server = route (Proxy @api) ctx $
    passToServer server id
And that’s it. It’s such an easy implementation that I’m surprised it’s not provided as part of the servant or servant-server packages. Here’s the full instance:

instance HasServer api ctx => HasServer (WithRawRequest :> api) ctx where
  type ServerT (WithRawRequest :> api) m = Request -> ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt . server

  route _ ctx server = route (Proxy @api) ctx $
    passToServer server id
Example: Parsing cookies
Browsers send cookies on each request with the Cookie header as a list of KEY=VAL separated by semicolons. We want to parse this into a map of cookie names to values and pass that to our handlers.

For this we’ll use the parseCookies from the cookie package. The implementation isn’t that different from the raw request combinator, since it doesn’t need to do any validation; if the Cookie header is missing or parseCookies returns no cookies, we pass in an empty map. The full implementation is below.

{-# LANGUAGE FlexibleInstances     #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE ScopedTypeVariables   #-}
{-# LANGUAGE TypeApplications      #-}
{-# LANGUAGE TypeFamilies          #-}
{-# LANGUAGE TypeOperators         #-}

import Data.ByteString ( ByteString )
import Data.Map.Strict ( Map )
import Network.HTTP.Types.Header ( hCookie )
import Network.Wai ( requestHeaders )
import Servant
import Servant.Server.Internal.Delayed ( passToServer )
import Web.Cookie ( parseCookies )

import qualified Data.Map.Strict as Map

type Cookies = Map ByteString ByteString

data WithCookies

instance HasServer api ctx => HasServer (WithCookies :> api) ctx where
  type ServerT (WithCookies :> api) m = Cookies -> ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt . server

  route _ ctx server = route (Proxy @api) ctx $
    server `passToServer` \req ->
      let mcookie = lookup hCookie (requestHeaders req)
      in maybe Map.empty (Map.fromList . parseCookies) mcookie
Vault, contexts, and communicating between combinators
While we’ve seen how we can get our combinators to communicate with our handlers, we haven’t seen how to get combinators to talk to each other. An example might be a combinator that limits incoming request size: if multiple such combinators are included in a given API path, we’d want the innermost one’s limit to apply, which means we’d need some way for the combinator to disable or overwrite the checks added by other combinators. Another use case would be an improvement to our cookie parsing combinator from earlier. That combinator provided cookies to the handler, but it’d also be nice if it provided the cookies to other combinators; we might have a lot of different combinators checking their specific cookies, and it’s inefficient to have each such combinator parse the cookies itself. Wouldn’t it be nice if our combinators could pass values to each other?

Sadly, there’s no direct way to do this. However, using some tools that Servant and WAI provide, as well as a bit of ingenuity on our part, we can pull off some clever things indeed.

Firstly, let’s take a second look at Servant “contexts.” We skimmed over these briefly when we looked at the HasServer typeclass, but not much more than that. In short, they’re a way to provide extra values to your combinators when you initialize your server. What does that mean? Say you have a combinator that blocks incoming IPs based on a blocklist. This combinator needs to be provided with a blocklist in order to function, but you can’t just pass such a list in as a parameter; combinators are not normal functions. However, what you can do is launch the server using serveWithContext instead of the usual serve, and provide the list of blocked IPs that way:

import Servant

blockedIPs :: [IP]
blockedIPs = [ ..., ..., ... ]

myServer :: Server MyAPI
myServer = ...

main :: IO ()
main = do
  ...
  run 8080 $ serveWithContext
    (Proxy @MyAPI)
    (blockedIPs :. EmptyContext)
    myServer
Then, in the definition of your IP blocking combinator, you can access that information through a HasContextEntry constraint:

data BlockIPs

instance
  ( HasServer api ctx
  , HasContextEntry ctx [IP]
  )
  => HasServer (BlockIPs :> api) where
  ...

  route _ ctx server =
    let blockedIPs = getContextEntry ctx :: [IP]
    in ...
It’s essentially a typed, heterogeneous list of data that all combinators can access. Note that the only way to lookup values is by their type, so you cannot have multiple values of the same type in your context, at least not if you expect to be able to access all of them. My recommendation is to wrap anything you want to put in the context in a newtype, or create a new datatype for them, unless you’re absolutely sure the types won’t conflict.

Since hoistServerWithContext and route have to take in a Proxy of the context’s type and the context value, respectively, it’s also possible for individual combinators to modify the context for its sub-API, usually by adding new elements. You might think that this gives us the ability to pass values between combinators. This is kind of true, but remember that route only gets called once, at the start of your server, so the context cannot hold values that depend on what’s actually in a request, like cookies. For that we’ll have to turn to Vaults.

A Vault, which is provided by the eponymous vault package, is a heterogeneous map which can store values of any (boxed) type. The catch is that you can’t just use anything as a key; you have to generate a vault Key with the newKey function, in our case in IO.

Every WAI Request contains a Vault. Since combinators have access to the Request through the ways we’ve already looked at, we can use that Vault to store and update shared values for our combinators or otherwise cache information we parsed from the request. There’s just one hitch, which is that if multiple combinators want to access the same shared value, they’ll all need the same vault Key. Thankfully that’s easy enough if we make use of Servant contexts: in our server initialisation code, we generate a Key, then pass it to our combinators in the context, and now our combinators can share values that depend on the request. Neither contexts nor Vault alone was enough to get us what we needed, but together they’re enough to allow combinators to communicate about individual requests.

One crucial note about the order of execution. This is easier to show with an example. Say we have the following two combinators and their HasServer instances:

data A
data B

instance HasServer api ctx => HasServer (A :> api) ctx where
  type ServerT (A :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server =
    let withCheck = addMethodCheck server $ do
          -- C1
          pure ()
    in route (Proxy @api) ctx withCheck <&> \app req respK -> do
         -- A1
         app req respK
         -- A2

instance HasServer api ctx => HasServer (B :> api) ctx where
  type ServerT (B :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server = route (Proxy @api) ctx server <&> \app req respK -> do
    -- B1
    app req respK
    -- B2
Given an endpoint defined like type API = A :> B :> "endpoint" :> GetNoContent, when a request comes into this endpoint, the server will execute the code added by the combinators in the order A1 > B1 > C1 > A2 > B2. Note that the code C1 that the A combinator added to the Delayed got executed after both A1 and B1; it will also see any changes that either A1 or B1 make to the Request. This is especially important if you want to take data that’s been cached in the request’s Vault and pass it to a handler function.

Example: Improving our cookie combinator to cache the parsed cookies
As mentioned, we’d prefer only ever parsing the cookies once per request. If you’re crazy enough to use Servant for HTML pages and not just JSON APIs, you might end up with lots of different cookies that your site needs to care about. Maybe one combinator will handle validating the session cookie, while another deals with a cookie that stores low-stakes user preferences like dark/light mode. Having both of these parse the whole cookie map would be a waste of cycles.

You know the drill, here’s a bunch of preamble for language pragmas and imports:

{-# OPTIONS -Wno-redundant-constraints #-}
{-# LANGUAGE DataKinds                 #-}
{-# LANGUAGE FlexibleContexts          #-}
{-# LANGUAGE FlexibleInstances         #-}
{-# LANGUAGE MultiParamTypeClasses     #-}
{-# LANGUAGE OverloadedStrings         #-}
{-# LANGUAGE ScopedTypeVariables       #-}
{-# LANGUAGE TypeApplications          #-}
{-# LANGUAGE TypeFamilies              #-}
{-# LANGUAGE TypeOperators             #-}

import Data.ByteString ( ByteString )
import Data.Functor ( (<&>) )
import Data.Map.Strict ( Map )
import Network.HTTP.Types.Header ( hCookie )
import Network.Wai ( requestHeaders, vault )
import Servant
import Servant.Server.Internal.Delayed ( addAcceptCheck, addHeaderCheck )
import Servant.Server.Internal.DelayedIO ( DelayedIO, withRequest, delayedFailFatal )
import Web.Cookie ( parseCookies )

import qualified Data.Map.Strict as Map
import qualified Data.Vault.Lazy as Vault
First is our ProvideCookies combinator to parse the cookies and cache them in the request Vault. This looks is pretty the similar to the definition for our previous WithCookies combinator, except that we need an extra constraint to make sure we have a vault Key available. We also put in another small touch and add a HasCookies type into the context for our sub-API. Since the parsing of the cookies and their usage has been split into separate combinators, we want to make sure any combinators that use the cookies won’t compile when placed in an API unless there’s also a ProvideCookies to parse them.

type Cookies = Map ByteString ByteString

data ProvideCookies

data HasCookies = HasCookies

instance
  ( HasServer api (HasCookies ': ctx)
  , HasContextEntry ctx (Vault.Key Cookies)
  )
  => HasServer (ProvideCookies :> api) ctx where
  type ServerT (ProvideCookies :> api) m = ServerT api m

  hoistServerWithContext _ _ nt server =
    hoistServerWithContext (Proxy @api) (Proxy @(HasCookies ': ctx)) nt server

  route _ ctx server =
    route (Proxy @api) (HasCookies :. ctx) server <&> \app req respK -> do
      let
        mcookie = lookup hCookie (requestHeaders req)
        cookies = maybe Map.empty (Map.fromList . parseCookies) mcookie
        key = getContextEntry ctx :: Vault.Key Cookies
        req' = req { vault = Vault.insert key cookies (vault req) }
      app req' respK
Note how we modify the context in our recursive calls to both hoistServerWithContext and route. We pull the vault Key from the context and use it to store the parsed cookies in the request Vault, then pass the modified request to our sub-API application.

With the cookies already cached in the request, all we have to do is write a separate WithCookies combinator that pulls the cached data out of the request and passes it to the handler function. Again, since Delayed runs just before the handler function gets called, we’ll be able to see the Vault modification made by ProvideCookies.

data WithCookies

instance
  ( HasServer api ctx
  , HasContextEntry ctx HasCookies
  , HasContextEntry ctx (Vault.Key Cookies)
  )
  => HasServer (WithCookies :> api) ctx where
  type ServerT (WithCookies :> api) m = Cookies -> ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt . server

  route _ ctx server =
    route (Proxy @api) ctx $
      server `addHeaderCheck` retrieveCookies
    where
      retrieveCookies :: DelayedIO Cookies
      retrieveCookies = withRequest $ \req -> do
        let key = getContextEntry ctx :: Vault.Key Cookies
        case Vault.lookup key (vault req) of
          Just cookies -> pure cookies
          Nothing -> delayedFailFatal $ err500
            { errBody = "Something has gone horribly wrong; could not find cached cookies." }
As long as the server initialisation adds a vault Key to the context, we get convenient cookie parsing that also caches the data.

Note that in the instance constraint we need two context entries: one for the vault Key, and another for the HasCookies type, to make sure that the cookies have been cached. This isn’t strictly necessary; another way to design this would be to only have WithCookies, and have it check whether the Vault contains the cookie data or not, only doing the parsing and caching when the data is missing.

Having this caching isn’t that useful if all you’re doing is passing the cookies along to your handlers, but it’s worth it once you have mulitple combinators handling different cookies; at that point it also makes sense to have just one combinator responsible for caching, rather than needing to put parsing/caching code in every cookie-related combinator.

Example: simple request size limiting, overwriting for individual routes
Every server should limit the size of incoming requests; while not sufficient, it’s a useful first line of defense against DoS attacks.

Concretely, what we’d like is a combinator RequestSizeLimit, where the limit can be specified as type-level arguments to the combinator.

data StorageUnit = B | KB | MB | GB

-- we can specify a limit like RequestSizeLimit 4 'MB
data RequestSizeLimit (amt :: Natural) (unit :: StorageUnit)
If multiple size limits are specified on the same path, we’d like the innermost such size limit to override. For instance, we might set a relatively small default limit across our entire API, but certain specific upload endpoints might need a large, multi-GB limit. This means we can’t naively include a check inside the definition of RequestSizeLimit, because if it was used multiple times in a path and each combinator ran its check independently, the net result would be that only the smallest specified limit would apply, even if our intention was to override that limit with something larger.

For the sake of brevity our example here will only check the Content-Length header. Note that that approach wouldn’t work for chunked request bodies; if you need something more robust, take a look at requestSizeCheck from the wai-extra package.

The approach we’ll take is to again make use of Delayed and the fact that it happens after code that combinators add directly to the RoutingApplication. We’ll have one combinator WithSizeCheck which adds a header check to the Delayed, pulling the limit to check against from the request Vault. Then, each RequestSizeLimit will insert its limit into the Vault, overwriting any limit set by a previous RequestSizeLimit on the same path. Since the Delayed header check runs just before calling the handler function, it will only see whatever limit was set by the innermost combinator.

Once again, a preamble:

{-# LANGUAGE AllowAmbiguousTypes       #-}
{-# LANGUAGE DataKinds                 #-}
{-# LANGUAGE FlexibleInstances         #-}
{-# LANGUAGE MultiParamTypeClasses     #-}
{-# LANGUAGE KindSignatures            #-}
{-# LANGUAGE ScopedTypeVariables       #-}
{-# LANGUAGE TypeApplications          #-}
{-# LANGUAGE TypeFamilies              #-}
{-# LANGUAGE TypeOperators             #-}

import Control.Monad ( when )
import Data.Functor ( (<&>) )
import GHC.TypeLits ( KnownNat, Natural, natVal )
import Network.Wai ( RequestBodyLength(..), requestBodyLength, vault )
import Servant
import Servant.Server.Internal.Delayed ( addHeaderCheck )
import Servant.Server.Internal.DelayedIO ( withRequest, delayedFailFatal )

import qualified Data.Vault.Lazy as Vault
We create an extra type StorageUnit such that we can use DataKinds to specify our limit in human-readable units instead of just bytes, as well as get the appropriate number of bytes as a value for a given unit:

data StorageUnit = B | KB | MB | GB

class KnownStorageUnit (unit :: StorageUnit) where
  storageUnitVal :: Int

instance KnownStorageUnit 'B where
  storageUnitVal = 1
instance KnownStorageUnit 'KB where
  storageUnitVal = 1024
instance KnownStorageUnit 'MB where
  storageUnitVal = 1024 * 1024
instance KnownStorageUnit 'GB where
  storageUnitVal = 1024 * 1024 * 1024
With that, the first combinator we’ll write is RequestSizeLimit, which only needs to calculate the byte size limit from the types its been given, and store that limit in the vault. We also create a newtype wrapper around Key Int to avoid any possible type conflicts.

newtype SizeLimitKey = SizeLimitKey { getSizeLimitKey :: Vault.Key Int }

data RequestSizeLimit (amt :: Natural) (unit :: StorageUnit)

instance
  ( HasServer api ctx
  , HasContextEntry ctx SizeLimitKey
  , KnownNat amt
  , KnownStorageUnit unit
  )
  => HasServer (RequestSizeLimit amt unit :> api) ctx where
  type ServerT (RequestSizeLimit amt unit :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server =
    route (Proxy @api) ctx server <&> \app req respK -> do
      let
        limit = fromIntegral (natVal (Proxy @amt)) * storageUnitVal @unit
        SizeLimitKey key = getContextEntry ctx
        req' = req { vault = Vault.insert key limit (vault req) }
      app req' respK
Pretty straightforward; nothing we haven’t seen before.

All that’s left is to implement WithSizeCheck, which actually adds a header check into the Delayed. We use addHeaderCheck to add in a validation, which pulls the stored limit from the Vault, as well as the Content-Length header from the request (using requestBodyLength from WAI), and makes sure the content length is within the specified limit, failing the request with an HTTP 413 (Content Too Large) otherwise. There’s one small hitch, which is that the type signature of addHeaderCheck expects our ServerT type to take an argument, when we don’t actually want to pass any such argument to our handler. We can get around this by wrapping our handler in a function with a dummy () argument; hence the unintuitive need for fmap const server.

data WithSizeCheck

instance
  ( HasServer api ctx
  , HasContextEntry ctx SizeLimitKey
  )
  => HasServer (WithSizeCheck :> api) ctx where
  type ServerT (WithSizeCheck :> api) m = ServerT api m

  hoistServerWithContext _ ctx nt server =
    hoistServerWithContext (Proxy @api) ctx nt server

  route _ ctx server =
    let serverDummyArg = fmap const server
    in route (Proxy @api) ctx $
         addHeaderCheck
           serverDummyArg
           (withRequest $ \req -> do
             let
               SizeLimitKey key = getContextEntry ctx
               mlimit = Vault.lookup key (vault req)
             case (mlimit, requestBodyLength req) of
               (Nothing, _) -> pure ()
               (_, ChunkedBody) -> pure ()
               (Just limit, KnownLength len) ->
                 when (len > fromIntegral limit) $
                   delayedFailFatal err413)
Writing your own combinators is extremely useful and convenient, but up to now there hasn’t been a lot of information about how to go about doing so. The official documentation doesn’t mention it at all, and I haven’t seen much discourse about it either. My hope is that you too can now solve your boilerplate middleware needs in a Servant-y way. In the future I’d even like to see libraries packaging up useful combinators so everyone can benefit, without needing to know to do esoteric type hacking.

This post has only gone over implementing combinators that are useful for server implementations. While we’re not going to go over it here, it’s also possible to do the same and write your own combinators for clients, or other Servant functionality like documentation. For instance, for servant-client you can implement the HasClient typeclass, or HasDocs for servant-docs.

If you missed it, once again here’s the source code for all the combinators shown in this post.

Good luck, and have fun implementing Servant combinators! If you found this helpful, or have more questions about how to bend Servant to your will, shoot me an email!

« Previous post  
Before you close that tab...
Want to become an expert at Haskell, but not sure how? I get it: it's an endless stream of inscrutable concepts and words, as if you've stepped into some strange bizarro world. Where do you even start? Why does any of this matter? How deep do these rabbit holes go?

I want to help. What if you always knew exactly what the next signpost on your journey was, if you always knew exactly what to learn next? That's why I created a Roadmap to Expert for you: a checklist of everything you need to know, sorted by difficulty, broken down into individual, easily-digestible chunks. Best of all: it's free! Just sign up for my email list below.

And there's more where that came from: Only a fraction of what I write ends up on this blog. Sign up and get advice, techniques, and templates for writing real, useful programs, straight to your inbox.

Email address
Absolutely no spam, ever. I respect your email privacy. Unsubscribe anytime.

↥1 However, even if you’re writing your handlers in your own monad stack, Servant still requires a natural transformation into Handler to run them. Because of that I’d recommend using Handler as the base of your monad stack, which will also ensure you have the ability to change the status code of your responses.

↥2 Specifically, when you call a function from the serve family of functions.

↥3 And also that the method of the incoming request is POST, which in the Delayed, happens before both header and body checks.

© Copyright 2019–2023 William Yao
Support me on Ko-fi