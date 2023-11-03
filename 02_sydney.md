# Microcorruption - Sydney

Good evening ladies and gentlemen. We are here to solve the CTF style reverse engineering challenges: Microcorruptions.

Let's first look at the main subroutine. 

``` text
4438 <main>
4438:  3150 9cff      add	#0xff9c, sp
443c:  3f40 b444      mov	#0x44b4 "Enter the password to continue.", r15
4440:  b012 6645      call	#0x4566 <puts>
4444:  0f41           mov	sp, r15
4446:  b012 8044      call	#0x4480 <get_password>
444a:  0f41           mov	sp, r15
444c:  b012 8a44      call	#0x448a <check_password>
4450:  0f93           tst	r15
4452:  0520           jnz	$+0xc <main+0x26>
4454:  3f40 d444      mov	#0x44d4 "Invalid password; try again.", r15
4458:  b012 6645      call	#0x4566 <puts>
445c:  093c           jmp	$+0x14 <main+0x38>
445e:  3f40 f144      mov	#0x44f1 "Access Granted!", r15
4462:  b012 6645      call	#0x4566 <puts>
4466:  3012 7f00      push	#0x7f
446a:  b012 0245      call	#0x4502 <INT>
446e:  2153           incd	sp
4470:  0f43           clr	r15
4472:  3150 6400      add	#0x64, sp
```

If we set a breakout at `0x4438`, and go one step, we set the stackpointer to `0xff9c`. If we go another step, 
we then put the memory address of a string into `register 15`. After that we call a subroutine named `puts`. I am
not sure if this is a standard subroutine, or important for us to look at. *I believe it just prints out the string.*

Regardless, let's continue. After another step, we move the current pointer's value to `register 15`. The current
`stack pointer` is `0x439c` so `register 15` will be `0x439c` as well. 

After a couple more steps and getting the **password**, we are now at this line: <span style="color:yellow">444a:  0f41          mov      sp, r15</span>.
We now move to the `check_password` subroutine:

```
448a <check_password>
448a:  bf90 335e 0000 cmp	#0x5e33, 0x0(r15)
4490:  0d20           jnz	$+0x1c <check_password+0x22>
4492:  bf90 4e71 0200 cmp	#0x714e, 0x2(r15)
4498:  0920           jnz	$+0x14 <check_password+0x22>
449a:  bf90 4858 0400 cmp	#0x5848, 0x4(r15)
44a0:  0520           jnz	$+0xc <check_password+0x22>
44a2:  1e43           mov	#0x1, r14
44a4:  bf90 584d 0600 cmp	#0x4d58, 0x6(r15)
44aa:  0124           jz	$+0x4 <check_password+0x24>
44ac:  0e43           clr	r14
44ae:  0f4e           mov	r14, r15
44b0:  3041           ret
```

After some closer inspection, I noticed that we make comparisons with `register 15` in increments of `0x2`. We compare
`register 15` against certain values, and if they are equal to one another, `cmp` returns 0. Therefore, we can assume
they hardcoded the password into the code, split off into fourths. 

>Conditional Jump Instruction (jnz): This checks the zero flag. If the zero flag is not set 
>(indicating the result of the last comparison was not zero), it will jump to the specified label/address.

Concatenate them together they yield: `335e4e714858584d`. Which is the password. 



