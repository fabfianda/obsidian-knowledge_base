ref: https://thma.github.io/posts/2021-01-30-How-QuickCheck-destroyed-my-favourite-theory.html 

by Thomas Mahler

Introduction
Quite a while back I wrote a larger article on the algebraic foundation of software patterns which also covered the MapReduce algorithm.

During the research digged out a paper on algebraic properties of distributed big data analytics, which explained that a MapReduce will always work correctly when the intermediate data structure resulting from the map-phase is a Monoid under the reduce-operation.

For some reason, I was not convinced that this Monoid-condition was enough, because all the typical examples like word-frequency maps are even commutative Monoids under the respective reduce operation.

So I came up with the following personal theory:

Only if the intermediate data structure resulting from the map-phase is a commutative Monoid under the reduce-operation, then a parallel MapReduce will produce correct results.

I tried to validate this property using the QuickCheck test framework.

Interestingly the QuickCheck tests failed! This finally convinced me that my theory was wrong, and after a little deeper thought, I could understand why.

I was impressed with the power of QuickCheck, so I thought it would be a good idea to share this lesson in falsification.

The code shown in this blog is also available on GitHub

Commutative Monoids
In abstract algebra, a monoid is a set equipped with an associative binary operation and an identity element.

The simplest example for a commutative Monoid is 
: the natural numbers under addition with 
 as the identity (or neutral) element. We can use QuickCheck to verify that indeed the Monoid laws plus commutativity are maintained.

If we want to use GHC.Natural type to represent natural numbers, we first have to make Natural instantiate the Arbitrary type class which is used by QuickCheck to automatically generate test data:

import           Test.QuickCheck (Arbitrary, arbitrary, NonNegative (..))
import           GHC.Natural     (Natural, naturalFromInteger)

instance Arbitrary Natural where
  arbitrary = do
    NonNegative nonNegative <- arbitrary
    return $ naturalFromInteger nonNegative
Now we can start to write our property based tests. For algebraic structures it is straightforward to come up with properties: we just write the required laws (associativity, 0 is identity element and commutativity) as properties.

I am using Hspec as a wrapper around QuickCheck as it provides a very nice testing DSL which makes it easy to read the code and the output of the test suite:

import           Test.Hspec

spec :: Spec
spec = do
  describe "The Monoid 'Natural Numbers under Addition'" $ do
    it "is associative" $
      property $ \x y z -> ((x + y) + z) `shouldBe` ((x + (y + z)) :: Natural)
      
    it "has 0 as left and right identity element" $
      property $ \x -> (x + 0 `shouldBe` (x :: Natural)) .&&. (0 + x `shouldBe` x)
      
    it "is commutative" $
      property $ \x y -> x + y `shouldBe` (y + x :: Natural)
The output of these tests will be as follows:

Monoid
  The Monoid 'Natural Numbers under Addition'
    is associative
      +++ OK, passed 100 tests.
    has 0 as identity (or neutral) element
      +++ OK, passed 100 tests.
    is commutative
      +++ OK, passed 100 tests.
So behind the scenes, QuickCheck has generated test data for 100 tests for each property under test. For all these data the test cases passed.

This is definitely not a proof. But it gives us some confidence that our math text-books are correct when giving Natural Numbers under addition as an example for a commutative Monoid.

OK, that was easy! Now let’s move to non-commutative Monoids.

Non-commutative Monoids
Strings (or any other Lists) under concatenation are a typical example. It’s easy to see that "hello" ++ ("dear" ++ "people") equals "(hello" ++ "dear") ++ "people", but that "hello" ++ "world" differs from "world" ++ "hello".

Now let’s try to formalize these intuitions as QuickCheck property based tests again.

First I’m introducing an alias for (++), as it is defined on any list type, it would be required to have type signatures in all properties (as we had all those :: Natural signatures in the examples above). So I define an operation (⊕) which is only defined on String instances:

(⊕) :: String -> String -> String
(⊕) a b = a ++ b
Now we can extend our test suite with the following test cases:

  describe "The Monoid 'Strings under concatenation'" $ do
    
    it "is associative" $ 
      property $ \x y z -> ((x ⊕ y) ⊕ z) `shouldBe` (x ⊕ (y ⊕ z))
      
    it "has \"\" as left and right identity element" $
      property $ \x -> (x ⊕ "" `shouldBe` x) .&&. ("" ⊕ x `shouldBe` x)
The output looks promising:

  The Monoid 'Strings under concatenation'
    is associative
      +++ OK, passed 100 tests.
    has "" as left and right identity element
      +++ OK, passed 100 tests.
Now let’s try to test the non-commutativity:

    it "is NOT commutative" $
      property $ \x y -> x ⊕ y `shouldNotBe` y ⊕ x
But unfortunately the output tells us that this is not true:

    is NOT commutative FAILED [1]

  1) Monoid, The Monoid 'Strings under concatenation', is NOT commutative
       Falsifiable (after 1 test):
         ""
         ""
       not expected: ""
We formulated the property in the wrong way. The (⊕) may be commutative for some edge cases, e.g. when one or both of the arguments are "". But it is not commutative in general – that is for all possible arguments.

We could rephrase this property as “There exists at least one pair of arguments 
 for which 
 is not commutative”:


QuickCheck does not come with a mechanism for existential quantification. But as is has forAll, that is universal quantification. So we can try to make use of the following equivalence:


Unfortunately we can not write this simply as not forAll, as forAll returns a Property but not expects a Bool. But as explained in this discussion on Stackoverflow it is still posible to implement our own exists:

exists :: (Show a, Arbitrary a) => (a -> Bool) -> Property
exists = forSome $ resize 1000 arbitrary

forSome :: (Show a, Testable prop) => Gen a -> (a -> prop) -> Property
forSome gen prop =
  mapResult (\r -> r {P.reason = "No witness found.", P.callbacks = []}) $
    once $ disjoin $ replicate 1000 $ forAll gen prop
Now we can rewrite the property 
 as follows:

    it "is not commutative (via exists)" $
      exists $ \(x,y) -> x ⊕ y /= y ⊕ x
I like how close the Haskell code stays to the concise mathematical formulation! The output of this test fits much better into our intuitive understanding:

    is not commutative (via exists)
      +++ OK, passed 1 test.
Sequential MapReduce
MapReduce is a programming model and an associated implementation for processing and generating large data sets. Users specify a map function that processes a key/value pair to generate a set of intermediate key/value pairs, and a reduce function that merges all intermediate values associated with the same intermediate key.

[This] abstraction is inspired by the map and reduce primitives present in Lisp and many other functional languages. Quoted from Google Research

I’m not going into more details here, as You’ll find detailed information on this approach and a working example in my original article.

Here is the definition of a sequential MapReduce:

simpleMapReduce 
  :: (a -> b)   -- map function
  -> ([b] -> c) -- reduce function
  -> [a]        -- list to map over
  -> c          -- result
simpleMapReduce mapFunc reduceFunc = reduceFunc . map mapFunc
We can test the sequential MapReduce algorithm with the following property based test:

    it "works correctly with a sequential map-reduce" $
      property $ \a b c d -> (simpleMapReduce reverse (foldr (⊕) "") [a,b,c,d]) 
                     `shouldBe` (reverse a) ⊕ (reverse b) ⊕ (reverse c) ⊕ (reverse d)
Excurs: foldMap
What I have shown so far just demonstrates the general mechanism of chaining map and reduce functions without implying any parallel execution. Essentially we are chaining a map with a fold (i.e. reduction) function. In the Haskell base library there is a higher order function foldMap that covers exactly this pattern of chaining. Please note that foldMapdoes only a single traversal of the foldable data structure. It fuses the map and reduce phase into a single one by function composition of mappend and the mapping function f:

-- | Map each element of the structure to a monoid,
-- and combine the results.
foldMap :: (Foldable t, Monoid m) => (a -> m) -> t a -> m
foldMap f = foldr (mappend . f) mempty
Parallel MapReduce
Now we come to the tricky part that kicked off this whole discussion: parallelism.

As an example we consider a simple sequential MapReduce, taking an input list of Ints, computing their squares and computing the sum of these squares:

λ> simpleMapReduce (^2) (foldr (+) 0) [1,2,3,4]
30
Let’s try to design this as a massively parallelized algorithm:

Mapping of (^2) over the input-list [1,2,3,4] would be started in parallel to the reduction of the intermediary list of squares by (foldr (+) 0).

The mapping phase will be executed as a set of parallel computations (one for each element of the input list).

The reduction phase will also be executed as a set of parallel computations (one for each addition).

Of course the reduction phase can begin only when at least one list element is squared. So in effect the mapping process would have to start first. The parallel computation of squares will result in a non-deterministic sequence of computations. In particular it is not guaranteed that all elements of the input list are processed in the original list order. So it might for example happen that 3 is squared first. Now the reduction phase would receive it’s first input 9, and would start reduction, that is compute 9 + 0.

Let’s assume the following random sequence of mapping steps: Next the first element of the input 1, then the fourth 4 and finally the second element 2 would be squared, resulting in a reduction sequence of 4 + 16 + 1 + 9 + 0. As this sums up to 30 everything is fine. Addition is commutative, so changing the sequence of reduction steps does not affect the overall result.

But now imagine we would parallelize:

λ> simpleMapReduce reverse (foldr (⊕) "") [" olleh"," ym"," raed"," sklof"]
"hello my dear folks "
If we assume the same sequence as above, the third element of the input list would be reversed first, resulting in a first reduction step "dear " ⊕ "". Next the first, the fourth and finally the second element would be reversed, resulting in a reduction sequence of "my " ⊕ "folks " ⊕ "hello " ⊕ "dear " ⊕ "" = "my folks hello dear ". As string concatenation is not commutative it does not really come as a surprise that random changes to the reduction sequence will eventually result in wrong computations.

So our conclusion is:

If the MapReduce algorithm is parallelized in the way that I outlined above — which may result in random changes of the order of list elements in the reduction phase — it will only work correct if the intermediary data structure is a commutative Monoid under the reduce operation.

In the following section we will implement a parallel MapReduce in Haskell in order to validate our theory with property based testing.

Parallel MapReduce in Haskell
We can define a parallel MapReduce implementation as follows (for more details see Real World Haskell, Chapter 24):

import           Control.Parallel (par)
import           Control.Parallel.Strategies (using, parMap, rpar)

parMapReduce 
  :: (a -> b)   -- map function
  -> ([b] -> c) -- reduce function
  -> [a]        -- list to map over
  -> c          -- result
parMapReduce mapFunc reduceFunc input =
    mapResult `par` reduceResult
    where mapResult    = parMap rpar mapFunc input
          reduceResult = reduceFunc mapResult `using` rpar
This implementation will start computing mapResult and reduceResult in parallel and finally return reduceResult. The mapResult is computed with a parallelized map function parMap. The reduceResult is computed by applying a parallel reduction strategy rpar.

Next we will write a property based test to valdate our theory:

    it "has some cases where parallel reduction deviates from sequential reduction" $
      exists $ \text -> parMapReduce reverse (foldr (⊕) "") text
                /= simpleMapReduce reverse (foldr (⊕) "") text
But it turns out that QuickCheck does not find any evidence for this assumption:

    has some cases where parallel reduction deviates from sequential reduction FAILED [1]

Failures:

  test\MonoidSpec.hs:69:5: 
  1) Monoid, The Monoid 'Strings under concatenation', has some cases where parallel reduction deviates from sequential reduction
       *** Failed! No witness found. (after 1 test):
After seeing this result I had to deal with some growing cognitive dissonance, much like a flat earther confronted with experimental evidence…

I began verifying my setup. I made sure that the package.yaml contains the right GHC options to provide parallel execution of the test suite:

    ghc-options:
    - -O2
    - -threaded
    - -rtsopts
    - -eventlog
    - -with-rtsopts=-N
I also made sure that all cores of my CPU were actually running at 100% utilization during the parallel tests.

I also inspected the runtime behaviour with the amazing ThreadScope tool (A ThreadScope eventlog can be produced by adding the runtime flags +RTS -ls -N when executing an application).

I also increased the number of test executions to give better chances to hit any rare cases.

But to no avail.

Since QuickCheck kept telling me, “You’re wrong,” I finally started to admit, “Well, maybe I am indeed wrong and should take a closer look at the problem.”

Rethinking parallel evaluation in Haskell
Taking a closer look at the definition of the parallel MapReduce will allow us to better understand what’s actually going on:

import           Control.Parallel (par)
import           Control.Parallel.Strategies (using, parMap, rpar)

parMapReduce 
  :: (a -> b)   -- map function
  -> ([b] -> c) -- reduce function
  -> [a]        -- list to map over
  -> c          -- result
parMapReduce mapFunc reduceFunc input =
    mapResult `par` reduceResult
    where mapResult    = parMap rpar mapFunc input
          reduceResult = reduceFunc mapResult `using` rpar

-- and now an actual example usage:
x = parMapReduce reverse (foldr (⊕) "") [" olleh"," ym"," raed"," sklof"]     
In this concrete example mapResult will be:

mapResult    = parMap rpar reverse [" olleh"," ym"," raed"," sklof"]
parMap is defined as follows:

parMap :: Strategy b -> (a -> b) -> [a] -> [b]
parMap strat f = (`using` parList strat) . map f
The parMap evaluation strategy will spark a parallel evaluation for each element of the input list. Nevertheless the actual order of elements will not be changed as internally the classical sequential map function is used which will not change the order of elements. So the reduce phase will never receive a changed order of elements from the map phase, even if map-computations for the individual list elements might be executed in random order!

mapResult will always be ["hello", "my ", "dear ", "folks"].

Thus reduceResult will be:

reduceResult = (foldr (⊕) "") ["hello", "my ", "dear ", "folks"] `using` rpar
Again the traditional semantics of foldr is maintained — in particular the order of arguments of all (⊕) operations, only we allow for parallel evaluation of those (⊕) operations during the reduction phase.

So the final output will always be "hello my dear folks". The parallelism introduced by the Control.Parallel package does not in any way change the semantics of pure functional programs as our example. This is exactly what the failed test cased kept telling me:

There do not exist any cases where sequential and parallel MapReduce result in deviating results!

We can again evaluate our improved theory with a QuickCheck test:

    it "parallel reduction always equals sequential reduction" $
      property $ \l -> simpleMapReduce reverse (foldr (⊕) "") l
                     `shouldBe` parMapReduce reverse (foldr (⊕) "") l
And — not so surprisingly — this test succeeds!

If you want to know more about parallel evaluation in Haskell I highly recommend the exellent Parallel and Concurrent Programming in Haskell by Simon Marlow.

Conclusions
The parallelism as provided by the Haskell Control.Parallel package maintains the semantics of pure functional code and thus a parallel MapReduce maintains the same properties as its sequential counterpart. So a parallel MapReduce will still work correctly if the intermediate data structure resulting from the map-phase is just a Monoid — not necessarily a commutative Monoid.

Nevertheless there may be implementations that do not strictly maintain the original order of the input data during the map- and reduce-phases. With such implementations the intermediate data structure resulting from the map-phase must be a commutative Monoid under the reduce-operation to produce correct results.

Property based testing with QuickCheck is a powerful tool to verify assumptions about a given code-base. I really like using it as intended by Karl Poppers Theory of Falsifiability:

Derive hypotheses from your theory which can be experimentally tested.
Perform experiments (property based tests) that test your hypotheses
If the experimental results contradict the hypotheses, the theory must be false.
Site proudly generated by Hakyll | Source code is available at GitHub