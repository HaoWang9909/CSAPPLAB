# CacheLab

## `Learning Materials`

- Material 1
  
    [cachelab.pdf](CacheLab%201491ab65f22f805ca9d0fe7320f90717/cachelab.pdf)
    
- Material 2
  
    [12-cache-memories.pdf](CacheLab%201491ab65f22f805ca9d0fe7320f90717/12-cache-memories.pdf)
    



# **Objectives of the Lab**

1. Understand the impact of cache memory on program performance.
2. Implement a cache simulator.
3. Optimize a matrix transpose function to minimize cache misses. 

---

# **Structure of the Lab**

The lab is divided into two parts:

1. **Part A: Implementing a Cache Simulator**
    - Use the `csim.c` file to implement a cache simulator that reads memory access traces generated by Valgrind.
    - Simulate cache hit, miss, and eviction behaviors, and output these statistics.
    - The cache uses the **LRU (Least Recently Used)** replacement policy.
2. **Part B: Optimizing Matrix Transpose**
    - Use the trans.c file to implement an efficient matrix transpose function.
    - Aim to minimize cache misses and optimize performance for specific matrix sizes (e.g., 32×32, 64×64, and 61×67).

---

# Preparation

- Environment Setup:
  
    ```bash
     tar xvf cachelab-handout.tar
     make clean
     make
     sudo apt-get install valgrind
     valgrind --log-fd=1 --tool=lackey -v --trace-mem=yes ls -l
    ```
    

---

# Part A

## **Tasks**

- Simulate cache behavior, process memory access traces, and count hits, misses, and evictions.
- Support arbitrary cache sizes and associativity configurations.
  
    example：
    
    ```
    Usage: ./csim-ref [-hv] -s <s> -E <E> -b <b> -t <tracefile>
    • -h: Optional help flag that prints usage info
    • -v: Optional verbose flag that displays trace info
    • -s <s>: Number of set index bits (S = 2s
    is the number of sets)
    • -E <E>: Associativity (number of lines per set)
    • -b <b>: Number of block bits (B = 2b
    is the block size)
    • -t <tracefile>: Name of the valgrind trace to replay
    ```
    
    ```
    linux> ./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace
    hits:4 misses:5 evictions:3
    ```
    
    ```
    linux> ./csim-ref -v -s 4 -E 1 -b 4 -t traces/yi.trace
    L 10,1 miss
    M 20,1 miss hit
    L 22,1 hit
    S 18,1 hit
    L 110,1 miss eviction
    L 210,1 miss eviction
    M 12,1 miss eviction hit
    hits:4 misses:5 evictions:3
    ```
    

## **Input and Output**

- **Input**: Cache configuration (number of sets, lines per set, block size) and a trace file.
- **Output**: Statistics on cache hits, misses, and evictions.

## Evaluation（Total Point：27）

```bash
make clean && make ./test-csim
```

```
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     0 (1,1,1)       0       0       0       9       8       6  traces/yi2.trace
     0 (4,2,4)       0       0       0       4       5       2  traces/yi.trace
     0 (2,1,4)       0       0       0       2       3       1  traces/dave.trace
     0 (2,1,3)       0       0       0     167      71      67  traces/trans.trace
     0 (2,2,3)       0       0       0     201      37      29  traces/trans.trace
     0 (2,4,3)       0       0       0     212      26      10  traces/trans.trace
     1 (5,1,5)       0       0       0     231       7       0  traces/trans.trace
     0 (5,1,5)       0       0       0  265189   21775   21743  traces/long.trace
     1

***TEST_CSIM_RESULTS=1***
```

## Step

(reference:https://github.com/kcxain/CSAPP-Lab/blob/master/solutions/05_Cache%20Lab/csim.c)

1. include libs
   
    ```c
    # include "cachelab.h"
    # include <stdio.h>//include the standard input/output library
    # include <stdlib.h>//include the standard library
    # include <getopt.h>//include the getopt library
    # include <unistd.h>//include the unix standard library
    # include <string.h>//include the string library
    # include <stdbool.h>//include the bool library
    ```
    
2. define the cache structure
   
    ```c
    typedef struct cache_line
    {
        int valid;     // valid bit
        int tag;       // tag
        int time_tamp; // called time
    } Cache_line;
    ```
    
    ```c
    typedef struct cache_
    {
        int S;
        int E;
        int B;
        Cache_line **line;
    } Cache;
    ```
    
3. define the global variables:
   
    ```c
    int hit_count = 0, miss_count = 0, eviction_count = 0; // hit, miss, eviction count
    int verbose = 0;                                       // verbose flag
    char t[1000];
    Cache *cache = NULL;
    ```
    
4. Allocate and free memory:
   
    ```c
    void Init_Cache(int s, int E, int b)
    {
        int S = 1 << s;
        int B = 1 << b;
        cache = (Cache *)malloc(sizeof(Cache));
        cache->S = S;
        cache->E = E;
        cache->B = B;
        cache->line = (Cache_line **)malloc(sizeof(Cache_line *) * S);
        for (int i = 0; i < S; i++)
        {
            cache->line[i] = (Cache_line *)malloc(sizeof(Cache_line) * E);
            for (int j = 0; j < E; j++)
            {
                cache->line[i][j].valid = 0;
                cache->line[i][j].tag = -1;
                cache->line[i][j].time_tamp = 0;
            }
        }
    }
    ```
    
    ```c
    void free_Cache()
    {
        int S = cache->S;
        for (int i = 0; i < S; i++)
        {
            free(cache->line[i]);
        }
        free(cache->line);
        free(cache);
    }
    ```
    
5. Searches for a cache line in a given set that matches a specific tag.
   
    ```c
    int get_index(int op_s, int op_tag)
    {
        for (int i = 0; i < cache->E; i++)
        {
            if (cache->line[op_s][i].valid && cache->line[op_s][i].tag == op_tag)
                return i;
        }
        return -1;
    }
    ```
    
6. Finds the least recently used (LRU) line in a set for eviction.
   
    ```c
    int find_LRU(int op_s)
    {
        int max_index = 0;
        int max_stamp = 0;
        for(int i = 0; i < cache->E; i++){
            if(cache->line[op_s][i].time_tamp > max_stamp){
                max_stamp = cache->line[op_s][i].time_tamp;
                max_index = i;
            }
        }
        return max_index;
    }
    ```
    
7. Checks if there is an empty line in a set.
   
    ```c
    int is_full(int op_s)
    {
        for (int i = 0; i < cache->E; i++)
        {
            if (cache->line[op_s][i].valid == 0)
                return i;
        }
        return -1;
    }
    ```
    
8. Updates a cache line after an access, implementing the LRU policy.
   
    ```c
    void update(int i, int op_s, int op_tag){
        cache->line[op_s][i].valid=1;
        cache->line[op_s][i].tag = op_tag;
        for(int k = 0; k < cache->E; k++)
            if(cache->line[op_s][k].valid==1)
                cache->line[op_s][k].time_tamp++;
        cache->line[op_s][i].time_tamp = 0;
    }
    ```
    
9. Handles a cache access, updating counts and cache state.
   
    ```c
    void update_info(int op_tag, int op_s)
    {
        int index = get_index(op_s, op_tag);
        if (index == -1)
        {
            miss_count++;
            if (verbose)
                printf("miss ");
            int i = is_full(op_s);
            if(i==-1){
                eviction_count++;
                if(verbose) printf("eviction");
                i = find_LRU(op_s);
            }
            update(i,op_s,op_tag);
        }
        else{
            hit_count++;
            if(verbose)
                printf("hit");
            update(index,op_s,op_tag);    
        }
    }
    ```
    
10. Reads the trace file and simulates cache operations.
    
    ```c
    void get_trace(int s, int E, int b)
    {
        FILE *pFile;
        pFile = fopen(t, "r");
        if (pFile == NULL)
        {
            exit(-1);
        }
        char identifier;
        unsigned address;
        int size;
        while (fscanf(pFile, " %c %x,%d", &identifier, &address, &size) > 0) 
        {
           
            int op_tag = address >> (s + b);
            int op_s = (address >> b) & ((unsigned)(-1) >> (8 * sizeof(unsigned) - s));
            switch (identifier)
            {
            case 'M': 
                update_info(op_tag, op_s);
                update_info(op_tag, op_s);
                break;
            case 'L':
                update_info(op_tag, op_s);
                break;
            case 'S':
                update_info(op_tag, op_s);
                break;
            }
        }
        fclose(pFile);
    }
    ```
    
11. Prints the help message to guide users on how to use the program.
    
    ```c
    void print_help()
    {
        printf("** A Cache Simulator by Deconx\n");
        printf("Usage: ./csim-ref [-hv] -s <num> -E <num> -b <num> -t <file>\n");
        printf("Options:\n");
        printf("-h         Print this help message.\n");
        printf("-v         Optional verbose flag.\n");
        printf("-s <num>   Number of set index bits.\n");
        printf("-E <num>   Number of lines per set.\n");
        printf("-b <num>   Number of block offset bits.\n");
        printf("-t <file>  Trace file.\n\n\n");
        printf("Examples:\n");
        printf("linux>  ./csim -s 4 -E 1 -b 4 -t traces/yi.trace\n");
        printf("linux>  ./csim -v -s 8 -E 2 -b 4 -t traces/yi.trace\n");
    }
    ```
    
12.  The main function orchestrates the flow of the program.
        
    ```c
    int main(int argc, char *argv[])
    {
        char opt;
        int s, E, b;
        
        while (-1 != (opt = getopt(argc, argv, "hvs:E:b:t:")))
        {
            switch (opt)
            {
            case 'h':
                print_help();
                exit(0);
            case 'v':
                verbose = 1;
                break;
            case 's':
                s = atoi(optarg);
                break;
            case 'E':
                E = atoi(optarg);
                break;
            case 'b':
                b = atoi(optarg);
                break;
            case 't':
                strcpy(t, optarg);
                break;
            default:
                print_help();
                exit(-1);
            }
        }
        Init_Cache(s, E, b); 
        get_trace(s, E, b);
        free_Cache();
        printSummary(hit_count, miss_count, eviction_count);
        return 0;
    }
    ```
    
13. Evaluation
    
    ```c
                            Your simulator     Reference simulator
    Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
         3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
         3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
         3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
         3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
         3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
         3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
         3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
         6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
        27
    
    TEST_CSIM_RESULTS=27
    ```
    

# Part B

## **Tasks**

- Optimize the matrix transpose function transpose_submit to reduce cache misses.
- Focus on the given matrix sizes: **32×32**, **64×64**, and **61×67**.

## **Implementation Requirements**

- Declare at most **12 local integer variables**.
- Do not modify the input matrix A, but you can freely modify the output matrix B.
- Do not use arrays or dynamic memory allocation (malloc).

## Preparation

- we do a naive transpose and use `make clean && make && ./test-trans -M 32 -N 32` and we will find a trace file named `trace.f0`
- Then we use `./csim-ref -v -s 5 -E 1 -b 5 -t trace.f0 > output.txt` we can see the first few lines are like this:
  
    ```
    S 10d080,1 miss 
    L 18d0c0,8 miss 
    L 18d0a4,4 miss 
    L 18d0a0,4 hit 
    L 10d0a0,4 miss eviction 
    L 10d0a4,4 hit 
    L 10d0a8,4 hit 
    L 10d0ac,4 hit 
    L 10d0b0,4 hit 
    L 10d0b4,4 hit 
    L 10d0b8,4 hit 
    L 10d0bc,4 hit 
    S 14d0a0,4 miss eviction 
    S 14d120,4 miss 
    S 14d1a0,4 miss 
    S 14d220,4 miss 
    S 14d2a0,4 miss 
    S 14d320,4 miss 
    S 14d3a0,4 miss 
    S 14d420,4 miss 
    L 10d120,4 miss eviction 
    ```
    
    we pay attention to this part:
    
    ```
    L **10d0a0**,4 miss eviction 
    L 10d0a4,4 hit 
    L 10d0a8,4 hit 
    L 10d0ac,4 hit 
    L 10d0b0,4 hit 
    L 10d0b4,4 hit 
    L 10d0b8,4 hit 
    L 10d0bc,4 hit 
    S **14d0a0**,4 miss eviction 
    S 14d120,4 miss 
    S 14d1a0,4 miss 
    S 14d220,4 miss 
    S 14d2a0,4 miss 
    S 14d320,4 miss 
    S 14d3a0,4 miss 
    S 14d420,4 miss 
    ```
    
    so we can see that the address for A[0][0] is `0x10d0a0` and the B[0][0] is `0x14d0a0` so the difference is `0x040000` which is 262144 bytes and it can be divided by the data storage in the cache(s=5 E=1 b=5 which is 1024 bytes)! (This is very important!)
    
    Also we can see in `tracegen.c` that:
    
    ```
    static int A[256][256];
    static int B[256][256];
    ```
    
    and 262144=4*255*255. This is the same as the corresponding memory address.
    

## 32x32

1. Step 1: use blocking (Since s=5 E=1 b=5), so the cache structure is like this
   
    ```
    Set Index  | Tag  | Block (32 bytes)
    -----------|------|-----------------
    Set 0      | Tag0 | [Data 0-31]
    Set 1      | Tag1 | [Data 0-31]
    Set 2      | Tag2 | [Data 0-31]
    ...        | ...  | ...
    Set 31     | Tag31| [Data 0-31]
    ```
    
    and because it is used to store int data, so the structure to store the A matrix
    
    ```
    
    Set Index  | Tag  | Block (8 integers)
    -----------|------|---------------------
    Set 0      | Tag0 | [Int0, Int1, Int2, Int3, Int4, Int5, Int6, Int7]
    Set 1      | Tag1 | [Int0, Int1, Int2, Int3, Int4, Int5, Int6, Int7]
    Set 2      | Tag2 | [Int0, Int1, Int2, Int3, Int4, Int5, Int6, Int7]
    ...        | ...  | ...
    Set 31     | Tag31| [Int0, Int1, Int2, Int3, Int4, Int5, Int6, Int7]
    ```
    
    if we use the naive way to do the transpose, the miss will be 1152(32*32+32*4).
    
2. A would be store in cache like this:
   
    ![Screenshot 2024-11-25 at 22.18.39.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-25_at_22.18.39.png)
    
    This picture is more clear:
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image.png)
    
    This means that if we use an 8×8 block, the red parts won’t collide. For example, in A, the cache sets used to store the elements are 0, 4, 8, 12, 16, 20, 24, and 28. In B, the sets are 2, 6, 10, 14, 18, 22, 26, and 30, so there’s no collision. However, if they are on the diagonal, they will conflict.
    
    ![Screenshot 2024-11-25 at 22.21.36.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-25_at_22.21.36.png)
    
    so the first program should be like this:
    
    ```c
    int blockSize = 8;
    int i, j, ii, jj;
    for (i = 0; i < N; i += blockSize)
    {
        for (j = 0; j < M; j += blockSize)
        {
            for (ii = i; ii < i + blockSize && ii < N; ii++)
            {
                for (jj = j; jj < j + blockSize && jj < M; jj++)
                {
    		             B[jj][ii] = A[ii][jj];
                }
            }
        }
    }
    ```
    
    we can use `make clean && make && ./test-trans -M 32 -N 32` to see the result;
    
    ```
    Function 0 (2 total)
    Step 1: Validating and generating memory traces
    Step 2: Evaluating performance (s=5, E=1, b=5)
    func 0 (Transpose submission): hits:1709, misses:344, evictions:312
    
    Function 1 (2 total)
    Step 1: Validating and generating memory traces
    Step 2: Evaluating performance (s=5, E=1, b=5)
    func 1 (Simple row-wise scan transpose): hits:869, misses:1184, evictions:1152
    
    Summary for official submission (func 0): correctness=1 misses=344
    
    TEST_TRANS_RESULTS=1:344
    ```
    
    Our score is 344, but we need it to be under 300 to achieve a perfect score, so there’s room for improvement.
    
    Theoretically, our number of misses should be 256 (8 × 16 × 2). So there must be some conflics that we have not discovered yet.
    
    This is because conflicts occur when dealing with diagonal blocks. The cache used to store these two diagonal blocks ends up being in the same cache set.
    
    ![Screenshot 2024-11-26 at 18.43.46.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-26_at_18.43.46.png)
    
    so we can calculate the miss in one 8*8 step of transpose, the miss is 37 times. So the total miss should be 37*4(4diagonal blocks)+16*12=340, that is close to 344.
    
2. Step2 

By using 8 temporary variables to directly load one row of matrix A and assign it to matrix B, we can reduce memory misses in both A and B.

![Screenshot 2024-11-26 at 20.27.02.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-26_at_20.27.02.png)

So the code should be:

```c
int i, j, row_b;
int tmp0, tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7;
for (i = 0; i < 32; i += 8) {
        for (j = 0; j < 32; j += 8) {
            for (row_b = i; row_b < i + 8; ++ row_b) {

                tmp0 = A[row_b][j + 0];
                tmp1 = A[row_b][j + 1];
                tmp2 = A[row_b][j + 2];
                tmp3 = A[row_b][j + 3];
                tmp4 = A[row_b][j + 4];
                tmp5 = A[row_b][j + 5];
                tmp6 = A[row_b][j + 6];
                tmp7 = A[row_b][j + 7];
                B[j + 0][row_b] = tmp0;
                B[j + 1][row_b] = tmp1;
                B[j + 2][row_b] = tmp2;
                B[j + 3][row_b] = tmp3;
                B[j + 4][row_b] = tmp4;
                B[j + 5][row_b] = tmp5;
                B[j + 6][row_b] = tmp6;
		            B[j + 7][row_b] = tmp7;
                      }
  }
}
```

and the result now is:

```
Function 0 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:1765, misses:288, evictions:256

Function 1 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 1 (Simple row-wise scan transpose): hits:869, misses:1184, evictions:1152

Summary for official submission (func 0): correctness=1 misses=288

TEST_TRANS_RESULTS=1:288
```

1. Step3, however, it is still not the best, remember the best is 256, so we can still improve it. as we can see in the picture above, the B matrix still have some conflict, so what can we do?
   
    reference([https://www.zhihu.com/search?type=content&q=cache lab](https://www.zhihu.com/search?type=content&q=cache%20lab))
    
    To address conflict misses, the key is to manage repeated loads. After accessing a block, if the same block is accessed again but has already been evicted, a miss occurs during the second access. This happens because the tags of the two accesses are different. For example, consider a miss at index 22: when transposing the first row, matrix B’s second row is accessed. Later, matrix A also accesses the second row, and then matrix B re-accesses it. This second access to the second row results in a conflict miss.
    
    To avoid this, one option is to prevent matrix B from accessing the second row before matrix A finishes accessing it. Another option is to ensure matrix B does not access the second row after matrix A finishes accessing it. However, the latter is not feasible because B must write data after A reads it. Therefore, the solution lies in the former approach.
    
    Initially, we transpose all elements directly into matrix B, which inevitably causes matrix B’s second row to be accessed. Instead, can we delay transposing until matrix A’s next row is processed? This raises another problem: we need a place to store the 8 elements of the first row temporarily, but temporary variables are limited. However, we can write these elements into an unused portion of matrix B. Once matrix A’s next row is processed, the data can then be transposed to matrix B’s next row.
    
    This is feasible because, after the initial cold miss for B’s first row, subsequent accesses to that row will always hit in the cache, as matrix A no longer accesses the first row. Hence, we can temporarily store matrix A’s elements in matrix B’s first row, and once the next row of matrix A is read, we can proceed to access and transpose into matrix B’s next row. The code for this approach is as follows:
    
    ```
    if (M == 32)
    {
    	int i, j, x1, x2, x3, x4, x5, x6, x7, x8, x, y;
    	for (i = 0; i < N; i += 8)
    		for (j = 0; j < N; j += 8)
    		{
    			if (i == j)
    			{
    				x=i;
    				x1=A[x][j];x2=A[x][j+1];x3=A[x][j+2];x4=A[x][j+3];
    				x5=A[x][j+4];x6=A[x][j+5];x7=A[x][j+6];x8=A[x][j+7];
     
    				B[x][j]=x1;B[x][j+1]=x2;B[x][j+2]=x3;B[x][j+3]=x4;
    				B[x][j+4]=x5;B[x][j+5]=x6;B[x][j+6]=x7;B[x][j+7]=x8;
     
    				x1=A[x+1][j];x2=A[x+1][j+1];x3=A[x+1][j+2];x4=A[x+1][j+3];
    				x5=A[x+1][j+4];x6=A[x+1][j+5];x7=A[x+1][j+6];x8=A[x+1][j+7];
     
    				B[x+1][j]=B[x][j+1];B[x][j+1]=x1;
     
    				B[x+1][j+1]=x2;B[x+1][j+2]=x3;B[x+1][j+3]=x4;
    				B[x+1][j+4]=x5;B[x+1][j+5]=x6;B[x+1][j+6]=x7;B[x+1][j+7]=x8;
     
    				x1=A[x+2][j];x2=A[x+2][j+1];x3=A[x+2][j+2];x4=A[x+2][j+3];
    				x5=A[x+2][j+4];x6=A[x+2][j+5];x7=A[x+2][j+6];x8=A[x+2][j+7];
     
    				B[x+2][j]=B[x][j+2];B[x+2][j+1]=B[x+1][j+2];
    				B[x][j+2]=x1;B[x+1][j+2]=x2;B[x+2][j+2]=x3;
    				B[x+2][j+3]=x4;B[x+2][j+4]=x5;B[x+2][j+5]=x6;B[x+2][j+6]=x7;B[x+2][j+7]=x8;
     
    				x1=A[x+3][j];x2=A[x+3][j+1];x3=A[x+3][j+2];x4=A[x+3][j+3];
    				x5=A[x+3][j+4];x6=A[x+3][j+5];x7=A[x+3][j+6];x8=A[x+3][j+7];
     
    				B[x+3][j]=B[x][j+3];B[x+3][j+1]=B[x+1][j+3];B[x+3][j+2]=B[x+2][j+3];
    				B[x][j+3]=x1;B[x+1][j+3]=x2;B[x+2][j+3]=x3;B[x+3][j+3]=x4;
    				B[x+3][j+4]=x5;B[x+3][j+5]=x6;B[x+3][j+6]=x7;B[x+3][j+7]=x8;
     
    				x1=A[x+4][j];x2=A[x+4][j+1];x3=A[x+4][j+2];x4=A[x+4][j+3];
    				x5=A[x+4][j+4];x6=A[x+4][j+5];x7=A[x+4][j+6];x8=A[x+4][j+7];
     
    				B[x+4][j]=B[x][j+4];B[x+4][j+1]=B[x+1][j+4];B[x+4][j+2]=B[x+2][j+4];B[x+4][j+3]=B[x+3][j+4];
    				B[x][j+4]=x1;B[x+1][j+4]=x2;B[x+2][j+4]=x3;B[x+3][j+4]=x4;B[x+4][j+4]=x5;
    				B[x+4][j+5]=x6;B[x+4][j+6]=x7;B[x+4][j+7]=x8;
     
    				x1=A[x+5][j];x2=A[x+5][j+1];x3=A[x+5][j+2];x4=A[x+5][j+3];
    				x5=A[x+5][j+4];x6=A[x+5][j+5];x7=A[x+5][j+6];x8=A[x+5][j+7];
     
    				B[x+5][j]=B[x][j+5];B[x+5][j+1]=B[x+1][j+5];B[x+5][j+2]=B[x+2][j+5];B[x+5][j+3]=B[x+3][j+5];B[x+5][j+4]=B[x+4][j+5];
    				B[x][j+5]=x1;B[x+1][j+5]=x2;B[x+2][j+5]=x3;B[x+3][j+5]=x4;B[x+4][j+5]=x5;B[x+5][j+5]=x6;
    				B[x+5][j+6]=x7;B[x+5][j+7]=x8;
     
    				x1=A[x+6][j];x2=A[x+6][j+1];x3=A[x+6][j+2];x4=A[x+6][j+3];
    				x5=A[x+6][j+4];x6=A[x+6][j+5];x7=A[x+6][j+6];x8=A[x+6][j+7];
     
    				B[x+6][j]=B[x][j+6];B[x+6][j+1]=B[x+1][j+6];B[x+6][j+2]=B[x+2][j+6];B[x+6][j+3]=B[x+3][j+6];
    				B[x+6][j+4]=B[x+4][j+6];B[x+6][j+5]=B[x+5][j+6];
    				B[x][j+6]=x1;B[x+1][j+6]=x2;B[x+2][j+6]=x3;B[x+3][j+6]=x4;B[x+4][j+6]=x5;B[x+5][j+6]=x6;
    				B[x+6][j+6]=x7;B[x+6][j+7]=x8;
     
    				x1=A[x+7][j];x2=A[x+7][j+1];x3=A[x+7][j+2];x4=A[x+7][j+3];
    				x5=A[x+7][j+4];x6=A[x+7][j+5];x7=A[x+7][j+6];x8=A[x+7][j+7];
     
    				B[x+7][j]=B[x][j+7];B[x+7][j+1]=B[x+1][j+7];B[x+7][j+2]=B[x+2][j+7];B[x+7][j+3]=B[x+3][j+7];
    				B[x+7][j+4]=B[x+4][j+7];B[x+7][j+5]=B[x+5][j+7];B[x+7][j+6]=B[x+6][j+7];
    				B[x][j+7]=x1;B[x+1][j+7]=x2;B[x+2][j+7]=x3;B[x+3][j+7]=x4;B[x+4][j+7]=x5;B[x+5][j+7]=x6;B[x+6][j+7]=x7;
    				B[x+7][j+7]=x8;
    			}
    				
    			else
    			{
    				for(x = i; x < (i + 8); ++x)
    					for(y = j; y < (j + 8); ++y)
    						B[y][x] = A[x][y];
    			}
    		}
    }
    ```
    
    and the miss is now 259.
    

## 64x64

1. Step 1 using blocking
   
    the cache is like this:
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%201.png)
    
    So using 8x8 block size doesn’t work; we can only try using 4x4 block sizes.
    
    The result is 1892 < 2000 but we need <1300 to get the full score.
    
2. Step 2 using A 4*8 and B 8*4
   
    ![Screenshot 2024-11-26 at 21.14.09.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-26_at_21.14.09.png)
    
    If we consider it from an 8x8 dimension perspective, the transpose of each non-diagonal matrix will result in 24 miss cases, not to mention the diagonal matrices. Calculating this way, the total misses will exceed 24 \times 64 = 1536, which is greater than 1300. Therefore, we need to optimize the non-diagonal cases as much as possible.
    
3. Step 3:
   
    reference: https://blog.csdn.net/xbb224007/article/details/81103995
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%202.png)
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%203.png)
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%204.png)
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%205.png)
    
    ![image.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/image%206.png)
    
    miss Analysis
    
    ![Screenshot 2024-11-27 at 13.10.48.png](CacheLab%201491ab65f22f805ca9d0fe7320f90717/Screenshot_2024-11-27_at_13.10.48.png)
    
    so the miss will only be 16 in every non-diagonal 8*8 block,
    
    For diagonal matrices, we do not perform optimization.
    
4. Step 4
   
    write the code:
    
    ```c
    else if (M == 64)
    {
        int k;
        for (i = 0; i < 64; i += 8)
        {
            for (j = 0; j < 64; j += 8)
            {
                for (k = i; k < i + 4; ++k)
                {
                    tmp0 = A[k][j + 0];
                    tmp1 = A[k][j + 1];
                    tmp2 = A[k][j + 2];
                    tmp3 = A[k][j + 3];
                    tmp4 = A[k][j + 4];
                    tmp5 = A[k][j + 5];
                    tmp6 = A[k][j + 6];
                    tmp7 = A[k][j + 7];
                    B[j + 0][k] = tmp0;
                    B[j + 1][k] = tmp1;
                    B[j + 2][k] = tmp2;
                    B[j + 3][k] = tmp3;
                    B[j + 0][k + 4] = tmp4;
                    B[j + 1][k + 4] = tmp5;
                    B[j + 2][k + 4] = tmp6;
                    B[j + 3][k + 4] = tmp7;
                }
    
                for (k = j; k < j + 4; ++k)
                {
                    tmp0 = A[i + 4][k];
                    tmp1 = A[i + 5][k];
                    tmp2 = A[i + 6][k];
                    tmp3 = A[i + 7][k];
                    tmp4 = B[k][i + 4];
                    tmp5 = B[k][i + 5];
                    tmp6 = B[k][i + 6];
                    tmp7 = B[k][i + 7];
                    B[k][i + 4] = tmp0;
                    B[k][i + 5] = tmp1;
                    B[k][i + 6] = tmp2;
                    B[k][i + 7] = tmp3;
                    B[k + 4][i] = tmp4;
                    B[k + 4][i + 1] = tmp5;
                    B[k + 4][i + 2] = tmp6;
                    B[k + 4][i + 3] = tmp7;
                }
                for (k = i + 4; k < i + 8; k += 2)
                {
                    tmp0 = A[k][j + 4];
                    tmp1 = A[k][j + 5];
                    tmp2 = A[k][j + 6];
                    tmp3 = A[k][j + 7];
                    tmp4 = A[k + 1][j + 4];
                    tmp5 = A[k + 1][j + 5];
                    tmp6 = A[k + 1][j + 6];
                    tmp7 = A[k + 1][j + 7];
                    B[j + 4][k] = tmp0;
                    B[j + 5][k] = tmp1;
                    B[j + 6][k] = tmp2;
                    B[j + 7][k] = tmp3;
                    B[j + 4][k + 1] = tmp4;
                    B[j + 5][k + 1] = tmp5;
                    B[j + 6][k + 1] = tmp6;
                    B[j + 7][k + 1] = tmp7;
                }
            }
        }
    }
    ```
    
    the result is:
    
    ```
    Function 0 (2 total)
    Step 1: Validating and generating memory traces
    Step 2: Evaluating performance (s=5, E=1, b=5)
    func 0 (Transpose submission): hits:9081, misses:1164, evictions:1132
    
    Function 1 (2 total)
    Step 1: Validating and generating memory traces
    Step 2: Evaluating performance (s=5, E=1, b=5)
    func 1 (Simple row-wise scan transpose): hits:3473, misses:4724, evictions:4692
    
    Summary for official submission (func 0): correctness=1 misses=1164
    
    TEST_TRANS_RESULTS=1:1164
    ```
    

## 61x67

This part only needs miss to be less than 2000, so it is relatively simple. Keep trying, and finally, it was found that using 16*16 blocks can make miss less than 2000. The code and results are as follows:

```c
else if (M == 61)
{
    int blockSize = 16;
    int i, j, ii, jj;
    for (i = 0; i < N; i += blockSize)
    {
        for (j = 0; j < M; j += blockSize)
        {
            for (ii = i; ii < i + blockSize && ii < N; ii++)
            {
                for (jj = j; jj < j + blockSize && jj < M; jj++)
                {
                    B[jj][ii] = A[ii][jj];
                }
            }
        }
    }
}
```

```
Function 0 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:6186, misses:1993, evictions:1961

Function 1 (2 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 1 (Simple row-wise scan transpose): hits:3755, misses:4424, evictions:4392

Summary for official submission (func 0): correctness=1 misses=1993

TEST_TRANS_RESULTS=1:1993
```

---