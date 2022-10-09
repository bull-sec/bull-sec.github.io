# BEHEMOTH!

Decided to do some more stuff on OverTheWire (really digging this site), and found a WarGame called "Behemoth" which seems to be full of binary challenges, and since I've been wanting to get some practise in with `gdb` it seems like the perfect oppourtunity to do so, and learn a few bits along the way.

A little bit about the challenges by the developers:

> This wargame deals with a lot of regular vulnerabilities found commonly 'out 
in the wild'. While the game makes no attempts at emulating a real environment
it will teach you how to exploit several of the most common coding mistakes 
including buffer overflows, race conditions and privilege escalation.

Let's get cracking!

## Level 00

OK, so navigate to the `/behemoth` folder and you'll see a bunch of binaries, these are our target files. We don't need to look at anything else on this box (other than one more file in the same folder, which we'll get to later)

## Level 01

This is a nice little binary to get us started. 

First thing we'll do with all of these binaries is to run them and see what we're being asked for. In the case of this first challenge, we're just being prompted for a password.

So, lets give it one by just spamming the "A" character until we get bored.

`ACCESS DENIED`

OK, that was predictable.

Let's run `strings` on the binary then, see if there isn't something lurking there we can use.

`strings behemoth0`

Ohhh, that's interesting, we have what look like three potential passwords in the `strings` output, let's give each of them a try.

`ACCESS DENIED`

Booo, we could also try concatenating them together though, maybe it's expecting all three strings as a single input?

`ACCESS DENIED`

Again, boooo! Looks like we're going to have to break out the debugger.

`gdb behemoth0`

First thing we'll do is set a breakpoint on `main` and take a look at it. 

`b * main`

Then type `r` or `run` to run the program until we hit our breakpoint. 

(OK, so at this point I wanted to copy paste some stuff off my Kali box, but I didn't have the virtualBox extensions setup properly. So I decided to quickly redo the binary from my host machine, and I ran into some issues, the issues being that I was unable to replicate what happened on my Kali box... I are the confuse... UPDATE! Turns out that I wasn't paying attention to the value I was trying to place a breakpoint on, it's working fine now)


OK, so once we've hit the breakpoint at `main` we can start to check out how this thing is actually working by disassembling the function.

`disass main`

Oh, you might also want to do a `set disassembly-flavor intel`, this is just a personal choice I make because I prefer the Intel syntax (but if you want to follow along it'll make a bit more sense if you're on the same assembly flavour as I am)

Disassembling the `main` function gives us the following data dump:

```
(gdb) disass main
Dump of assembler code for function main:
   0x08048602 <+0>:	push   ebp
   0x08048603 <+1>:	mov    ebp,esp
   0x08048605 <+3>:	push   ebx
   0x08048606 <+4>:	and    esp,0xfffffff0
   0x08048609 <+7>:	sub    esp,0x70
   0x0804860c <+10>:	mov    eax,gs:0x14
   0x08048612 <+16>:	mov    DWORD PTR [esp+0x6c],eax
   0x08048616 <+20>:	xor    eax,eax
   0x08048618 <+22>:	mov    DWORD PTR [esp+0x1f],0x475e4b4f
   0x08048620 <+30>:	mov    DWORD PTR [esp+0x23],0x45425953
   0x08048628 <+38>:	mov    DWORD PTR [esp+0x27],0x595e58
   0x08048630 <+46>:	mov    DWORD PTR [esp+0x10],0x8048780
   0x08048638 <+54>:	mov    DWORD PTR [esp+0x14],0x8048798
   0x08048640 <+62>:	mov    DWORD PTR [esp+0x18],0x80487ad
   0x08048648 <+70>:	mov    DWORD PTR [esp],0x80487c1
   0x0804864f <+77>:	call   0x8048440 <printf@plt>
   0x08048654 <+82>:	lea    eax,[esp+0x2b]
   0x08048658 <+86>:	mov    DWORD PTR [esp+0x4],eax
   0x0804865c <+90>:	mov    DWORD PTR [esp],0x80487cc
   0x08048663 <+97>:	call   0x80484c0 <__isoc99_scanf@plt>
   0x08048668 <+102>:	lea    eax,[esp+0x1f]
   0x0804866c <+106>:	mov    DWORD PTR [esp],eax
   0x0804866f <+109>:	call   0x80484a0 <strlen@plt>
   0x08048674 <+114>:	mov    DWORD PTR [esp+0x4],eax
   0x08048678 <+118>:	lea    eax,[esp+0x1f]
   0x0804867c <+122>:	mov    DWORD PTR [esp],eax
   0x0804867f <+125>:	call   0x80485dd <memfrob>
   0x08048684 <+130>:	lea    eax,[esp+0x1f]
   0x08048688 <+134>:	mov    DWORD PTR [esp+0x4],eax
   0x0804868c <+138>:	lea    eax,[esp+0x2b]
   0x08048690 <+142>:	mov    DWORD PTR [esp],eax
   0x08048693 <+145>:	call   0x8048430 <strcmp@plt>
   0x08048698 <+150>:	test   eax,eax
   0x0804869a <+152>:	jne    0x80486ce <main+204>
   0x0804869c <+154>:	mov    DWORD PTR [esp],0x80487d1
   0x080486a3 <+161>:	call   0x8048470 <puts@plt>
   0x080486a8 <+166>:	call   0x8048460 <geteuid@plt>
   0x080486ad <+171>:	mov    ebx,eax
   0x080486af <+173>:	call   0x8048460 <geteuid@plt>
   0x080486b4 <+178>:	mov    DWORD PTR [esp+0x4],ebx
   0x080486b8 <+182>:	mov    DWORD PTR [esp],eax
   0x080486bb <+185>:	call   0x8048490 <setreuid@plt>
   0x080486c0 <+190>:	mov    DWORD PTR [esp],0x80487e2
   0x080486c7 <+197>:	call   0x8048480 <system@plt>
   0x080486cc <+202>:	jmp    0x80486da <main+216>
   0x080486ce <+204>:	mov    DWORD PTR [esp],0x80487ea
   0x080486d5 <+211>:	call   0x8048470 <puts@plt>
   0x080486da <+216>:	mov    eax,0x0
   0x080486df <+221>:	mov    edx,DWORD PTR [esp+0x6c]
   0x080486e3 <+225>:	xor    edx,DWORD PTR gs:0x14
   0x080486ea <+232>:	je     0x80486f1 <main+239>
   0x080486ec <+234>:	call   0x8048450 <__stack_chk_fail@plt>
   0x080486f1 <+239>:	mov    ebx,DWORD PTR [ebp-0x4]
   0x080486f4 <+242>:	leave  
   0x080486f5 <+243>:	ret    
End of assembler dump.
```

OK, so lets take a look at the exectution flow here... Remember, all of the disassembled instructions read from right to left.

Specifically we're looking for data being copied in and out of registers. (because we know we're looking for a password to the next level) 

We're also looking for the point at which the user is prompted for input, because we know that any logic the application performs cannot be done until we have a value to compare. We can see what looks to be the prompt at instructions 86-97, where the application calls `printf` and then it makes a call to `scanf` (that's the user input part)

Moving on from that bit, we can see the application moving some stuff around and setting up some pointers, and then we find the interesting part...

`strcmp` a.k.a. String compare. 

Then we're testing whether the value of `eax` is correct. With me so far? Cool. 

OK, so just before that `strcmp` we have some more interesting code to check out. Specifically the part where we're doing a `mov` from `eax` into `esp+0x4`.

Brief Aside: This binary contains perhaps my favourite function of GNU C, `memfrob`. Bro, do you even Frobnicate? Frobnication is the function of performing an exclusive OR on the first N bytes of a string, using the number 42... No I have no idea either, but the fact that it uses the number 42 gives me the warm fuzzies, so it can stay, just don't let me catch you using it in production code.

Back to more pertinent things, we just noticed that `mov` function. What that is doing, is moving our value, from `ebx` into `esp+0x4` (that's `esp` + 4 bytes)


<!-- Notes: Once we've cracked the binary, we need to cat the file at /etc/behemoth_pass/<next-level-name> -->



## Level 02

OK, so as soon as I got onto this one I managed to cause a SEGFAULT (mmmm SEGFAULT) by just throwing a load of "A" characters into the prompt. Which suggests this might be a buffer overflow (something to bear in mind as we continue). 

But, instead of just throwing random amounts of A's at things, lets be a bit more scientific about this.

`python -c 'print "A" * 32'`

Which, if you're unfamiliar with Python, will print us exactly 32 "A" characters. 

Using this method gives us a bit more control over what we're doing. 

OK, so 32 worked (i.e. no SEGFAULT)

Let's try something longer...

`python -c 'print "A" * 64'`

64 seems to work as well, so we're going to have to keep doing this until we find the "magic" number. 

128 caused a SEGFAULT, now we need to work backwards to find where exactly the length of this buffer is. 

Turns out that we need 79. 
