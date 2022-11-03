Vim file templates are useful for starting new files with boilerplate code. There are plenty of [snippet](http://l.main.getrevue.co/ls/click?upn=gVFUf18rutnxjaj5Ut67g-2BdYkcH1ct3ETZpsP8S9wB3ByuQo2D-2BW2s3Ij5ai0vHF8zUEe5b7hsCi86M-2FkolQRLVmP8gSH5wgJz6SCln2hZmzIvmTdc15PmHeGSHCqOMAmcv9vtmm9EI0MyHA0Ytj2VoQRxglWz1pflsORHLIhsY-3DyPD-_bSU5UaU3UURpzsBsHtzXEWpUE2eOJvOv1vv7GR2dl9B-2FkZYGMlR5wbl-2BlDsbpVdNtkAJH-2FPdiN-2F4x87b-2FJBryzw-2FtjNP1p8LpDWFR2vD0RcJ34BpqS9MPOshnp4rd-2Fw8J5U-2F8-2FpjUP-2F-2FiDskKlpeqSBh5zZjYi2oXajlDvtJJ3eu0j99qefEUva8qMpKS2gIxqSDI1dtw4BFkTTGLo1Z-2F7WDVV1pWhRCpJEjb0iQqd4hM8g51KzY-2FaYMqsGLD5jdsaeb7mb9vkqx5OuuW54PYhBxtiLA-2BRd-2BaLbi6feaZl4hV87xQi2qKLztsTbsGcpmeEfaHWvEdQop-2Bys1X3XKU0k7xITxrX6Z5SkTmzprITOWDcLwx0zUipV8gMl8pz4d-2FIla8hnNll-2BRq8G5BfnriN3PwJeOvyho-2FGKPEsuJF8w-3D) and templating plugins for Vim, but you can create file templates without any plugins using a concept Vim calls **skeleton files**. This allows you to automatically populate a new file with a given template. Learn about it with `:help skeleton` but we will dive into it:

To create a Vim file template, you configure your Vim to read a file from disk every time you open a new, empty buffer with a filename that matches a given pattern. Skeleton files are really just a trick using Vim’s `[autocmd](http://l.main.getrevue.co/ls/click?upn=gVFUf18rutnxjaj5Ut67g-2B45LcNMpBLn3H8FZr8gIvOnwmDE2QYm-2FphGKW2Kh9jHI6IU4-2BNVaobus2EXjB6Yd3HGEZSe-2BdjnJR-2BIqMDRmrYVkZmTZ48MQCAc6oPb2BOlxMWNwgUgHDnNLtfg-2F-2F5jMgjNz3Gc0GZViylxLdeE4pw-3D798t_bSU5UaU3UURpzsBsHtzXEWpUE2eOJvOv1vv7GR2dl9B-2FkZYGMlR5wbl-2BlDsbpVdNtkAJH-2FPdiN-2F4x87b-2FJBryzw-2FtjNP1p8LpDWFR2vD0RcJ34BpqS9MPOshnp4rd-2Fw8J5U-2F8-2FpjUP-2F-2FiDskKlpeqSBh5zZjYi2oXajlDvtJJ3eu0j99qefEUva8qMpKS2gIxqSDI1dtw4BFkTTGLo1Z-2F7WDVV1pWhRCpJEjb0iQqd4hM8g51KzY-2FaYMqsGLD5jdsaeb7mb9vkqx5OuuW54PYm2fl4ZfeAHafm0FkzlPPk5K8xe5yDwdTV33gsrbwbw4eoqByndWRYlQXDa-2BRYkcZri-2FNC0ZwhWFM-2FowXgTkiD1BFc2or48FKZB2aYqv3X0SUQpj2E5cn-2B5yXDUMG1TaO31wkyUZAHvoXjea184gzqs-3D)` capabilities to run commands on certain events.

One file template example might be when writing shell scripts. You can create a skeleton file that contains the Bash shebang line like so:

`#!/usr/bin/env bash`

Then configure your Vim to populate any new and empty buffer that ends in `.sh` with this file’s contents. If you’re working in a language that requires a lot of boilerplate in every file, this can be an enormous time saver.

Another example use case is a README file. You could create a [template Markdown file](http://l.main.getrevue.co/ls/click?upn=gVFUf18rutnxjaj5Ut67gze3-2BsmiJZYmnLI7X3Arj0R-2BVNW3aNERIe66n1EPifqYwMreYKDWDIqNQCJxkPFYQQ0SNP8ftJjGoUN-2FEh1CFh2XIfGXae0axgalwdRTiWq8lm7aORg8CAAKsftqiNk-2FhfRxGW7LNXOoIi-2Bj8bOKxE-2FrLUzTwad-2F83jm1rieDfR4rA5d_bSU5UaU3UURpzsBsHtzXEWpUE2eOJvOv1vv7GR2dl9B-2FkZYGMlR5wbl-2BlDsbpVdNtkAJH-2FPdiN-2F4x87b-2FJBryzw-2FtjNP1p8LpDWFR2vD0RcJ34BpqS9MPOshnp4rd-2Fw8J5U-2F8-2FpjUP-2F-2FiDskKlpeqSBh5zZjYi2oXajlDvtJJ3eu0j99qefEUva8qMpKS2gIxqSDI1dtw4BFkTTGLo1Z-2F7WDVV1pWhRCpJEjb0iQqd4hM8g51KzY-2FaYMqsGLD5jdsaeb7mb9vkqx5OuuW54PYgLjOquihkq2L8JPXb6lE8wKZc5HZqsIcA-2B-2BvQO-2B56p9QLaLgxjZQv14NOwAQDHAuNNx4RQfks37QfSRr6A399ZKRULj4-2FR-2FC-2B2-2BIPIb7KBZgr184KAKUFILyPahiF9mFCvviW8AuX6TXT1izATMYYM-3D) that includes all the headings and basic bullet points that every good README should have. Then whenever you open a new file called `README.md` in Vim, the buffer will automatically pre-populate with the skeleton template file.

Configuring these templates in your `.vimrc` is simple:

`autocmd BufNewFile readme.md 0r ~/skeletons/readme.md`

`autocmd BufNewFile *.sh 0r ~/skeletons/bash.sh`

In this case, I’ve created a new directory inside my home directory called `skeletons` but you could put these anywhere. Here’s how the first one works:

-   `autocmd` – run this automatically on some event
-   `BufNewFile` – this is Vim’s new file event
-   `readme.md` – this is the pattern you want the new file to match
-   `0r` – read into the buffer starting at line 0, the first line
-   `~/skeletons/readme.md` – the file to read in

The second example works the same way but uses a file glob `*.sh` to match any file ending in `.sh`. There’s a quick demo of this in action below.