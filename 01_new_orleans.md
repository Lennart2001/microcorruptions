# Microcorruption - New Orleans

Good evening ladies and gentlemen. We are here to solve the CTF style reverse engineering challenges: Microcorruptions.

Let's see what we can find here. When looking through the level, I found an interesting subroutine.
The subroutine is called `create_password` which, I suppose, creates the password for the level.

`create_password` subroutine:

``` text
447e <create_password>
447e:  3f40 0024      mov	#0x2400, r15
4482:  ff40 3b00 0000 mov.b	#0x3b, 0x0(r15)
4488:  ff40 6b00 0100 mov.b	#0x6b, 0x1(r15)
448e:  ff40 7900 0200 mov.b	#0x79, 0x2(r15)
4494:  ff40 5600 0300 mov.b	#0x56, 0x3(r15)
449a:  ff40 7500 0400 mov.b	#0x75, 0x4(r15)
44a0:  ff40 6f00 0500 mov.b	#0x6f, 0x5(r15)
44a6:  ff40 6000 0600 mov.b	#0x60, 0x6(r15)
44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)
44b0:  3041           ret
```

We can set a break point at memory `447e` using `break 447e`: <span style="color:blue">447e:  3f40 0024      mov	#0x2400, r15</span>

We know that we make `register 15` point to the memory address `0x2400`. Let's see what happens when we step through
the function.

``` text
0000: 0000 4400 0000 0000 0000 0000 0000 0000   ..D.............
0010: 3041 0000 0000 0000 0000 0000 0000 0000   0A..............
0020: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0170: *  
2400: 3b00 0000 0000 0000 0000 0000 0000 0000   ;...............   <-------
2410: 0000 0000 0000 0000 0000 0000 0000 0000   ................
2420: *   
4400: 3140 0044 1542 5c01 75f3 35d0 085a 3f40   1@.D.B\.u.5..Z?@
4410: 0000 0f93 0724 8245 5c01 2f83 9f4f c245   .....$.E\./..O.E
4420: 0024 f923 3f40 0800 0f93 0624 8245 5c01   .$.#?@.....$.E\.
```

After we executed line: <span style="color:yellow">4482:  ff40 3b00 0000 mov.b  #0x3b, 0x0(r15)</span> in our program,
we can see that there has been a change made to the value at memory address `0x2400`. 

This is because we moved `0x3b` to `0x0(r15)`, which means we moved the value to the memory address of `register 15`
plus `0x0` (or zero) -> `0x2400`.

If we do another step, thereby executing <span style="color:yellow">4488:  ff40 6b00 0100 mov.b	#0x6b, 0x1(r15)</span>
we can see that we are adding another byte to the memory address `0x1(r15)`, which means we are adding to `register 15`
plus `0x1` (or one) -> `0x2401`. 


``` text
0000: 0000 4400 0000 0000 0000 0000 0000 0000   ..D.............
0010: 3041 0000 0000 0000 0000 0000 0000 0000   0A..............
0020: 0000 0000 0000 0000 0000 0000 0000 0000   ................
0170: *  
2400: 3b6b 0000 0000 0000 0000 0000 0000 0000   ;k..............   <-------
2410: 0000 0000 0000 0000 0000 0000 0000 0000   ................
2420: *    
4400: 3140 0044 1542 5c01 75f3 35d0 085a 3f40   1@.D.B\.u.5..Z?@
4410: 0000 0f93 0724 8245 5c01 2f83 9f4f c245   .....$.E\./..O.E
4420: 0024 f923 3f40 0800 0f93 0624 8245 5c01   .$.#?@.....$.E\.
```

If we keep stepping through the function, we will eventually end up with: 

``` text
0000: 0000 4400 0000 0000 0000 0000 0000 0000   ..D.............
0010: 3041 0000 0000 0000 0000 0000 0000 0000   0A..............
0020: *   
2400: 3b6b 7956 756f 6000 0000 0000 0000 0000   ;kyVuo`.........   <-------
2410: 0000 0000 0000 0000 0000 0000 0000 0000   ................
2420: *   
4400: 3140 0044 1542 5c01 75f3 35d0 085a 3f40   1@.D.B\.u.5..Z?@
4410: 0000 0f93 0724 8245 5c01 2f83 9f4f c245   .....$.E\./..O.E
```

So the password in hex would be: `3b6b7956756f60`.


