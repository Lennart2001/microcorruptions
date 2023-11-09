# Microcorruption - Hanoi

Good evening ladies and gentlemen. We are here to solve the CTF style reverse engineering challenges: Microcorruptions.

Let's first look at the main subroutine. 

``` text
4438 <main>
4438:  b012 2045      call	#0x4520 <login>
443c:  0f43           clr	r15
```

That is not a very informative main function. However, I suppose we can imply that we shifted our focus from the `<main>`
subroutine to the `<login>` subroutine.

``` text
4520 <login>
4520:  c243 1024      mov.b	#0x0, &0x2410
4524:  3f40 7e44      mov	#0x447e "Enter the password to continue.", r15
4528:  b012 de45      call	#0x45de <puts>
452c:  3f40 9e44      mov	#0x449e "Remember: passwords are between 8 and 16 characters.", r15
4530:  b012 de45      call	#0x45de <puts>
4534:  3e40 1c00      mov	#0x1c, r14
4538:  3f40 0024      mov	#0x2400, r15
453c:  b012 ce45      call	#0x45ce <getsn>
4540:  3f40 0024      mov	#0x2400, r15
4544:  b012 5444      call	#0x4454 <test_password_valid>
4548:  0f93           tst	r15
454a:  0324           jz	$+0x8 <login+0x32>
454c:  f240 fa00 1024 mov.b	#0xfa, &0x2410
4552:  3f40 d344      mov	#0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call	#0x45de <puts>
455a:  f290 8900 1024 cmp.b	#0x89, &0x2410
4560:  0720           jnz	$+0x10 <login+0x50>
4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
4566:  b012 de45      call	#0x45de <puts>
456a:  b012 4844      call	#0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov	#0x4501 "That password is not correct.", r15
4574:  b012 de45      call	#0x45de <puts>
4578:  3041           ret
```

> The `<getsn>` subroutine asks the user for input and then returns a new line.

We can see in line <span style="color:yellow">4544:  b012 5444       call     #0x4454 <test_password_valid></span>
that we are calling another subroutine called `test_password_valid`. Let's check that subroutine out.

``` text
4454 <test_password_valid>
4454:  0412           push	r4
4456:  0441           mov	sp, r4
4458:  2453           incd	r4
445a:  2183           decd	sp
445c:  c443 fcff      mov.b	#0x0, -0x4(r4)
4460:  3e40 fcff      mov	#0xfffc, r14
4464:  0e54           add	r4, r14
4466:  0e12           push	r14
4468:  0f12           push	r15
446a:  3012 7d00      push	#0x7d
446e:  b012 7a45      call	#0x457a <INT>
4472:  5f44 fcff      mov.b	-0x4(r4), r15
4476:  8f11           sxt	r15
4478:  3152           add	#0x8, sp
447a:  3441           pop	r4
447c:  3041           ret
```

We do not have any jumps here. That means we check the password in one go going through the subroutine. However, we
also do not have any `cmp` statements. So what gives? 

Let's take another look at the `<login>` subroutine.


``` text
<snip>
454c:  f240 fa00 1024 mov.b	#0xfa, &0x2410
4552:  3f40 d344      mov	#0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call	#0x45de <puts>
455a:  f290 8900 1024 cmp.b	#0x89, &0x2410   <-----------
4560:  0720           jnz	$+0x10 <login+0x50>
4562:  3f40 f144      mov	#0x44f1 "Access granted.", r15
4566:  b012 de45      call	#0x45de <puts>
456a:  b012 4844      call	#0x4448 <unlock_door>
456e:  3041           ret
<snip>
```

We can see at line `455a` we have a compare statement, where the `ALU` sets the `zero-flag` if the comparison
is true (if both operands are equal).

> 455a:  f290 8900 1024 cmp.b	#0x89, &0x2410

We have an immediate value `0x89` that's being compared to the stored value at address `0x2410` the `&`
means **absolute address**. 

Where does the password get stored? Maybe we can "overflow" the value and overwrite the value stored in the 
`0x2410` memory address. 

``` text
0030: *  
0150: 0000 0000 0000 0000 0000 0000 085a 0000   .............Z..
0160: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0170: *  
2400: 686b 6565 6b65 0000 0000 0000 0000 0000   hkeeke..........   <---------
2410: 0000 0000 0000 0000 0000 0000 0000 0000   ................
2420: *  
43f0: 8e45 0000 d845 0200 0024 1c00 4045 3c44   .E...E...$..@E<D
4400: 3140 0044 1542 5c01 75f3 35d0 085a 3f40   1@.D.B\.u.5..Z?@
4410: *
```

Hmm. So the password we give gets written to address `0x2400` this means we can indeed overflow into the memory
address `0x2410`. Given that one char is one byte, and one byte is 2 hex characters. We can create the following
password `4242424242424242424242424242424289` and try to overwrite the memory address at `0x2410`

Boom. We got it correct. 


