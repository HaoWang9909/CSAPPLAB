# Archlab

## `Learning Materials`

---

- Learning Materials
  
    [archlab32.pdf](Archlab/archlab32.pdf)

# Preparations

1. **Downloading the Handout**: The section instructs students to insert a paragraph explaining how to download the `archlab-handout.tar` file. This is site-specific and would be tailored to the institution's procedures.
2. **Unpacking the Handout**: Students are instructed to copy the `archlab-handout.tar` file to a protected directory where they plan to work. They then execute the command `tar xvf archlab-handout.tar` to unpack the following files into the directory: `README`, `Makefile`, `sim.tar`, `archlab.pdf`, and `simguide.pdf`.
3. **Creating the Y86-64 Tools Directory**: After unpacking, students are to execute `tar xvf sim.tar` to create a directory named `sim`, which contains their personal copy of the Y86-64 tools. All work for the lab is to be done inside this directory.
4. **Building the Y86-64 Tools**: Students are instructed to change to the `sim` directory and build the Y86-64 tools by running the commands `make clean; make`.
- Here’s a summary of issues encountered during environment setup and their solutions:
  
    **Library File Missing Error**
    
    - **Solution:**
      
        ```bash
        sudo apt-get update
        sudo apt-get install flex
        sudo apt-get install tk-dev tcl-dev
        ```
        
    
    **Multiple Definition Error**
    
    - **Solution:**
      
        Modify the Makefile in both misc and pipe directories as follows:
        
        Change:
        
        ```bash
        CFLAGS=-Wall -O1 -g
        LCFLAGS=-O1
        ```
        
        To:
        
        ```bash
        CFLAGS=-Wall -O1 -g -fcommon
        LCFLAGS=-O1 -fcommon
        ```
        
    

# Part A

- **Directory**: Work in the sim/misc directory.
- **Task**: Write and simulate three Y86-64 programs, with behavior defined by example C functions in examples.c.
- **Testing**: Assemble programs using YAS and run them with the YIS simulator to test functionality.
- **Conventions**: Follow x86-64 conventions for:
    - Passing function arguments.
    - Using registers and the stack.
    - Saving and restoring any callee-save registers you use.
- Code in example.c:
  
    ```c
    /* 
     * Architecture Lab: Part A 
     * 
     * High level specs for the functions that the students will rewrite
     * in Y86-64 assembly language
     */
    
    /* $begin examples */
    /* linked list element */
    typedef struct ELE {
        long val;
        struct ELE *next;
    } *list_ptr;
    
    /* sum_list - Sum the elements of a linked list */
    long sum_list(list_ptr ls)
    {
        long val = 0;
        while (ls) {
    	val += ls->val;
    	ls = ls->next;
        }
        return val;
    }
    
    /* rsum_list - Recursive version of sum_list */
    long rsum_list(list_ptr ls)
    {
        if (!ls)
    	return 0;
        else {
    	long val = ls->val;
    	long rest = rsum_list(ls->next);
    	return val + rest;
        }
    }
    
    /* copy_block - Copy src to dest and return xor checksum of src */
    long copy_block(long *src, long *dest, long len)
    {
        long result = 0;
        while (len > 0) {
    	long val = *src++;
    	*dest++ = val;
    	result ^= val;
    	len--;
        }
        return result;
    }
    /* $end examples */
    
    ```
    
    1. **Extract the outline of the y86-code.** Find the code in the sim/y86-code directory, for example the `asum.ys`.
       
        ```nasm
        # Execution begins at address 0 
        	.pos 0
        	irmovq stack, %rsp  	# Set up stack pointer
        	call main		# Execute main program
        	halt			# Terminate program 
        
        # Array of 4 elements
        	.align 8
        array:	.quad 0x000d000d000d
        	.quad 0x00c000c000c0
        	.quad 0x0b000b000b00
        	.quad 0xa000a000a000
        
        main:	irmovq array,%rdi
        	irmovq $4,%rsi
        	call sum		# sum(array, 4)
        	ret
        
        # long sum(long *start, long count)
        # start in %rdi, count in %rsi
        sum:	irmovq $8,%r8        # Constant 8
        	irmovq $1,%r9	     # Constant 1
        	xorq %rax,%rax	     # sum = 0
        	andq %rsi,%rsi	     # Set CC
        	jmp     test         # Goto test
        loop:	mrmovq (%rdi),%r10   # Get *start
        	addq %r10,%rax       # Add to sum
        	addq %r8,%rdi        # start++
        	subq %r9,%rsi        # count--.  Set CC
        test:	jne    loop          # Stop when 0
        	ret                  # Return
        
        # Stack starts here and grows to lower addresses
        	.pos 0x200
        stack:
        
        ```
        
        And we can extract the outline of the y86-code
        
        ```nasm
        # Execution begins at address 0 
        	.pos 0
        	irmovq stack, %rsp  	# Set up stack pointer
        	call main		# Execute main program
        	halt			# Terminate program 
        
        # Data
        ...
        
        main:
        
        # Stack starts here and grows to lower addresses
        	.pos 0x200
        stack:
        ```
        
    2. Write the `sum.ys` code
       
        ```nasm
        # Execution begins at address 0 
        	.pos 0
        	irmovq stack, %rsp  	# Set up stack pointer
        	call main		# Execute main program
            halt			# Terminate program 
        
        # Sample linked list
        .align 8
        ele1:
            .quad 0x00a
            .quad ele2
        ele2:
            .quad 0x0b0
            .quad ele3
        ele3:
            .quad 0xc00
            .quad 0
        
        main: 
            irmovq ele1, %rdi
            call sum
            ret
        
        # long sum(long *start)
        # start in %rdi
        sum:
            xorq %rax,%rax	     # sum = 0
            pushq %r10           # Save %r10
            jmp test            # Goto test
        
        loop:
            mrmovq (%rdi),%r10   # Get *start
            addq %r10,%rax       # Add to sum
            mrmovq 8(%rdi), %rdi # start++
            jmp test          # Goto test
        
        test:
            andq %rdi, %rdi     # Set CC
            jne loop           # Stop when 0
            popq %r10            # Restore %r10
            ret
        
        # Stack starts here and grows to lower addresses
            .pos 0x200
        stack:
          
        ```
        
    3. execute the code using
       
        ```bash
        ./yas sum.ys
        ./yis sum.yo
        ```
        
    4. examine the result and find that
       
        ```bash
        Stopped in 31 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
        Changes to registers:
        %rax:   0x0000000000000000      0x0000000000000cba
        %rsp:   0x0000000000000000      0x0000000000000200
        
        Changes to memory:
        0x01f0: 0x0000000000000000      0x000000000000005b
        0x01f8: 0x0000000000000000      0x0000000000000013
        ```
        
        As in the PDF, when %rax is changed to 0x0cba then the result is correct!
        
    5. Next, write the `rsm.ys` code
       
        ```nasm
        # Execution begins at address 0 
        	.pos 0
        	irmovq stack, %rsp  	# Set up stack pointer
        	call main		# Execute main program
            halt			# Terminate program 
        
        # Sample linked list
        .align 8
        ele1:
            .quad 0x00a
            .quad ele2
        ele2:
            .quad 0x0b0
            .quad ele3
        ele3:
            .quad 0xc00
            .quad 0
        
        main: 
            irmovq ele1, %rdi
            xorq %rax,%rax	 
            call rsum
            ret
        
        # long rsum_list(list_ptr ls)
        # start in %rdi
        rsum:  
            andq %rdi,%rdi
            je end
            mrmovq (%rdi), %r10
            mrmovq 8(%rdi), %rdi
            pushq %r10
            call rsum
            popq %r10
            addq %r10, %rax
            ret
        
        end:
            irmovq $0, %rax
            ret
        
        # Stack starts here and grows to lower addresses
            .pos 0x200
        stack:
          
        ```
        
        the result is
        
        ```bash
        Stopped in 32 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
        Changes to registers:
        %rax:   0x0000000000000000      0x0000000000000cba
        %rsp:   0x0000000000000000      0x0000000000000200
        
        Changes to memory:
        0x01e0: 0x0000000000000000      0x0000000000000069
        0x01f0: 0x0000000000000000      0x000000000000005b
        0x01f8: 0x0000000000000000      0x0000000000000013
        ```
        
    6. Next write the `copy.ys` code
       
        ```bash
        # Execution begins at address 0 
        	.pos 0
        	irmovq stack, %rsp  	# Set up stack pointer
        	call main		# Execute main program
            halt			# Terminate program 
        
        # Sample linked list
        .align 8
        # Source block
        src:
            .quad 0x00a
            .quad 0x0b0
            .quad 0xc00
        # Destination block
        dest:
            .quad 0x111
            .quad 0x222
            .quad 0x333
        
        main: 
            pushq %rdi
            pushq %rsi
            pushq %rdx
            irmovq src, %rdi        # long *src
            irmovq dest, %rsi       # long *dest
            irmovq $3, %rdx         # long len
            call copy
            popq %rdx
            popq %rsi
            popq %rdi
            ret
        
        # long copy_block(long *src, long *dest, long len)
        # start in %rdi
        copy: 
            pushq %r9
            pushq %r8
            pushq %r10
            xorq %rax,%rax                 #long result = 0 
            irmovq $1, %r9                #long i = 1  
            irmovq $8, %r8                #long temp = 0       
            jmp test 
        
        loop:
            mrmovq (%rdi), %r10
            rmmovq %r10, (%rsi)
            addq %r8, %rdi
            addq %r8, %rsi
            subq %r9, %rdx
            xorq %r10, %rax
            jne test
        
        test:
            andq %rdx,%rdx
            jne loop
            popq %r10
            popq %r8
            popq %r9                  
            ret
            ret
        
        # Stack starts here and grows to lower addresses
            .pos 0x200
        stack:
          
        ```
        
        the result is 
        
        ```bash
        Stopped in 54 steps at PC = 0x13.  Status 'HLT', CC Z=1 S=0 O=0
        Changes to registers:
        %rax:   0x0000000000000000      0x0000000000000cba
        %rsp:   0x0000000000000000      0x0000000000000200
        
        Changes to memory:
        0x0030: 0x0000000000000111      0x000000000000000a
        0x0038: 0x0000000000000222      0x00000000000000b0
        0x0040: 0x0000000000000333      0x0000000000000c00
        0x01d8: 0x0000000000000000      0x0000000000000075
        0x01f8: 0x0000000000000000      0x0000000000000013
        ```
        
    7. Insights
        - When designing functions, you should save the registers you use first and then restore them afterwards
        - The OP instructions of Y86 can only use registers, not operands, for example `subq`
        - Incrementing a linked list and incrementing an array are different. One uses `mrmovq 8(%rdi), %rdi`, while the other uses `addq %r?, %rdi`
    

# Part B

- **Objective**: Extend the SEQ processor to support the `iaddq` instruction
- **Modification**:Edit the `seq-full.hcl` file, which contains the SEQ processor implementation, to add control logic for the `iaddq` instruction.
- **Testing:**
    - Use make VERSION=full to build a new SEQ simulator (ssim) after modifying the `seq-full.hcl` file.
    - Test `iaddq` using small programs like asumi.yo in TTY mode:
      
        ```bash
        ./ssim -t ../y86-code/asumi.yo
        ```
        
1. write the phases
   
   
    | **Phase** | **iaddq V, rB** |
    | --- | --- |
    | Fetch | icode:ifun - M1[PC]
    rA:rB - M1[PC+1]
    valC - M8[PC+2]
    valP - PC+10 |
    | Decode | valB - R[rB] |
    | Execute | valE - valB+valC
    set CC |
    | Memory | - |
    | Write back | R[rB]-valE |
    | Update | PC - valP |
2. Validate the Instruction
   
    ```hcl
    ##### Symbolic representation of Y86-64 Instruction Codes #############
    wordsig INOP 	'I_NOP'
    wordsig IHALT	'I_HALT'
    wordsig IRRMOVQ	'I_RRMOVQ'
    wordsig IIRMOVQ	'I_IRMOVQ'
    wordsig IRMMOVQ	'I_RMMOVQ'
    wordsig IMRMOVQ	'I_MRMOVQ'
    wordsig IOPQ	'I_ALU'
    wordsig IJXX	'I_JMP'
    wordsig ICALL	'I_CALL'
    wordsig IRET	'I_RET'
    wordsig IPUSHQ	'I_PUSHQ'
    wordsig IPOPQ	'I_POPQ'
    # Instruction code for iaddq instruction
    **wordsig IIADDQ	'I_IADDQ'**
    ```
    
    We notice that this has already been added.
    
3. Modify each stage
   
    **Fetch:**
    
    ```hcl
    bool instr_valid = icode in 
    	{ INOP, IHALT, IRRMOVQ, IIRMOVQ, IRMMOVQ, IMRMOVQ,
    	       IOPQ, IJXX, ICALL, IRET, IPUSHQ, IPOPQ , **IIADDQ**};
    
    # Does fetched instruction require a regid byte?
    bool need_regids =
    	icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
    		     IIRMOVQ, IRMMOVQ, IMRMOVQ , **IIADDQ**};
    
    # Does fetched instruction require a constant word?
    bool need_valC =
    	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL , **IIADDQ**};
    ```
    
    **Decode and Write back:**
    
    ```hcl
    ## What register should be used as the A source?
    word srcA = [
    	icode in { IRRMOVQ, IRMMOVQ, IOPQ, IPUSHQ  } : rA;
    	icode in { IPOPQ, IRET } : RRSP;
    	1 : RNONE; # Don't need register
    ];
    
    ## What register should be used as the B source?
    word srcB = [
    	icode in { IOPQ, IRMMOVQ, IMRMOVQ, **IIADDQ** } : rB;
    	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
    	1 : RNONE;  # Don't need register
    ];
    
    ## What register should be used as the E destination?
    word dstE = [
    	icode in { IRRMOVQ } && Cnd : rB;
    	icode in { IIRMOVQ, IOPQ, **IIADDQ**} : rB;
    	icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
    	1 : RNONE;  # Don't write any register
    ];
    
    ## What register should be used as the M destination?
    word dstM = [
    	icode in { IMRMOVQ, IPOPQ } : rA;
    	1 : RNONE;  # Don't write any register
    ];
    ```
    
    **Execute:**
    
    ```hcl
    ## Select input A to ALU
    word aluA = [
    	icode in { IRRMOVQ, IOPQ } : valA;
    	icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, **IIADDQ** } : valC;
    	icode in { ICALL, IPUSHQ } : -8;
    	icode in { IRET, IPOPQ } : 8;
    	# Other instructions don't need ALU
    ];
    
    ## Select input B to ALU
    word aluB = [
    	icode in { IRMMOVQ, IMRMOVQ, IOPQ, ICALL, 
    		      IPUSHQ, IRET, IPOPQ, **IIADDQ** } : valB;
    	icode in { IRRMOVQ, IIRMOVQ } : 0;
    	# Other instructions don't need ALU
    ];
    
    ## Should the condition codes be updated?
    bool set_cc = icode in { IOPQ, **IIADDQ** };
    ```
    
4. Test
   
    ```bash
    make VERSION=full
    ```
    
    find error
    
    ```bash
    # Building the seq-full.hcl version of SEQ
    ../misc/hcl2c -n seq-full.hcl <seq-full.hcl >seq-full.c
    gcc -Wall -O2 -isystem /usr/include/tcl8.5 -I../misc -DHAS_GUI -o ssim \
            seq-full.c ssim.c ../misc/isa.c -L/usr/lib -ltk -ltcl -lm
    ssim.c:20:10: fatal error: tk.h: No such file or directory
       20 | #include <tk.h>
          |          ^~~~~~
    compilation terminated.
    make: *** [Makefile:44: ssim] Error 1
    ```
    
    we need to updates the Tcl version reference from tcl8.5 to tcl8.6 in the Makefile.
    
    And we also need to Modifies the CFLAGS to include the -DUSE_INTERP_RESULT flag, to maintain compatibility with legacy code that uses interp->result.
    
    ```bash
    sed -i "s/tcl8.5/tcl8.6/g" Makefile
    sed -i "s/CFLAGS=/CFLAGS=-DUSE_INTERP_RESULT /g" Makefile 
    ```
    
    Then we find another error:
    
    ```bash
    /usr/bin/ld: /tmp/ccKj98Ru.o:(.data.rel+0x0): undefined reference to `matherr'
    collect2: error: ld returned 1 exit status
    ```
    
    Then we need to uncomment these two lines in `ssim.c`:
    
    ```bash
    //extern int matherr();
    //int *tclDummyMathPtr = (int *) matherr;
    ```
    
    Then we use:
    
    ```bash
    (cd ../ptest; make SIM=../seq/ssim TFLAGS=-i)
    ```
    
    The result is correct!
    
    ```bash
    All 756 ISA Checks Succeed
    ```
    

# Part C

**Objective：**Accelerate the execution of the `ncopy` function.

- The `ncopy.ys` function needs to copy `len` elements from the `src` array to the `dst` array and return the count of positive integers in `src`.
  
    ```c
    #include <stdio.h>
    
    typedef word_t word_t;
    
    word_t src[8], dst[8];
    
    /* $begin ncopy */
    /*
     * ncopy - copy src to dst, returning number of positive ints
     * contained in src array.
     */
    word_t ncopy(word_t *src, word_t *dst, word_t len)
    {
        word_t count = 0;
        word_t val;
    
        while (len > 0) {
    	val = *src++;
    	*dst++ = val;
    	if (val > 0)
    	    count++;
    	len--;
        }
        return count;
    }
    /* $end ncopy */
    
    int main()
    {
        word_t i, count;
    
        for (i=0; i<8; i++)
    	src[i]= i+1;
        count = ncopy(src, dst, 8);
        printf ("count=%d\n", count);
        exit(0);
    }
    ```
    
- The objective is to make `ncopy.ys` run as quickly as possible in the Y86-64 PIPE simulator while maintaining functional correctness.

**Constraints**:

- **Adaptability for Any Array Size**: The implementation must handle arrays of any size dynamically. Hardcoding a fixed array size is not allowed, and the function must correctly process any value of `len` .
- **Size Limitation**: The assembled program size of `ncopy` must not exceed 1000 bytes.
- **Correctness**: The function must pass the tests on both the YIS and PIPE simulators, and the correct result must be returned in `%rax`.

**Useful commands:**

- Test length of the code:
  
    ```bash
    ../misc/yas ncopy.ys 
    ./check-len.pl < ncopy.yo 
    ```
    
- use the following command to see the result
  
    ```bash
    make VERSION=full #refer to Part B when find the matherr and tk issue
    pipe (cd ../ptest; make SIM=../pipe/psim TFLAGS=-i)     #check if the pipe-full.hcl is correct
    ./correctness.pl
    ./benchmark.pl
    ```
    
- **Solution:**
    - add the `iaddq` instruction to `pipe-full.hcl`
      
        ```hcl
        #/* $begin pipe-all-hcl */
        ####################################################################
        #    HCL Description of Control for Pipelined Y86-64 Processor     #
        #    Copyright (C) Randal E. Bryant, David R. O'Hallaron, 2014     #
        ####################################################################
        
        ## Your task is to implement the iaddq instruction
        ## The file contains a declaration of the icodes
        ## for iaddq (IIADDQ)
        ## Your job is to add the rest of the logic to make it work
        
        ####################################################################
        #    C Include's.  Don't alter these                               #
        ####################################################################
        
        quote '#include <stdio.h>'
        quote '#include "isa.h"'
        quote '#include "pipeline.h"'
        quote '#include "stages.h"'
        quote '#include "sim.h"'
        quote 'int sim_main(int argc, char *argv[]);'
        quote 'int main(int argc, char *argv[]){return sim_main(argc,argv);}'
        
        ####################################################################
        #    Declarations.  Do not change/remove/delete any of these       #
        ####################################################################
        
        ##### Symbolic representation of Y86-64 Instruction Codes #############
        wordsig INOP 	'I_NOP'
        wordsig IHALT	'I_HALT'
        wordsig IRRMOVQ	'I_RRMOVQ'
        wordsig IIRMOVQ	'I_IRMOVQ'
        wordsig IRMMOVQ	'I_RMMOVQ'
        wordsig IMRMOVQ	'I_MRMOVQ'
        wordsig IOPQ	'I_ALU'
        wordsig IJXX	'I_JMP'
        wordsig ICALL	'I_CALL'
        wordsig IRET	'I_RET'
        wordsig IPUSHQ	'I_PUSHQ'
        wordsig IPOPQ	'I_POPQ'
        # Instruction code for iaddq instruction
        wordsig IIADDQ	'I_IADDQ'
        
        ##### Symbolic represenations of Y86-64 function codes            #####
        wordsig FNONE    'F_NONE'        # Default function code
        
        ##### Symbolic representation of Y86-64 Registers referenced      #####
        wordsig RRSP     'REG_RSP'    	     # Stack Pointer
        wordsig RNONE    'REG_NONE'   	     # Special value indicating "no register"
        
        ##### ALU Functions referenced explicitly ##########################
        wordsig ALUADD	'A_ADD'		     # ALU should add its arguments
        
        ##### Possible instruction status values                       #####
        wordsig SBUB	'STAT_BUB'	# Bubble in stage
        wordsig SAOK	'STAT_AOK'	# Normal execution
        wordsig SADR	'STAT_ADR'	# Invalid memory address
        wordsig SINS	'STAT_INS'	# Invalid instruction
        wordsig SHLT	'STAT_HLT'	# Halt instruction encountered
        
        ##### Signals that can be referenced by control logic ##############
        
        ##### Pipeline Register F ##########################################
        
        wordsig F_predPC 'pc_curr->pc'	     # Predicted value of PC
        
        ##### Intermediate Values in Fetch Stage ###########################
        
        wordsig imem_icode  'imem_icode'      # icode field from instruction memory
        wordsig imem_ifun   'imem_ifun'       # ifun  field from instruction memory
        wordsig f_icode	'if_id_next->icode'  # (Possibly modified) instruction code
        wordsig f_ifun	'if_id_next->ifun'   # Fetched instruction function
        wordsig f_valC	'if_id_next->valc'   # Constant data of fetched instruction
        wordsig f_valP	'if_id_next->valp'   # Address of following instruction
        boolsig imem_error 'imem_error'	     # Error signal from instruction memory
        boolsig instr_valid 'instr_valid'    # Is fetched instruction valid?
        
        ##### Pipeline Register D ##########################################
        wordsig D_icode 'if_id_curr->icode'   # Instruction code
        wordsig D_rA 'if_id_curr->ra'	     # rA field from instruction
        wordsig D_rB 'if_id_curr->rb'	     # rB field from instruction
        wordsig D_valP 'if_id_curr->valp'     # Incremented PC
        
        ##### Intermediate Values in Decode Stage  #########################
        
        wordsig d_srcA	 'id_ex_next->srca'  # srcA from decoded instruction
        wordsig d_srcB	 'id_ex_next->srcb'  # srcB from decoded instruction
        wordsig d_rvalA 'd_regvala'	     # valA read from register file
        wordsig d_rvalB 'd_regvalb'	     # valB read from register file
        
        ##### Pipeline Register E ##########################################
        wordsig E_icode 'id_ex_curr->icode'   # Instruction code
        wordsig E_ifun  'id_ex_curr->ifun'    # Instruction function
        wordsig E_valC  'id_ex_curr->valc'    # Constant data
        wordsig E_srcA  'id_ex_curr->srca'    # Source A register ID
        wordsig E_valA  'id_ex_curr->vala'    # Source A value
        wordsig E_srcB  'id_ex_curr->srcb'    # Source B register ID
        wordsig E_valB  'id_ex_curr->valb'    # Source B value
        wordsig E_dstE 'id_ex_curr->deste'    # Destination E register ID
        wordsig E_dstM 'id_ex_curr->destm'    # Destination M register ID
        
        ##### Intermediate Values in Execute Stage #########################
        wordsig e_valE 'ex_mem_next->vale'	# valE generated by ALU
        boolsig e_Cnd 'ex_mem_next->takebranch' # Does condition hold?
        wordsig e_dstE 'ex_mem_next->deste'      # dstE (possibly modified to be RNONE)
        
        ##### Pipeline Register M                  #########################
        wordsig M_stat 'ex_mem_curr->status'     # Instruction status
        wordsig M_icode 'ex_mem_curr->icode'	# Instruction code
        wordsig M_ifun  'ex_mem_curr->ifun'	# Instruction function
        wordsig M_valA  'ex_mem_curr->vala'      # Source A value
        wordsig M_dstE 'ex_mem_curr->deste'	# Destination E register ID
        wordsig M_valE  'ex_mem_curr->vale'      # ALU E value
        wordsig M_dstM 'ex_mem_curr->destm'	# Destination M register ID
        boolsig M_Cnd 'ex_mem_curr->takebranch'	# Condition flag
        boolsig dmem_error 'dmem_error'	        # Error signal from instruction memory
        
        ##### Intermediate Values in Memory Stage ##########################
        wordsig m_valM 'mem_wb_next->valm'	# valM generated by memory
        wordsig m_stat 'mem_wb_next->status'	# stat (possibly modified to be SADR)
        
        ##### Pipeline Register W ##########################################
        wordsig W_stat 'mem_wb_curr->status'     # Instruction status
        wordsig W_icode 'mem_wb_curr->icode'	# Instruction code
        wordsig W_dstE 'mem_wb_curr->deste'	# Destination E register ID
        wordsig W_valE  'mem_wb_curr->vale'      # ALU E value
        wordsig W_dstM 'mem_wb_curr->destm'	# Destination M register ID
        wordsig W_valM  'mem_wb_curr->valm'	# Memory M value
        
        ####################################################################
        #    Control Signal Definitions.                                   #
        ####################################################################
        
        ################ Fetch Stage     ###################################
        
        ## What address should instruction be fetched at
        word f_pc = [
        	# Mispredicted branch.  Fetch at incremented PC
        	M_icode == IJXX && !M_Cnd : M_valA;
        	# Completion of RET instruction
        	W_icode == IRET : W_valM;
        	# Default: Use predicted value of PC
        	1 : F_predPC;
        ];
        
        ## Determine icode of fetched instruction
        word f_icode = [
        	imem_error : INOP;
        	1: imem_icode;
        ];
        
        # Determine ifun
        word f_ifun = [
        	imem_error : FNONE;
        	1: imem_ifun;
        ];
        
        # Is instruction valid?
        bool instr_valid = f_icode in 
        	{ INOP, IHALT, IRRMOVQ, IIRMOVQ, IRMMOVQ, IMRMOVQ,
        	  IOPQ, IJXX, ICALL, IRET, IPUSHQ, IPOPQ, IIADDQ };
        
        # Determine status code for fetched instruction
        word f_stat = [
        	imem_error: SADR;
        	!instr_valid : SINS;
        	f_icode == IHALT : SHLT;
        	1 : SAOK;
        ];
        
        # Does fetched instruction require a regid byte?
        bool need_regids =
        	f_icode in { IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, 
        		     IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ };
        
        # Does fetched instruction require a constant word?
        bool need_valC =
        	f_icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL, IIADDQ };
        
        # Predict next value of PC
        word f_predPC = [
        	f_icode in { IJXX, ICALL } : f_valC;
        	1 : f_valP;
        ];
        
        ################ Decode Stage ######################################
        
        ## What register should be used as the A source?
        word d_srcA = [
        	D_icode in { IRRMOVQ, IRMMOVQ, IOPQ, IPUSHQ  } : D_rA;
        	D_icode in { IPOPQ, IRET } : RRSP;
        	1 : RNONE; # Don't need register
        ];
        
        ## What register should be used as the B source?
        word d_srcB = [
        	D_icode in { IOPQ, IRMMOVQ, IMRMOVQ, IIADDQ } : D_rB;
        	D_icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        	1 : RNONE;  # Don't need register
        ];
        
        ## What register should be used as the E destination?
        word d_dstE = [
        	D_icode in { IRRMOVQ, IIRMOVQ, IOPQ, IIADDQ } : D_rB;
        	D_icode in { IPUSHQ, IPOPQ, ICALL, IRET } : RRSP;
        	1 : RNONE;  # Don't write any register
        ];
        
        ## What register should be used as the M destination?
        word d_dstM = [
        	D_icode in { IMRMOVQ, IPOPQ } : D_rA;
        	1 : RNONE;  # Don't write any register
        ];
        
        ## What should be the A value?
        ## Forward into decode stage for valA
        word d_valA = [
        	D_icode in { ICALL, IJXX } : D_valP; # Use incremented PC
        	d_srcA == e_dstE : e_valE;    # Forward valE from execute
        	d_srcA == M_dstM : m_valM;    # Forward valM from memory
        	d_srcA == M_dstE : M_valE;    # Forward valE from memory
        	d_srcA == W_dstM : W_valM;    # Forward valM from write back
        	d_srcA == W_dstE : W_valE;    # Forward valE from write back
        	1 : d_rvalA;  # Use value read from register file
        ];
        
        word d_valB = [
        	d_srcB == e_dstE : e_valE;    # Forward valE from execute
        	d_srcB == M_dstM : m_valM;    # Forward valM from memory
        	d_srcB == M_dstE : M_valE;    # Forward valE from memory
        	d_srcB == W_dstM : W_valM;    # Forward valM from write back
        	d_srcB == W_dstE : W_valE;    # Forward valE from write back
        	1 : d_rvalB;  # Use value read from register file
        ];
        
        ################ Execute Stage #####################################
        
        ## Select input A to ALU
        word aluA = [
        	E_icode in { IRRMOVQ, IOPQ } : E_valA;
        	E_icode in { IIRMOVQ, IRMMOVQ, IMRMOVQ, IIADDQ } : E_valC;
        	E_icode in { ICALL, IPUSHQ } : -8;
        	E_icode in { IRET, IPOPQ } : 8;
        	# Other instructions don't need ALU
        ];
        
        ## Select input B to ALU
        word aluB = [
        	E_icode in { IRMMOVQ, IMRMOVQ, IOPQ, ICALL, 
        		     IPUSHQ, IRET, IPOPQ, IIADDQ } : E_valB;
        	E_icode in { IRRMOVQ, IIRMOVQ } : 0;
        	# Other instructions don't need ALU
        ];
        
        ## Set the ALU function
        word alufun = [
        	E_icode == IOPQ : E_ifun;
        	1 : ALUADD;
        ];
        
        ## Should the condition codes be updated?
        bool set_cc = (E_icode == IOPQ || E_icode == IIADDQ) &&
        	# State changes only during normal operation
        	!m_stat in { SADR, SINS, SHLT } && !W_stat in { SADR, SINS, SHLT };
        
        ## Generate valA in execute stage
        word e_valA = E_valA;    # Pass valA through stage
        
        ## Set dstE to RNONE in event of not-taken conditional move
        word e_dstE = [
        	E_icode == IRRMOVQ && !e_Cnd : RNONE;
        	1 : E_dstE;
        ];
        
        ################ Memory Stage ######################################
        
        ## Select memory address
        word mem_addr = [
        	M_icode in { IRMMOVQ, IPUSHQ, ICALL, IMRMOVQ } : M_valE;
        	M_icode in { IPOPQ, IRET } : M_valA;
        	# Other instructions don't need address
        ];
        
        ## Set read control signal
        bool mem_read = M_icode in { IMRMOVQ, IPOPQ, IRET };
        
        ## Set write control signal
        bool mem_write = M_icode in { IRMMOVQ, IPUSHQ, ICALL };
        
        #/* $begin pipe-m_stat-hcl */
        ## Update the status
        word m_stat = [
        	dmem_error : SADR;
        	1 : M_stat;
        ];
        #/* $end pipe-m_stat-hcl */
        
        ## Set E port register ID
        word w_dstE = W_dstE;
        
        ## Set E port value
        word w_valE = W_valE;
        
        ## Set M port register ID
        word w_dstM = W_dstM;
        
        ## Set M port value
        word w_valM = W_valM;
        
        ## Update processor status
        word Stat = [
        	W_stat == SBUB : SAOK;
        	1 : W_stat;
        ];
        
        ################ Pipeline Register Control #########################
        
        # Should I stall or inject a bubble into Pipeline Register F?
        # At most one of these can be true.
        bool F_bubble = 0;
        bool F_stall =
        	# Conditions for a load/use hazard
        	E_icode in { IMRMOVQ, IPOPQ } &&
        	 E_dstM in { d_srcA, d_srcB } ||
        	# Stalling at fetch while ret passes through pipeline
        	IRET in { D_icode, E_icode, M_icode };
        
        # Should I stall or inject a bubble into Pipeline Register D?
        # At most one of these can be true.
        bool D_stall = 
        	# Conditions for a load/use hazard
        	E_icode in { IMRMOVQ, IPOPQ } &&
        	 E_dstM in { d_srcA, d_srcB };
        
        bool D_bubble =
        	# Mispredicted branch
        	(E_icode == IJXX && !e_Cnd) ||
        	# Stalling at fetch while ret passes through pipeline
        	# but not condition for a load/use hazard
        	!(E_icode in { IMRMOVQ, IPOPQ } && E_dstM in { d_srcA, d_srcB }) &&
        	  IRET in { D_icode, E_icode, M_icode };
        
        # Should I stall or inject a bubble into Pipeline Register E?
        # At most one of these can be true.
        bool E_stall = 0;
        bool E_bubble =
        	# Mispredicted branch
        	(E_icode == IJXX && !e_Cnd) ||
        	# Conditions for a load/use hazard
        	E_icode in { IMRMOVQ, IPOPQ } &&
        	 E_dstM in { d_srcA, d_srcB};
        
        # Should I stall or inject a bubble into Pipeline Register M?
        # At most one of these can be true.
        bool M_stall = 0;
        # Start injecting bubbles as soon as exception passes through memory stage
        bool M_bubble = m_stat in { SADR, SINS, SHLT } || W_stat in { SADR, SINS, SHLT };
        
        # Should I stall or inject a bubble into Pipeline Register W?
        bool W_stall = W_stat in { SADR, SINS, SHLT };
        bool W_bubble = 0;
        #/* $end pipe-all-hcl */
        
        ```
        
    - rewrite the `ncopy.ys` to add the `iaddq` instruction
      
        ```nasm
        #/* $begin ncopy-ys */
        ##################################################################
        # ncopy.ys - Copy a src block of len words to dst.
        # Return the number of positive words (>0) contained in src.
        #
        # Include your name and ID here.
        #
        # Describe how and why you modified the baseline code.
        #
        ##################################################################
        # Do not modify this portion
        # Function prologue.
        # %rdi = src, %rsi = dst, %rdx = len
        ncopy:
        
        ##################################################################
        # You can modify this portion
        	# Loop header
        	xorq %rax,%rax		# count = 0;
        	andq %rdx,%rdx		# len <= 0?
        	jle Done		# if so, goto Done:
        
        Loop:	mrmovq (%rdi), %r10	# read val from src...
        	rmmovq %r10, (%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Npos		# if so, goto Npos:
        	iaddq $1, %rax		# count++
        Npos:	
        	iaddq $-1, %rdx		# len--
        	iaddq $8, %rdi		# src++
        	iaddq $8, %rsi		# dst++
        	andq %rdx,%rdx		# len > 0?
        	jg Loop			# if so, goto Loop:
        ##################################################################
        # Do not modify the following section of code
        # Function epilogue.
        Done:
        	ret
        ##################################################################
        # Keep the following label at the end of your function
        End:
        #/* $end ncopy-ys */
        
        ```
        
    1. **Result 1 (only use iaddq)**
       
        ```
                ncopy
        0       13
        1       27      27.00
        2       40      20.00
        3       50      16.67
        4       63      15.75
        5       73      14.60
        6       86      14.33
        7       96      13.71
        8       109     13.62
        9       119     13.22
        10      132     13.20
        11      142     12.91
        12      155     12.92
        13      165     12.69
        14      178     12.71
        15      188     12.53
        16      201     12.56
        17      211     12.41
        18      224     12.44
        19      234     12.32
        20      247     12.35
        21      257     12.24
        22      270     12.27
        23      280     12.17
        24      293     12.21
        25      303     12.12
        26      316     12.15
        27      326     12.07
        28      339     12.11
        29      349     12.03
        30      362     12.07
        31      372     12.00
        32      385     12.03
        33      395     11.97
        34      408     12.00
        35      418     11.94
        36      431     11.97
        37      441     11.92
        38      454     11.95
        39      464     11.90
        40      477     11.93
        41      487     11.88
        42      500     11.90
        43      510     11.86
        44      523     11.89
        45      533     11.84
        46      546     11.87
        47      556     11.83
        48      569     11.85
        49      579     11.82
        50      592     11.84
        51      602     11.80
        52      615     11.83
        53      625     11.79
        54      638     11.81
        55      648     11.78
        56      661     11.80
        57      671     11.77
        58      684     11.79
        59      694     11.76
        60      707     11.78
        61      717     11.75
        62      730     11.77
        63      740     11.75
        64      753     11.77
        Average CPE     12.70
        Score   0.0/60.0
        ```
        
        ***(CPE=12.70 Score=0.0/60)***
        
    2. **Result 2** 
       
        Using six-way loop unrolling, 6*6, the code is as follows.
        
        ```nasm
        #/* $begin ncopy-ys */
        ##################################################################
        # ncopy.ys - Copy a src block of len words to dst.
        # Return the number of positive words (>0) contained in src.
        #
        # Include your name and ID here.
        #
        # Describe how and why you modified the baseline code.
        #
        ##################################################################
        # Do not modify this portion
        # Function prologue.
        # %rdi = src, %rsi = dst, %rdx = len
        ncopy:
        
        ##################################################################
        # You can modify this portion
        	# Loop header
            #xorq %rax, %rax      # count = 0
            andq %rdx, %rdx      # len <= 0?
            jle Done             # if so, return
        
            # Calculate iterations for 6-way unrolling
            rrmovq %rdx, %r8     # Copy len
            iaddq $-6, %r8       # len - 6
            jl RemainderLoop     # Jump if less than 6 elements
        
        UnrolledLoop:
            # Load 6 elements at once
            mrmovq (%rdi), %r10
            mrmovq 8(%rdi), %r11
            mrmovq 16(%rdi), %r12
            mrmovq 24(%rdi), %r13
            mrmovq 32(%rdi), %r14
            mrmovq 40(%rdi), %r9
        
            # Store 6 elements
            rmmovq %r10, (%rsi)
            rmmovq %r11, 8(%rsi)
            rmmovq %r12, 16(%rsi)
            rmmovq %r13, 24(%rsi)
            rmmovq %r14, 32(%rsi)
            rmmovq %r9, 40(%rsi)
        
            # Check and count positive numbers
            andq %r10, %r10
            jle L1
            iaddq $1, %rax
        L1: andq %r11, %r11
            jle L2
            iaddq $1, %rax
        L2: andq %r12, %r12
            jle L3
            iaddq $1, %rax
        L3: andq %r13, %r13
            jle L4
            iaddq $1, %rax
        L4: andq %r14, %r14
            jle L5
            iaddq $1, %rax
        L5: andq %r9, %r9
            jle L6
            iaddq $1, %rax
        
        L6: # Update pointers and counter
            iaddq $48, %rdi      # src += 6*8
            iaddq $48, %rsi      # dst += 6*8
            iaddq $-6, %rdx      # len -= 6
            
            # Check if we can do another full iteration
            rrmovq %rdx, %r8
            iaddq $-6, %r8
            jge UnrolledLoop     # If len >= 6, continue unrolled loop
        
        RemainderLoop:
            andq %rdx, %rdx      # Check remaining length
            jle Done             # If done, return
        
            # Handle remaining elements one by one
            mrmovq (%rdi), %r10
            rmmovq %r10, (%rsi)
            andq %r10, %r10
            jle Skip
            iaddq $1, %rax
        Skip:
            iaddq $8, %rdi
            iaddq $8, %rsi
            iaddq $-1, %rdx
            jg RemainderLoop
        
        ##################################################################
        # Do not modify the following section of code
        # Function epilogue.
        Done:
        	ret
        ##################################################################
        # Keep the following label at the end of your function
        End:
        #/* $end ncopy-ys */
        
        ```
        
        ***(CPE=9.33 Score=23.4/60)***
        
    3. **Result 3**
       
        Using eight-way loop unrolling, 8*8, the code is as follows.
        
        ```nasm
        #/* $begin ncopy-ys */
        ##################################################################
        # ncopy.ys - Copy a src block of len words to dst.
        # Return the number of positive words (>0) contained in src.
        #
        # Include your name and ID here.
        #
        # Describe how and why you modified the baseline code.
        #
        ##################################################################
        # Do not modify this portion
        # Function prologue.
        # %rdi = src, %rsi = dst, %rdx = len
        ncopy:
        
        ##################################################################
        # You can modify this portion
        	# Loop header
            #xorq %rax, %rax      # count = 0
        ##################################################################
        # Optimized ncopy.ys 
        # Optimizations applied:
        # 1. Loop unrolling (6-way)
        # 2. Register use optimization
        # 3. Condition code optimization
        # 4. Branch prediction optimization
        ##################################################################
        
        ncopy:
            # Setup and edge case handling
            #xorq %rax, %rax      # count = 0
            andq %rdx, %rdx      # len <= 0?
            jle Done             # if so, return
        
            # Direct length check for 8-way unrolling
            iaddq $-8, %rdx      # len - 8
            jl RemainderStart    # if len < 8, handle remainder
        
        UnrolledLoop:
            # Load 8 elements
            mrmovq (%rdi), %r8
            mrmovq 8(%rdi), %r9
            mrmovq 16(%rdi), %r10
            mrmovq 24(%rdi), %r11
            mrmovq 32(%rdi), %r12
            mrmovq 40(%rdi), %r13
            mrmovq 48(%rdi), %r14
            mrmovq 56(%rdi), %rcx
        
            # Store 8 elements
            rmmovq %r8, (%rsi)
            rmmovq %r9, 8(%rsi)
            rmmovq %r10, 16(%rsi)
            rmmovq %r11, 24(%rsi)
            rmmovq %r12, 32(%rsi)
            rmmovq %r13, 40(%rsi)
            rmmovq %r14, 48(%rsi)
            rmmovq %rcx, 56(%rsi)
        
            # Check positive numbers and count
            andq %r8, %r8
            jle L1
            iaddq $1, %rax
        L1: 
            andq %r9, %r9
            jle L2
            iaddq $1, %rax
        L2: 
            andq %r10, %r10
            jle L3
            iaddq $1, %rax
        L3: 
            andq %r11, %r11
            jle L4
            iaddq $1, %rax
        L4: 
            andq %r12, %r12
            jle L5
            iaddq $1, %rax
        L5: 
            andq %r13, %r13
            jle L6
            iaddq $1, %rax
        L6: 
            andq %r14, %r14
            jle L7
            iaddq $1, %rax
        L7: 
            andq %rcx, %rcx
            jle L8
            iaddq $1, %rax
        L8:
            # Update pointers and check for more iterations
            iaddq $64, %rdi      # src += 8*8
            iaddq $64, %rsi      # dst += 8*8
            iaddq $-8, %rdx      # len -= 8
            jge UnrolledLoop     # if len >= 8, continue
        
        RemainderStart:
            # Restore last subtraction to get remaining length
            iaddq $8, %rdx
            je Done              # if len == 0, done
        
        RemainderLoop:
            mrmovq (%rdi), %r8
            rmmovq %r8, (%rsi)
            andq %r8, %r8
            jle RemainderNext
            iaddq $1, %rax
        
        RemainderNext:
            iaddq $8, %rdi
            iaddq $8, %rsi
            iaddq $-1, %rdx
            jg RemainderLoop
        
        ##################################################################
        # Do not modify the following section of code
        # Function epilogue.
        Done:
        	ret
        ##################################################################
        # Keep the following label at the end of your function
        End:
        #/* $end ncopy-ys */
        
        ```
        
        ***(CPE=8.66 Score=36.9/60)***
        
    4. **Result 4 （reference** [https://www.bilibili.com/opus/810315412273102883](https://www.bilibili.com/opus/810315412273102883)**）**
       
        Using **84321**
        
        ```nasm
        #/* $begin ncopy-ys */
        ##################################################################
        # ncopy.ys - Copy a src block of len words to dst.
        # Return the number of positive words (>0) contained in src.
        #
        # Include your name and ID here.
        #
        # Describe how and why you modified the baseline code.
        #
        ##################################################################
        # Do not modify this portion
        # Function prologue.
        # %rdi = src, %rsi = dst, %rdx = len
        ncopy:
        
        ##################################################################
        # You can modify this portion
        
                 # Loop header
        
                 #xorq %rax,%rax           # count = 0;
        
                 iaddq $-8,%rdx
        
                 jl L4
        
        Loop8:
        
                 mrmovq (%rdi), %r10        # read val from src...
        
                 mrmovq 8(%rdi), %r11       # read val from src...
        
                 mrmovq 16(%rdi), %r12      # read val from src...
        
                 mrmovq 24(%rdi), %r13      # read val from src...
        
                 mrmovq 32(%rdi), %r14      # read val from src...
        
                 mrmovq 40(%rdi), %r8       # read val from src...
        
                 mrmovq 48(%rdi), %r9       # read val from src...
        
                 mrmovq 56(%rdi), %rbp      # read val from src...
        
                 rmmovq %r10, (%rsi)        # ...and store it to dst
        
                 rmmovq %r11, 8(%rsi)       # ...and store it to dst
        
                 rmmovq %r12, 16(%rsi)      # ...and store it to dst
        
                 rmmovq %r13, 24(%rsi)      # ...and store it to dst
        
                 rmmovq %r14, 32(%rsi)      # ...and store it to dst
        
                 rmmovq %r8, 40(%rsi)       # ...and store it to dst
        
                 rmmovq %r9, 48(%rsi)       # ...and store it to dst
        
                 rmmovq %rbp, 56(%rsi)      # ...and store it to dst
        
                 andq %r10, %r10            # val &lt;= 0?
        
                 jle n1
        
                 iaddq $1,%rax     # count++
        
        n1:
        
                 andq %r11, %r11            # val &lt;= 0?
        
                 jle n2
        
                 iaddq $1,%rax     # count++
        
        n2:
        
                 andq %r12, %r12            # val &lt;= 0?
        
                 jle n3
        
                 iaddq $1,%rax     # count++
        
        n3:
        
                 andq %r13, %r13            # val &lt;= 0?
        
                 jle n4
        
                 iaddq $1,%rax     # count++
        
        n4:
        
                 andq %r14, %r14            # val &lt;= 0?
        
                 jle n5
        
                 iaddq $1,%rax     # count++
        
        n5:
        
                 andq %r8, %r8              # val &lt;= 0?
        
                 jle n6
        
                 iaddq $1,%rax     # count++
        
        n6:
        
                 andq %r9, %r9              # val &lt;= 0?
        
                 jle n7
        
                 iaddq $1,%rax     # count++
        
        n7:
        
                 andq %rbp, %rbp            # val &lt;= 0?
        
                 jle n8
        
                 iaddq $1,%rax     # count++
        
        n8:
        
                 iaddq $64,%rdi
        
                 iaddq $64,%rsi
        
                 iaddq $-8,%rdx
        
                 jge Loop8         #
        
        L4:
        
                 iaddq $4,%rdx
        
                 jl L3             # if so, goto Done:
        
        Loop4:
        
                 mrmovq (%rdi), %r10        # read val from src...
        
                 mrmovq 8(%rdi), %r11       # read val from src...
        
                 mrmovq 16(%rdi), %r12      # read val from src...
        
                 mrmovq 24(%rdi), %r13      # read val from src...
        
                 rmmovq %r10, (%rsi)        # ...and store it to dst
        
                 rmmovq %r11, 8(%rsi)       # ...and store it to dst
        
                 rmmovq %r12, 16(%rsi)      # ...and store it to dst
        
                 rmmovq %r13, 24(%rsi)      # ...and store it to dst
        
                 andq %r10, %r10            # val &lt;= 0?
        
                 jle n41
        
                 iaddq $1,%rax     # count++
        
        n41:
        
                 andq %r11, %r11            # val &lt;= 0?
        
                 jle n42
        
                 iaddq $1,%rax     # count++
        
        n42:
        
                 andq %r12, %r12            # val &lt;= 0?
        
                 jle n43
        
                 iaddq $1,%rax     # count++
        
        n43:
        
                 andq %r13, %r13            # val &lt;= 0?
        
                 jle n44
        
                 iaddq $1,%rax     # count++
        
        n44:
        
                 iaddq $32,%rdi
        
                 iaddq $32,%rsi
        
                 iaddq $-4,%rdx
        
        L3:
        
                 iaddq $1,%rdx
        
                 jl L2
        
                 mrmovq (%rdi), %r10        # read val from src...
        
                 mrmovq 8(%rdi), %r11       # read val from src...
        
                 mrmovq 16(%rdi), %r12      # read val from src...
        
                 rmmovq %r10, (%rsi)        # ...and store it to dst
        
                 rmmovq %r11, 8(%rsi)       # ...and store it to dst
        
                 rmmovq %r12, 16(%rsi)      # ...and store it to dst
        
                 andq %r10, %r10            # val &lt;= 0?
        
                 jle N31           # if so, goto Npos:
        
                 iaddq $1,%rax     # count++
        
        N31:
        
                 andq %r11,%r11
        
                 jle N32
        
                 iaddq $1,%rax
        
        N32:
        
                 andq %r12,%r12
        
                 jle Done
        
                 iaddq $1,%rax
        
                 ret
        
        L2:
        
                 iaddq $1,%rdx
        
                 jl L1
        
                 mrmovq (%rdi), %r10        # read val from src...
        
                 mrmovq 8(%rdi), %r11       # read val from src...
        
                 rmmovq %r10, (%rsi)        # ...and store it to dst
        
                 rmmovq %r11, 8(%rsi)       # ...and store it to dst
        
                 andq %r10, %r10            # val &lt;= 0?
        
                 jle N21           # if so, goto Npos:
        
                 iaddq $1,%rax     # count++
        
        N21:
        
                 andq %r11,%r11
        
                 jle Done
        
                 iaddq $1,%rax
        
                 ret
        
        L1:
        
                 iaddq $1,%rdx
        
                 je N11
        
                 ret
        
        N11:
        
                 mrmovq (%rdi), %r10        # read val from src...
        
                 rmmovq %r10, (%rsi)        # ...and store it to dst
        
                 andq %r10, %r10            # val &lt;= 0?
        
                 jle Done          # if so, goto Npos:
        
                 iaddq $1,%rax     # count++
        
        ##################################################################
        # Do not modify the following section of code
        # Function epilogue.
        Done:
        	ret
        ##################################################################
        # Keep the following label at the end of your function
        End:
        #/* $end ncopy-ys */
        
        ```
        
        ***(CPE=7.60 Score=58/60)***
        
    5. Result 5 (reference:[https://zhuanlan.zhihu.com/p/77072339](https://zhuanlan.zhihu.com/p/77072339))
       
        ```nasm
        #/* $begin ncopy-ys */
        ##################################################################
        # ncopy.ys - Copy a src block of len words to dst.
        # Return the number of positive words (>0) contained in src.
        #
        # Include your name and ID here.
        #
        # Describe how and why you modified the baseline code.
        #
        ##################################################################
        # Do not modify this portion
        # Function prologue.
        # %rdi = src, %rsi = dst, %rdx = len
        ncopy:
        
        ##################################################################
        # You can modify this portion
        	# Loop header
        	iaddq $-10,%rdx		# len < 10?
        	jl Root			# if so, goto Root:
        
        Loop1:	mrmovq (%rdi), %r10	# read val from src...
        	mrmovq 8(%rdi), %r11	# read val from src...
        	rmmovq %r10, (%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Loop2		# if so, goto Loop2:
        	iaddq $0x1, %rax		# count++
        Loop2:	mrmovq 16(%rdi), %r10	# read val from src...
        	rmmovq %r11, 8(%rsi)	# ...and store it to dst
        	andq %r11, %r11		# val <= 0?
        	jle Loop3		# if so, goto Loop3:
        	iaddq $0x1, %rax		# count++
        Loop3:	mrmovq 24(%rdi), %r11	# read val from src...
        	rmmovq %r10, 16(%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Loop4		# if so, goto Loop4:
        	iaddq $0x1, %rax		# count++
        Loop4:	mrmovq 32(%rdi), %r10	# read val from src...
        	rmmovq %r11, 24(%rsi)	# ...and store it to dst
        	andq %r11, %r11		# val <= 0?
        	jle Loop5		# if so, goto Loop5:
        	iaddq $0x1, %rax		# count++
        Loop5:	mrmovq 40(%rdi), %r11	# read val from src...
        	rmmovq %r10, 32(%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Loop6		# if so, goto Loop6:
        	iaddq $0x1, %rax		# count++
        Loop6:	mrmovq 48(%rdi), %r10	# read val from src...
        	rmmovq %r11, 40(%rsi)	# ...and store it to dst
        	andq %r11, %r11		# val <= 0?
        	jle Loop7		# if so, goto Loop7:
        	iaddq $0x1, %rax		# count++
        Loop7:	mrmovq 56(%rdi), %r11	# read val from src...
        	rmmovq %r10, 48(%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Loop8		# if so, goto Loop8:
        	iaddq $0x1, %rax		# count++
        Loop8:	mrmovq 64(%rdi), %r10	# read val from src...
        	rmmovq %r11, 56(%rsi)	# ...and store it to dst
        	andq %r11, %r11		# val <= 0?
        	jle Loop9		# if so, goto Loop9:
        	iaddq $0x1, %rax		# count++
        Loop9:	mrmovq 72(%rdi), %r11	# read val from src...
        	rmmovq %r10, 64(%rsi)	# ...and store it to dst
        	andq %r10, %r10		# val <= 0?
        	jle Loop10		# if so, goto Loop10:
        	iaddq $0x1, %rax		# count++
        Loop10:	#mrmovq 64(%rdi), %r10	# read val from src...
        	rmmovq %r11, 72(%rsi)	# ...and store it to dst
        	andq %r11, %r11		# val <= 0?
        	jle Loop		# if so, goto Loop:
        	iaddq $0x1, %rax		# count++
        
        Loop:
        	iaddq $0x50, %rdi	# src++
        	iaddq $0x50, %rsi	# dst++
        	iaddq $-10,%rdx		# len >= 10?
        	jge Loop1		# if so, goto Loop1:
        Root:
        	iaddq	$7,%rdx		# len <= 3
        	jl	Left
        	jg	Right	
        	je	Remain3		# len == 3 Middle
        	
        
        Left:
        	iaddq	$2,%rdx		# len == 1
        	je	Remain1
        	iaddq	$-1,%rdx	# len == 2
        	je	Remain2
        	ret			# len == 0 
        Right:
        	iaddq	$-3,%rdx	# len <= 6 
        	jg	RightRight
        	je	Remain6		# len == 6
        	iaddq	$1,%rdx		# RightLeft
        	je	Remain5		# len == 5
        	jmp	Remain4		# len == 4
        	
        RightRight:
        	iaddq	$-2,%rdx
        	jl	Remain7
        	je	Remain8
        
        Remain9:
        	mrmovq 64(%rdi), %r11	# read val from src...
        	rmmovq %r11, 64(%rsi)
        	andq %r11, %r11		# val <= 0?
        
        Remain8:
        	mrmovq 56(%rdi), %r11	# read val from src...
        	jle Remain82		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        
        Remain82:
        	
        	rmmovq %r11, 56(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain7:
        	mrmovq 48(%rdi), %r11	# read val from src...
        	jle Remain72		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain72:
        		
        	rmmovq %r11, 48(%rsi)
        	andq %r11, %r11		# val <= 0?
        
        Remain6:
        	mrmovq 40(%rdi), %r11	# read val from src...
        	jle Remain62		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain62:
        		
        	rmmovq %r11, 40(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain5:
        	mrmovq 32(%rdi), %r11	# read val from src...
        	jle Remain52		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain52:
        		
        	rmmovq %r11, 32(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain4:
        	mrmovq 24(%rdi), %r11	# read val from src...
        	jle Remain42	# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain42:
        
        	rmmovq %r11, 24(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain3:
        	mrmovq 16(%rdi), %r11	# read val from src...
        	jle Remain32		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain32:
        
        	rmmovq %r11, 16(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain2:
        	mrmovq 8(%rdi), %r11	# read val from src...
        	jle Remain22		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain22:
        
        	rmmovq %r11, 8(%rsi)
        	andq %r11, %r11		# val <= 0?
        Remain1:
        	mrmovq (%rdi), %r11	# read val from src...
        	jle Remain12		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        Remain12:
        	
        	rmmovq %r11, (%rsi)
        	andq %r11, %r11		# val <= 0?
        	jle Done		# if so, goto Npos:
        	iaddq $0x1, %rax		# count++
        
        ##################################################################
        # Do not modify the following section of code
        # Function epilogue.
        Done:
        	ret
        ##################################################################
        # Keep the following label at the end of your function
        End:
        #/* $end ncopy-ys */
        
        ```
        
        ***(CPE=7.49 Score=60/60)***
        
        insight:
        
        - Improved Instruction Interleaving:
          
            ```nasm
            Loop1: 
            mrmovq (%rdi), %r10    # read 1st element
            mrmovq 8(%rdi), %r11   # prefetch 2nd element
            rmmovq %r10, (%rsi)    # store 1st element
            andq %r10, %r10        # check 1st element
            jle Loop2              # jump
            ...
            Loop2:
            mrmovq 16(%rdi), %r10  # prefetch 3rd element
            rmmovq %r11, 8(%rsi)   # store 2nd element
            andq %r11, %r11        # check 2nd element
            ```
            

---