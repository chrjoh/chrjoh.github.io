---
layout: post
title: "Bind shell"
date: 2014-05-25 09:47:42 +0200
comments: false
categories: 
---
bindshell
=============
This is my second try, and now the goal is to write some code to setup a listening server for OSX 64-bit system. I started with creating the c code to do the job
to have someting to work from the code can be found [here](https://github.com/chrjoh/shell_code/blob/master/bindshell/bindshell_template.c)
Will keep this post a bit short as much of the things to do are described in my previous post, 
[Reverse Shell](http://blog.lodakai.com/blog/2014/05/20/reverse-shell/). Following the c code I cam up with this 
[assembler version](https://github.com/chrjoh/shell_code/blob/master/bindshell/bindshell.s).
(Had some problem with the code, but @norsec0de and @TheColonial helped me to fix it, big thx to them for the help)

#### bindshell, Usage:
  bindshell start a listening server that you can connect to with the command: `nc -nv 127.0.0.1 4444` in another folder for example your home directory
  
```
  nasm -g -f macho64 bindshell.s
  ld  -arch x86_64 -macosx_version_min 10.7.0 -lSystem -o bindshell bindshell.o
  ./bindshell
```  
Go to the terminal and run `nc -nv 127.0.0.1 4444` then type ls, you should now see a directory listing of the directory you started
bindshell in.

The next step was to convert it to something that could be used as payload, i.e remove "bad values", see the reverse shell post.
I ended up with this, not optomized with respect of length the result was 
[bindshell_no_bad_values.s](https://github.com/chrjoh/shell_code/blob/master/bindshell/bindshell_no_bad_values.s).
Then I converted this to shell code values and tried the c code 
[bindshell_test.c](https://github.com/chrjoh/shell_code/blob/master/bindshell/bindshell_test.c) there was some problems could not
get the code to listen for incoming calls. To solve that I used the excellent command `dtruss ./bindshell_test` and by looking
at the output I found that my socket call got the wrong values this I fixed by adding the setuid call. Example of what `dtruss`
result in can be seen bellow:

```
.
.
.
getaudit_addr(0x7FFF554B7C90, 0x30, 0x0)   = 0 0
csops(0x16821, 0x7, 0x7FFF554B7870)        = -1 Err#22
setuid(0x0, 0x0, 0x7FFF554B89F0)           = 0 0
socket(0x2, 0x1, 0x0)                      = 3 0
bind(0x3, 0x7FFF554B89AC, 0x10)            = 0 0
listen(0x3, 0x0, 0x0)                      = 0 0
close(0x3)                                 = 0 0
.
.
```
Then that was done the c code worded as expected. For simplicity I also include a Makefile to compile the various examples. To continue
my road into the shell code I bought 
[The Shellcoder's Handbook: Discovering and Exploiting Security Holes](http://www.amazon.com/gp/product/B004P5O38Q/ref=oh_d__o00_details_o00__i00?ie=UTF8&psc=1)
Do not know if the next post will be something about wifi hacking or assembler the time will tell.





