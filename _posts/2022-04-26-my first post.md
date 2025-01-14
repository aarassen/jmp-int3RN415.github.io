---
published: true
---
# Welcome to jmp internals 

So i would like to start by stating the purpose of this blog. In here i will post about reverse engineering and exploit developement, and i can share also my ctf writeup here.

so let's begin with our [first challenge](https://challenges.re/1/)

## THE CHALLENGE


### THE TASK
What does this code do? The function has 4 arguments and it is compiled by GCC for Linux x64 ABI (i.e., arguments are passed in registers).
```asm
f:
   0:                   mov    r8,rdi
   3:                   push   rbx
   4:                   mov    rdi,rsi
   7:                   mov    rbx,rdx
   a:                   mov    rsi,r8
   d:                   xor    rdx,rdx

begin:
  10:                   lods   rax,QWORD PTR ds:[rsi]
  12:                   div    rbx
  15:                   stos   QWORD PTR es:[rdi],rax
  17:                   loop   begin
  19:                   pop    rbx
  1a:                   mov    rax,rdx
  1d:                   ret
```

### SOLUTION: 

**Step1 : understanding the question?**

	The function has **4 arguments**  (i.e., arguments are passed in registers).== check calling convention
    compiled by GCC for Linux x64 ABI == 64bit abi
    
    
**Step2 : find resources  online** 
	
[x64 assembly cheat sheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)
    
[asm pdf](http://www.lacl.fr/tan/asm)

[abi x64  resource](https://web.archive.org/web/20160801075146/http://www.x86-64.org/documentation/abi.pdf)

[x64 calling convention](https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/linux-x64-calling-convention-stack-frame)
    
**Step3 : understand the code using the resouurce**
from calling convention i understund that :

	1st arg is in RDI
    2st arg is in RSI
    3st arg is in RDX
    4st arg is in RCX
    
	
understatunding f:
  	
```asm
    f:
       0:                   mov    r8,rdi 	# copy rdi  to r8 
       3:                   push   rbx 		#  puch the value from rbx to the stack
       4:                   mov    rdi,rsi 	# copy the rsi  to rdi 
       7:                   mov    rbx,rdx  # copy the value from rdx to rbx 
       a:                   mov    rsi,r8 	# copy the value from r8 to rsi 
       d:                   xor    rdx,rdx 	# set rdx to 0
```

**so what happen here ???**

	1st arg was in rdi now its in rsi and r8
    
    2end arg was in rsi now its in rdi
    
    3rd arg was in rdx  after pushing rbx to the stack we place rdx in rbx then we set rdx to 0
    
    4th arg stay in ecx
    
finding more resource

[Registers purpose](http://6.s081.scripts.mit.edu/sp18/x86-64-architecture-guide.html)

[qword pointer = 8byte](https://stackoverflow.com/questions/31211561/what-does-qword-ptr-hexvalue-mean-without-a-base-register) meaning take 8 byte from the location pointing at.

[data segment register](https://www.tek-tips.com/viewthread.cfm?qid=717198)



understaunding start:
```asm
begin:
  10:                   lods   rax,QWORD PTR ds:[rsi] #Load qword at address RSI into RAX.
  12:                   div    rbx 
  			# div rbx will divide the 128-bit value in (**rdx, rax**) by **rbx**
            # then store the quotient in rax and the remainder in rdx.

  15:                   stos   QWORD PTR es:[rdi],rax #Store RAX at address RDI 
  
  17:                   loop   begin 
  			# Performs a loop operation using the RCX as a counter and keep decrementing it 
  			# if rcx = 0 pass
  19:                   pop    rbx 
  			# pop from top of the stack to rbx and increment or decrement the stack pointer
  			# increment stack pointer if df (flag) is set or not
  1a:                   mov    rax,rdx
  1d:                   ret # copy  value from rdx to rax and return
```



line 10 : after \<f\> so it find a value 64bit in lenght pointed by rsi and put it in rax after that  increment or decrement  rsi by 8  (check DF flag ;)

line 12 :then divide rax by the value in rbx (dont forgot that rdx is 0) and result saved in rax 

line 15 :then move  rax  to memory pointed by rdi after that  increment or decrement  rsi by 8  (check DF flag ;)

line 17 :loop from start until rcx is 0 then execute next instruction

line 19 :pop rbx because we needed to use rbx in start so we saved it's value in \<f\> at the stack and now we get the value back

line 1a : mov rdx which is 0 in value to rax then return


finding more resource

[lods instuction](https://www.felixcloutier.com/x86/lods:lodsb:lodsw:lodsd:lodsq)

[stos instruction](https://www.felixcloutier.com/x86/stos:stosb:stosw:stosd:stosq)

[loop instruction](https://www.felixcloutier.com/x86/loop:loopcc)


# final thought:
- save rbx to the stack
- it copy 8 byte from memory pointed by the 1st argument 
- then divid it with 3rd argument that it is always 0
- and move the result to memory pointed by the 2nd argument 
- until 4 argument equal 0
- pob value we saved in the stack back to rbx
- return 0



# CONCLUSION :

i need a job in the field of reverse engineering and exploit developement ;)
