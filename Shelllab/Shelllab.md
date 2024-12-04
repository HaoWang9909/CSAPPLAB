# Shelllab

## `Learning Materials`

[shlab.pdf](Shelllab%201511ab65f22f80848aecca8014a50234/shlab.pdf)

# Key points in the pdf

## **Instructions**

- copying the file shlab-handout.tar to the protected directory

```bash
tar xvf shlab-handout.tar
cd shlab-handout
make
```

- complete the remaining empty functions listed below
    - `eval:` Main routine that parses and interprets the command line. [70 lines]
    - `builtin_cmd`: Recognizes and interprets the built-in commands: `quit`, `fg`, `bg`, and `jobs`. [25
    
    lines]
    
    - `do_bgfg`: Implements the `bg` and `fg` built-in commands. [50 lines]
    - `waitfg`: Waits for a foreground job to complete. [20 lines]
    - `sigchld_handler`: Catches `SIGCHILD` signals. 80 lines]
    - `sigint_handler`: Catches `SIGINT` (ctrl-c) signals. [15 lines]
    - `sigtstp_handler`: Catches `SIGTSTP` (ctrl-z) signals. [15 lines]
    - Each time you modify your `tsh.c` file, type make to recompile it. To run your shell, type tsh to the command line:
      
        ```bash
        unix> ./tsh
        tsh> *[type commands to your shell here]*
        ```
        

## **General Overview of Unix Shells**

- **Command Handling**:
    - **Built-in commands** are executed directly by the shell.
    - Other commands invoke executable files by forking a child process and running the program.
- **Foreground and Background Jobs**:
    - Commands ending with & are executed in the background.
    - Foreground jobs block the shell until completion.
- **Job Control**:
    - Typing ctrl-c causes a SIGINT signal to be delivered to each process in the foreground job.
    - typing ctrl-z causes a SIGTSTP signal to be delivered to each process in the foreground job.
    - jobs: List the running and stopped background jobs.
    - bg <job>: Change a stopped background job to a running background job.
    - fg <job>: Change a stopped or running background job to a running in the foreground.
    - kill <job>: Terminate a job.

## **The** tsh **Specification !!!!**

The following outlines the required features and behavior of the tsh (tiny shell) implementation:

1. **Prompt**
    - The shell prompt must display:
      
        ```bash
        tsh>
        ```
    
2. **Command Line Input**
    - The command line consists of:
        - A **command name**.
        - Zero or more **arguments**, separated by one or more spaces.
    - **Behavior:**
        - If the command is a **built-in command**, tsh handles it immediately and waits for the next command line.
        - Otherwise, tsh assumes the command name is the **path of an executable file**, which it:
            1. Loads and runs in the context of an initial child process.
            2. Refers to this child process as a **job**.
3. **Unsupported Features**
    - tsh does **not** support:
        - Pipes (`|`).
        - I/O redirection (`<` and `>`).
4. **Signal Handling**
    - **Ctrl-C (SIGINT)**:
        - Sends the signal to the **current foreground job** and all its descendant processes.
        - Has no effect if no foreground job exists.
    - **Ctrl-Z (SIGTSTP)**:
        - Sends the signal to the **current foreground job** and all its descendant processes.
        - Stops the job until it is resumed with a `SIGCONT`.
5. **Background and Foreground Jobs**
    - **Background Job**:
        - If the command line ends with `&`, tsh runs the job in the background.
    - **Foreground Job**:
        - If the command line does not end with `&`, tsh runs the job in the foreground.
6. **Job Identification**
    - Each job is identified by:
        - A **Process ID (PID)**.
        - A **Job ID (JID)**:
            - Positive integers assigned by tsh.
            - Denoted by `%` on the command line:
            - Example: `%5` refers to JID 5.
            - Example: `5` refers to PID 5.
7. **Built-in Commands**
    - tsh must support the following built-in commands:
        1. `quit`: Terminates the shell.
        2. `jobs`: Lists all background jobs.
        3. `bg <job>`:
            - Restarts `<job>` by sending it a `SIGCONT` signal.
            - Runs `<job>` in the **background**.
            - `<job>` can be identified by either PID or JID.
        4. `fg <job>`:
            - Restarts `<job>` by sending it a `SIGCONT` signal.
            - Runs `<job>` in the **foreground**.
            - `<job>` can be identified by either PID or JID.
8. **Zombie Process Reaping**
    - tsh must reap all **zombie child processes**:
        - If a job terminates due to receiving an uncaught signal, tsh should:
            - Recognize the event.
            - Print a message with the job’s PID and a description of the signal.

## **Checking**

1. **Reference Solution**
    - The reference program `tshref` is the official implementation of the shell.
    - Run this program to understand how your shell should behave.
    - Your shell should produce output identical to the reference solution (except for PIDs, which vary with each run).
2. **Shell Test Driver**
    - `sdriver.pl` is a test driver program that:
        - Runs the shell as a child process.
        - Sends commands and signals to it based on a trace file.
        - Captures and displays the shell's output.
    - Use the `h` flag to view the usage of `sdriver.pl`:
      
        ```bash
        unix> ./sdriver.pl -h
        Usage: sdriver.pl [-hv] -t <trace> -s <shellprog> -a <args>
        Options:
          -h            Print this message
          -v            Be more verbose
          -t <trace>    Trace file
          -s <shell>    Shell program to test
          -a <args>     Shell arguments
          -g            Generate output for autograder
        ```
    
3. **Provided Trace Files**
    - We provide 16 trace files `(trace{01-16}.txt)` for testing your shell with the test driver:
    - **Lower-numbered trace files** perform simple tests.
    - **Higher-numbered trace files** test more complex scenarios.
4. **Testing Examples**
    - Test your shell with `trace01.txt`: Run `sdriver.pl` with the trace file and your shell program, and use `a "-p"` to suppress the prompt.
      
        ```bash
        unix> ./sdriver.pl -t trace01.txt -s ./tsh -a "-p"
        ```
        
    - Alternatively, use `make test01` to simplify running the test.
    - Compare your shell's result with the reference shell by using the same trace file and the `tshref` program, or use `make rtest01` for convenience.
      
        ```bash
        unix> ./sdriver.pl -t trace01.txt -s ./tshref -a "-p"
        ```
    
5. **Reference Output**
    - The file `tshref.out` contains the reference solution's output for all trace files.
    - You can use it to verify correctness without manually running all tests.
6. **Features of Trace Files**
    - Trace files produce the same output you would get in an interactive session, except for an initial comment that identifies the trace.
7. **Example Output**
    - Here is an example from running the `trace15.txt` test:
      
        ```bash
        bass> make test15
        ./sdriver.pl -t trace15.txt -s ./tsh -a "-p"
        #
        # trace15.txt - Putting it all together
        #
        tsh> ./bogus
        ./bogus: Command not found.
        tsh> ./myspin 10
        Job (9721) terminated by signal 2
        tsh> ./myspin 3 &
        [1] (9723) ./myspin 3 &
        tsh> ./myspin 4 &
        [2] (9725) ./myspin 4 &
        tsh> jobs
        [1] (9723) Running ./myspin 3 &
        [2] (9725) Running ./myspin 4 &
        tsh> fg %1
        Job [1] (9723) stopped by signal 20
        tsh> jobs
        [1] (9723) Stopped ./myspin 3 &
        [2] (9725) Running ./myspin 4 &
        tsh> bg %3
        %3: No such job
        tsh> bg %1
        [1] (9723) ./myspin 3 &
        tsh> jobs
        [1] (9723) Running ./myspin 3 &
        [2] (9725) Running ./myspin 4 &
        tsh> fg %1
        tsh> quit
        bass>
        ```
        
    - The shell responds with expected outputs, showing interactions with commands like `jobs`, `fg`, `bg`, and handling stopped or running jobs correctly.
    - It also handles invalid job IDs gracefully, printing relevant error messages.
    - The shell terminates successfully when `quit` is executed.

## Hint

- **Reading Requirement：**
Carefully read every word of Chapter 8, *Exceptional Control Flow*, in your textbook.
- **Development Steps**
  Use the trace files to guide the development of your shell.
    - Start with `trace01.txt`, ensuring that your shell produces output identical to the reference shell.
    - Then proceed to `trace02.txt` and so on.
- **Key Functions**
  The following functions will be very useful during implementation:
    - `waitpid`
    - `kill`
    - `fork`
    - `execve`
    - `setpgid`
    - `sigprocmask`
      Additionally, the `WUNTRACED` and `WNOHANG` options for `waitpid` will be helpful.
- **Signal Handling Notes**
    - When implementing signal handlers, ensure that `SIGINT` and `SIGTSTP` signals are sent to the entire foreground process group.
    - Use `-pid` instead of `pid` as the argument to the `kill` function to send the signal to the whole process group.
    - The `sdriver.pl` program will test for this error.
- **Work Allocation: `waitfg` and `sigchld` Handler**
  One tricky part is deciding how to allocate work between the `waitfg` and `sigchld` handler functions.
  Recommended approach:
    - Use a busy loop with the `sleep` function in `waitfg`.
    - Use exactly one call to `waitpid` in the `sigchld` handler.
      While other solutions, such as calling `waitpid` in both `waitfg` and the handler, are possible, they can become confusing. It is simpler to handle all reaping in the handler.
- **Blocking Signals: Notes for `eval`**
    - In `eval`, the parent must block `SIGCHLD` signals using `sigprocmask`:
        1. Before forking the child process.
        2. After adding the child to the job list by calling `addjob`, unblock the signals.
    - Since child processes inherit the blocked signal vector of their parent, the child must unblock `SIGCHLD` signals before executing the new program (`exec`).
    - Blocking `SIGCHLD` signals prevents a race condition where the child is reaped by the `sigchld` handler and removed from the job list before the parent calls `addjob`.
- **Run Simple Programs**
    - Avoid running programs like `more`, `less`, `vi`, or `emacs`, as they modify terminal settings.
    - Stick to simple text-based programs like `/bin/ls`, `/bin/ps`, and `/bin/echo`.
- **Foreground Process Group Issues and Solution**
    - When your shell runs from the standard Unix shell, it is part of the foreground process group.
    - If your shell creates a child process, by default, that child will also be a member of the foreground process group.
    - Pressing `Ctrl-C` sends a `SIGINT` to every process in the foreground group, including your shell and its child processes, which is incorrect.
    
    **Solution:**
    
    - After `fork` but before `execve`, the child process should call `setpgid(0, 0)`.
    - This moves the child to a new process group with a group ID identical to its PID.
    - This ensures that only your shell remains in the foreground process group.
    - When you press `Ctrl-C`, the shell should catch the resulting `SIGINT` signal and forward it to the appropriate foreground job (or, more precisely, the process group containing the foreground job).

# Solution

## Step 0 (copy!!)

Write some error handling wrapper functions similar to `csapp.c` http://csapp.cs.cmu.edu/3e/ics3/code/src/csapp.c

```c
/* Wrappers for Unix process control functions */
pid_t Fork(void);
void Execve(const char *filename, char *const argv[], char *const envp[]);
pid_t Wait(int *status);
pid_t Waitpid(pid_t pid, int *iptr, int options);
void Kill(pid_t pid, int signum);
void Pause();
unsigned int Sleep(unsigned int secs);
unsigned int Alarm(unsigned int seconds);
void Setpgid(pid_t pid, pid_t pgid);
pid_t Getpgrp(void);
void Sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
void Sigemptyset(sigset_t *set);
void Sigfillset(sigset_t *set);
void Sigaddset(sigset_t *set, int signum);
void Sigdelset(sigset_t *set, int signum);
int Sigismember(const sigset_t *set, int signum);
int Sigsuspend(const sigset_t *set);

....
....
....

/*********************************************
 * Wrappers for Unix process control functions
 ********************************************/

/* $begin forkwrapper */
pid_t Fork(void) 
{
    pid_t pid;

    if ((pid = fork()) < 0)
	unix_error("Fork error");
    return pid;
}
/* $end forkwrapper */

void Execve(const char *filename, char *const argv[], char *const envp[]) 
{
    if (execve(filename, argv, envp) < 0)
	unix_error("Execve error");
}

/* $begin wait */
pid_t Wait(int *status) 
{
    pid_t pid;

    if ((pid  = wait(status)) < 0)
	unix_error("Wait error");
    return pid;
}
/* $end wait */

pid_t Waitpid(pid_t pid, int *iptr, int options) 
{
    pid_t retpid;

    if ((retpid  = waitpid(pid, iptr, options)) < 0) 
	unix_error("Waitpid error");
    return(retpid);
}

/* $begin kill */
void Kill(pid_t pid, int signum) 
{
    int rc;

    if ((rc = kill(pid, signum)) < 0)
	unix_error("Kill error");
}
/* $end kill */

void Pause() 
{
    (void)pause();
    return;
}

unsigned int Sleep(unsigned int secs) 
{
    unsigned int rc;

    if ((rc = sleep(secs)) < 0)
	unix_error("Sleep error");
    return rc;
}

unsigned int Alarm(unsigned int seconds) {
    return alarm(seconds);
}
 
void Setpgid(pid_t pid, pid_t pgid) {
    int rc;

    if ((rc = setpgid(pid, pgid)) < 0)
	unix_error("Setpgid error");
    return;
}

pid_t Getpgrp(void) {
    return getpgrp();
}

/************************************
 * Wrappers for Unix signal functions 
 ***********************************/

/* $begin sigaction */
handler_t *Signal(int signum, handler_t *handler) 
{
    struct sigaction action, old_action;

    action.sa_handler = handler;  
    sigemptyset(&action.sa_mask); /* Block sigs of type being handled */
    action.sa_flags = SA_RESTART; /* Restart syscalls if possible */

    if (sigaction(signum, &action, &old_action) < 0)
	unix_error("Signal error");
    return (old_action.sa_handler);
}
/* $end sigaction */

void Sigprocmask(int how, const sigset_t *set, sigset_t *oldset)
{
    if (sigprocmask(how, set, oldset) < 0)
	unix_error("Sigprocmask error");
    return;
}

void Sigemptyset(sigset_t *set)
{
    if (sigemptyset(set) < 0)
	unix_error("Sigemptyset error");
    return;
}

void Sigfillset(sigset_t *set)
{ 
    if (sigfillset(set) < 0)
	unix_error("Sigfillset error");
    return;
}

void Sigaddset(sigset_t *set, int signum)
{
    if (sigaddset(set, signum) < 0)
	unix_error("Sigaddset error");
    return;
}

void Sigdelset(sigset_t *set, int signum)
{
    if (sigdelset(set, signum) < 0)
	unix_error("Sigdelset error");
    return;
}

int Sigismember(const sigset_t *set, int signum)
{
    int rc;
    if ((rc = sigismember(set, signum)) < 0)
	unix_error("Sigismember error");
    return rc;
}

int Sigsuspend(const sigset_t *set)
{
    int rc = sigsuspend(set); /* always returns -1 */
    if (errno != EINTR)
        unix_error("Sigsuspend error");
    return rc;
}
```

**VS Code Error: Undefined identifier "sigset_t"**

Press **`cmd+shift+p`** to bring up the command palette, search for **`C/C++ Edit Configuration (json)`**, and then change **`cStandard`** to **`gnu11`**.

```c
{
  "configurations": [
    {
      "name": "Linux",
      "includePath": ["${workspaceFolder}/**"],
      "defines": [],
      "cStandard": "gnu11",
      "cppStandard": "gnu++17",
      "intelliSenseMode": "linux-gcc-x64"
    }
  ],
  "version": 4
}
	
```

## **Trace01**

```
#
# trace01.txt - Properly terminate on EOF.
#
CLOSE
WAIT

```

This function has already been completed in `tsh.c`(line 145-149)

```c
        if (feof(stdin))
        { /* End of file (ctrl-d) */
            fflush(stdout);
            exit(0);
        }
```

## **Trace02**

```
#
# trace02.txt - Process builtin quit command.
#
quit
WAIT

```

`eval()` can be referred in picture 8-24 in the textbook.

```c
void eval(char *cmdline)
{
    char *argv[MAXARGS]; /* Argument list execve() */
    char buf[MAXLINE];   /* Holds modified command line */
    int bg;              /* Should the job run in bg or fg? */
    pid_t pid;           /* Process id */

    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if (argv[0] == NULL)
        return; /* Ignore empty lines */

    if (!builtin_cmd(argv))
    {
        if ((pid = Fork()) == 0)
        { /* Child runs user job */
            if (execve(argv[0], argv, environ) < 0)
            {
                printf("%s: Command not found.\n", argv[0]);
                exit(0);
            }
        }

        /* Parent waits for foreground job to terminate */
        if (!bg)
        {
            int status;
            if (waitpid(pid, &status, 0) < 0)
                unix_error("waitfg: waitpid error");
        }
        else
            printf("%d %s", pid, cmdline);
    }
    return;
}
```

```c
int builtin_cmd(char **argv)
{
    if (!strcmp(argv[0], "quit"))
    {
        exit(0);
    }
    return 0; /* not a builtin command */
}
```

Then we use `make test02` to see that shel automatically exits.

## **Trace03**

```
#
# trace03.txt - Run a foreground job.
#
/bin/echo tsh> quit
quit

```

The result is correct.

## **Trace04**

```
#
# trace04.txt - Run a background job.
#
/bin/echo -e tsh> ./myspin 1 \046
./myspin 1 &

```

If we do nothing, the result is now:

```
./sdriver.pl -t trace04.txt -s ./tsh -a "-p"
#
# trace04.txt - Run a background job.
#
tsh> ./myspin 1 &
11350 ./myspin 1 &
```

but it is different to the reference:`./sdriver.pl -t trace04.txt -s ./tshref -a "-p”` 

```
./sdriver.pl -t trace04.txt -s ./tshref -a "-p"
#
# trace04.txt - Run a background job.
#
tsh> ./myspin 1 &
[1] (11487) ./myspin 1 &
```

So we need to modify the eval function, add the background-running program to the jobs, and while we're at it, also add the foreground-running program to the jobs. Referring to the requirements in the hint, we still need to:

> In `eval`, the parent must block `SIGCHLD` signals using `sigprocmask`:
> 
> 1. Before forking the child process.
> 2. After adding the child to the job list by calling `addjob`, unblock the signals.

> Since child processes inherit the blocked signal vector of their parent, the child must unblock `SIGCHLD` signals before executing the new program (`exec`).
> 

> Blocking `SIGCHLD` signals prevents a race condition where the child is reaped by the `sigchld` handler and removed from the job list before the parent calls `addjob`.
> 

Then we rewrite the eval function:

```c
void eval(char *cmdline)
{
    char *argv[MAXARGS]; /* Argument list execve() */
    char buf[MAXLINE];   /* Holds modified command line */
    int bg;              /* Should the job run in bg or fg? */
    pid_t pid;           /* Process id */
    sigset_t mask_all, prev_all;
    sigset_t mask_one, prev_one;
    Sigfillset(&mask_all);
    Sigemptyset(&mask_one);
    Sigaddset(&mask_one, SIGCHLD);

    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if (argv[0] == NULL)
        return; /* Ignore empty lines */

    if (!builtin_cmd(argv))
    {
        Sigprocmask(SIG_BLOCK, &mask_one, &prev_one);
        if ((pid = Fork()) == 0)
        { /* Child runs user job */
            Sigprocmask(SIG_SETMASK, &prev_one, NULL);
            Setpgid(0, 0);
            Execve(argv[0], argv, environ);
        }

        /* Parent waits for foreground job to terminate */
        if (!bg)
        {
            Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
            addjob(jobs, pid, FG, cmdline);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
            int status;
            Waitpid(pid, &status, 0);
        }
        else
        {
            Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
            addjob(jobs, pid, BG, cmdline);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
            printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
        }
        Sigprocmask(SIG_SETMASK, &prev_one, NULL);
    }
    return;
}
```

now the result is:

```
➜  shlab-handout make test04
./sdriver.pl -t trace04.txt -s ./tsh -a "-p"
#
# trace04.txt - Run a background job.
#
tsh> ./myspin 1 &
[2] (13114) ./myspin 1 &
```

still different, because we did not delete the job that has already finished, the foreground one `./myspin 1 \046` 

so we need to write the `waitfg()` function and replace the `Waitpid` function in `eval()` with it. We can refer to the figure 8-24 in the text book.

```c
void waitfg(pid_t pid)
{
    sigset_t mask, prev;
    sigset_t mask_none;

    Sigemptyset(&mask);
    Sigemptyset(&mask_none);
    Sigaddset(&mask, SIGCHLD);

    Sigprocmask(SIG_BLOCK, &mask, &prev);

    while (pid == fgpid(jobs))
        Sigsuspend(&mask_none);

    Sigprocmask(SIG_SETMASK, &prev, NULL);

    return;
}
```

And now we need to write the `sigchld_handler` 

```c
/*
 * sigchld_handler - The kernel sends a SIGCHLD to the shell whenever
 *     a child job terminates (becomes a zombie), or stops because it
 *     received a SIGSTOP or SIGTSTP signal. The handler reaps all
 *     available zombie children, but doesn't wait for any other
 *     currently running children to terminate.
 */
```

so we need to set the options in the `Waitpid()` to `WNOHANG | WUNTRACED`

```c
void sigchld_handler(int sig) 
{
    int olderron = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    int status;
    Sigfillset(&mask_all);
    while((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0)//note do not use Waitpid()
    {
        if(WIFEXITED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            deletejob(jobs, pid);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }
    }

    errno = olderron;
    return ;
}
```

Now the result is correct!

```
➜  shlab-handout make test04
./sdriver.pl -t trace04.txt -s ./tsh -a "-p"
#
# trace04.txt - Run a background job.
#
tsh> ./myspin 1 &
[1] (15003) ./myspin 1 &
```

## **Trace05**

```
#
# trace05.txt - Process jobs builtin command.
#
/bin/echo -e tsh> ./myspin 2 \046
./myspin 2 &

/bin/echo -e tsh> ./myspin 3 \046
./myspin 3 &

/bin/echo tsh> jobs
jobs

```

the result should be

```
➜  shlab-handout ./sdriver.pl -t trace05.txt -s ./tshref -a "-p"
#
# trace05.txt - Process jobs builtin command.
#
tsh> ./myspin 2 &
[1] (15115) ./myspin 2 &
tsh> ./myspin 3 &
[2] (15117) ./myspin 3 &
tsh> jobs
[1] (15115) Running ./myspin 2 &
[2] (15117) Running ./myspin 3 &
```

complete the `builtin_cmd` 

```c
int builtin_cmd(char **argv)
{
    if (!strcmp(argv[0], "quit"))
    {
        exit(0);
    }
    else if (!strcmp(argv[0], "jobs"))
    {
        listjobs(jobs);
        return 1;
    }
    else if (!strcmp(argv[0], "&"))
        return 1;
    return 0; /* not a builtin command */
}

```

the result is now correct!

```
➜  shlab-handout make test05
./sdriver.pl -t trace05.txt -s ./tsh -a "-p"
#
# trace05.txt - Process jobs builtin command.
#
tsh> ./myspin 2 &
[1] (15286) ./myspin 2 &
tsh> ./myspin 3 &
[2] (15288) ./myspin 3 &
tsh> jobs
[1] (15286) Running ./myspin 2 &
[2] (15288) Running ./myspin 3 &
```

## **Trace06**

```
#
# trace06.txt - Forward SIGINT to foreground job.
#
/bin/echo -e tsh> ./myspin 4
./myspin 4 

SLEEP 2
INT

```

the result is:

```
➜  shlab-handout ./sdriver.pl -t trace06.txt -s ./tshref -a "-p"
#
# trace06.txt - Forward SIGINT to foreground job.
#
tsh> ./myspin 4
Job [1] (15389) terminated by signal 2
```

We need to complete the `sigint_handler` to terminate the process and `sigchld_handler` to show the information.

```c
void sigint_handler(int sig)
{
    int olderron = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    Sigfillset(&mask_all);

    Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
    pid = fgpid(jobs);
    Sigprocmask(SIG_SETMASK, &prev_all, NULL);
    if (pid != 0)
        Kill(-pid, sig);

    errno = olderron;
    return;
}
```

```c
void sigchld_handler(int sig)
{
    int olderron = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    int status;
    Sigfillset(&mask_all);
    while ((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0)
    {
        if (WIFEXITED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            deletejob(jobs, pid);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }
        else if (WIFSIGNALED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            char buf[256];
            int len = snprintf(buf, sizeof(buf), "Job [%d] (%d) terminated by signal %d\n",
                               pid2jid(pid), pid, WTERMSIG(status));
            if (len > 0)
            {
                ssize_t written = write(STDOUT_FILENO, buf, len);
                (void)written;
            }
            deletejob(jobs, pid);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }

        errno = olderron;
        return;
    }
}
```

then the result is correct:

```
➜  shlab-handout make test06
./sdriver.pl -t trace06.txt -s ./tsh -a "-p"
#
# trace06.txt - Forward SIGINT to foreground job.
#
tsh> ./myspin 4
Job [1] (15799) terminated by signal 2
```

## **Trace07**

```
#
# trace07.txt - Forward SIGINT only to foreground job.
#
/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

/bin/echo -e tsh> ./myspin 5
./myspin 5 

SLEEP 2
INT

/bin/echo tsh> jobs
jobs

```

already finished!

```
➜  shlab-handout make test07
./sdriver.pl -t trace07.txt -s ./tsh -a "-p"
#
# trace07.txt - Forward SIGINT only to foreground job.
#
tsh> ./myspin 4 &
[1] (15951) ./myspin 4 &
tsh> ./myspin 5
Job [2] (15953) terminated by signal 2
tsh> jobs
[1] (15951) Running ./myspin 4 &
```

## **Trace08**

```
#
# trace08.txt - Forward SIGTSTP only to foreground job.
#
/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

/bin/echo -e tsh> ./myspin 5
./myspin 5 

SLEEP 2
TSTP

/bin/echo tsh> jobs
jobs

```

```
➜  shlab-handout ./sdriver.pl -t trace08.txt -s ./tshref -a "-p"
#
# trace08.txt - Forward SIGTSTP only to foreground job.
#
tsh> ./myspin 4 &
[1] (16077) ./myspin 4 &
tsh> ./myspin 5
Job [2] (16079) stopped by signal 20
tsh> jobs
[1] (16077) Running ./myspin 4 &
[2] (16079) Stopped ./myspin 5 
```

We need to complete the `sigstp_handler` to stop the process and `sigchld_handler` to show the information.

```c
void sigtstp_handler(int sig)
{
    int olderron = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    Sigfillset(&mask_all);

    Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
    pid = fgpid(jobs);
    Sigprocmask(SIG_SETMASK, &prev_all, NULL);
    if (pid != 0)
        Kill(-pid, sig);

    errno = olderron;
    return;
}
```

```c
void sigchld_handler(int sig)
{
    int olderron = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;
    int status;
    Sigfillset(&mask_all);
    while ((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0)
    {
        if (WIFEXITED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            deletejob(jobs, pid);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }
        else if (WIFSIGNALED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            char buf[256];
            int len = snprintf(buf, sizeof(buf), "Job [%d] (%d) terminated by signal %d\n",
                               pid2jid(pid), pid, WTERMSIG(status));
            if (len > 0)
            {
                ssize_t written = write(STDOUT_FILENO, buf, len);
                (void)written;
            }
            deletejob(jobs, pid);
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }
        else if (WIFSTOPPED(status))
        {
            Sigprocmask(SIG_SETMASK, &mask_all, &prev_all);
            struct job_t *job = getjobpid(jobs, pid);
            job->state = ST;
            char buf[256];
            int len = snprintf(buf, sizeof(buf), "Job [%d] (%d) stopped by signal %d\n",
                               pid2jid(pid), pid, WSTOPSIG(status));
            if (len > 0)
            {
                ssize_t written = write(STDOUT_FILENO, buf, len);
                (void)written;
            }
            Sigprocmask(SIG_SETMASK, &prev_all, NULL);
        }

        errno = olderron;
        return;
    }
}
```

test

```c
➜  shlab-handout make test08
./sdriver.pl -t trace08.txt -s ./tsh -a "-p"
#
# trace08.txt - Forward SIGTSTP only to foreground job.
#
tsh> ./myspin 4 &
[1] (16239) ./myspin 4 &
tsh> ./myspin 5
Job [2] (16241) stopped by signal 20
tsh> jobs
[1] (16239) Running ./myspin 4 &
[2] (16241) Stopped ./myspin 5 
```

correct!

## **Trace09**

```
#
# trace09.txt - Process bg builtin command
#
/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

/bin/echo -e tsh> ./myspin 5
./myspin 5 

SLEEP 2
TSTP

/bin/echo tsh> jobs
jobs

/bin/echo tsh> bg %2
bg %2

/bin/echo tsh> jobs
jobs

```

reference:

```
➜  shlab-handout ./sdriver.pl -t trace09.txt -s ./tshref -a "-p"
#
# trace09.txt - Process bg builtin command
#
tsh> ./myspin 4 &
[1] (16327) ./myspin 4 &
tsh> ./myspin 5
Job [2] (16329) stopped by signal 20
tsh> jobs
[1] (16327) Running ./myspin 4 &
[2] (16329) Stopped ./myspin 5 
tsh> bg %2
[2] (16329) ./myspin 5 
tsh> jobs
[1] (16327) Running ./myspin 4 &
[2] (16329) Running ./myspin 5 
```

so we need to write the `do_bgfg()` and complete the `builtin_cmd()`:

```c
int builtin_cmd(char **argv)
{
    if (!strcmp(argv[0], "quit"))
    {
        exit(0);
    }
    else if (!strcmp(argv[0], "jobs"))
    {
        listjobs(jobs);
        return 1;
    }
    else if (!strcmp(argv[0], "&"))
        return 1;
    else if (!strcmp(argv[0], "bg") || !strcmp(argv[0], "fg"))
    {
        do_bgfg(argv);
        return 1;
    }
    return 0; /* not a builtin command */
}
```

```c
void do_bgfg(char **argv)
{
    struct job_t *job = NULL;
    int state;

    if (!strcmp(argv[0], "bg"))
        state = BG;
    else if (!strcmp(argv[0], "fg"))
        state = FG;
    else
        return;

    if (argv[1] == NULL)
    {
        return;
    }

    if (argv[1][0] == '%')
    {
        int jid = atoi(argv[1] + 1);
        job = getjobjid(jobs, jid);
    }
    else if (argv[1][0] == '\0')
    {
        return;
    }

    Kill(-(job->pid), SIGCONT);
    job->state = state;
    if (state == BG)
        printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
    else if (state == FG)
        waitfg(job->pid);
    return;
}
```

test:

```
➜  shlab-handout make test09
./sdriver.pl -t trace09.txt -s ./tsh -a "-p"
#
# trace09.txt - Process bg builtin command
#
tsh> ./myspin 4 &
[1] (16611) ./myspin 4 &
tsh> ./myspin 5
Job [2] (16613) stopped by signal 20
tsh> jobs
[1] (16611) Running ./myspin 4 &
[2] (16613) Stopped ./myspin 5 
tsh> bg %2
[2] (16613) ./myspin 5 
tsh> jobs
[1] (16611) Running ./myspin 4 &
[2] (16613) Running ./myspin 5 
```

correct!

## **Trace10**

```
#
# trace10.txt - Process fg builtin command. 
#
/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

SLEEP 1
/bin/echo tsh> fg %1
fg %1

SLEEP 1
TSTP

/bin/echo tsh> jobs
jobs

/bin/echo tsh> fg %1
fg %1

/bin/echo tsh> jobs
jobs

```

the result should be:

```
➜  shlab-handout ./sdriver.pl -t trace10.txt -s ./tshref -a "-p"
#
# trace10.txt - Process fg builtin command. 
#

tsh> ./myspin 4 &
[1] (16943) ./myspin 4 &
tsh> fg %1
Job [1] (16943) stopped by signal 20
tsh> jobs
[1] (16943) Stopped ./myspin 4 &
tsh> fg %1
tsh> jobs
```

test:

```
➜  shlab-handout make test10
./sdriver.pl -t trace10.txt -s ./tsh -a "-p"
#
# trace10.txt - Process fg builtin command. 
#
tsh> ./myspin 4 &
[1] (17104) ./myspin 4 &
tsh> fg %1
Job [1] (17104) stopped by signal 20
tsh> jobs
[1] (17104) Stopped ./myspin 4 &
tsh> fg %1
tsh> jobs
```

correct!

## **Trace11**

```
#
# trace11.txt - Forward SIGINT to every process in foreground process group
#
/bin/echo -e tsh> ./mysplit 4
./mysplit 4 

SLEEP 2
INT

/bin/echo tsh> /bin/ps a
/bin/ps a

```

test

```
➜  shlab-handout make test11
./sdriver.pl -t trace11.txt -s ./tsh -a "-p"
#
# trace11.txt - Forward SIGINT to every process in foreground process group
#
tsh> ./mysplit 4
Job [1] (17329) terminated by signal 2
tsh> /bin/ps a
    PID TTY      STAT   TIME COMMAND
    782 ttyS0    Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 ttyS0 vt220
    786 tty1     Ss+    0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
   8662 pts/1    Ss+    0:01 /usr/bin/zsh -i
   9811 pts/2    Ss+    0:00 /usr/bin/zsh -i
  11213 pts/3    Ss     0:06 /usr/bin/zsh -i
  13863 pts/3    S      0:00 ./tsh -p
  13864 pts/3    Z      0:00 [echo] <defunct>
  14066 pts/3    S      0:00 ./tsh -p
  14067 pts/3    Z      0:00 [echo] <defunct>
  17324 pts/3    S+     0:00 make test11
  17325 pts/3    S+     0:00 /bin/sh -c ./sdriver.pl -t trace11.txt -s ./tsh -a "-p"
  17326 pts/3    S+     0:00 /usr/bin/perl ./sdriver.pl -t trace11.txt -s ./tsh -a -p
  17327 pts/3    S+     0:00 ./tsh -p
  17350 pts/3    R      0:00 /bin/ps a
```

correct!

## **Trace12**

```
#
# trace12.txt - Forward SIGTSTP to every process in foreground process group
#
/bin/echo -e tsh> ./mysplit 4
./mysplit 4 

SLEEP 2
TSTP

/bin/echo tsh> jobs
jobs

/bin/echo tsh> /bin/ps a
/bin/ps a

```

test:

```
➜  shlab-handout make test12
./sdriver.pl -t trace12.txt -s ./tsh -a "-p"
#
# trace12.txt - Forward SIGTSTP to every process in foreground process group
#
tsh> ./mysplit 4
Job [1] (17403) stopped by signal 20
tsh> jobs
[1] (17403) Stopped ./mysplit 4 
tsh> /bin/ps a
    PID TTY      STAT   TIME COMMAND
    782 ttyS0    Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 ttyS0 vt220
    786 tty1     Ss+    0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
   8662 pts/1    Ss+    0:01 /usr/bin/zsh -i
   9811 pts/2    Ss+    0:00 /usr/bin/zsh -i
  11213 pts/3    Ss     0:06 /usr/bin/zsh -i
  13863 pts/3    S      0:00 ./tsh -p
  13864 pts/3    Z      0:00 [echo] <defunct>
  14066 pts/3    S      0:00 ./tsh -p
  14067 pts/3    Z      0:00 [echo] <defunct>
  17398 pts/3    S+     0:00 make test12
  17399 pts/3    S+     0:00 /bin/sh -c ./sdriver.pl -t trace12.txt -s ./tsh -a "-p"
  17400 pts/3    S+     0:00 /usr/bin/perl ./sdriver.pl -t trace12.txt -s ./tsh -a -p
  17401 pts/3    S+     0:00 ./tsh -p
  17403 pts/3    T      0:00 ./mysplit 4
  17404 pts/3    T      0:00 ./mysplit 4
  17428 pts/3    R      0:00 /bin/ps a
```

correct!

## **Trace13**

```
#
# trace13.txt - Restart every stopped process in process group
#
/bin/echo -e tsh> ./mysplit 4
./mysplit 4 

SLEEP 2
TSTP

/bin/echo tsh> jobs
jobs

/bin/echo tsh> /bin/ps a
/bin/ps a

/bin/echo tsh> fg %1
fg %1

/bin/echo tsh> /bin/ps a
/bin/ps a

```

test

```
➜  shlab-handout make test13
./sdriver.pl -t trace13.txt -s ./tsh -a "-p"
#
# trace13.txt - Restart every stopped process in process group
#
tsh> ./mysplit 4
Job [1] (17701) stopped by signal 20
tsh> jobs
[1] (17701) Stopped ./mysplit 4 
tsh> /bin/ps a
    PID TTY      STAT   TIME COMMAND
    782 ttyS0    Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 ttyS0 vt220
    786 tty1     Ss+    0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
   8662 pts/1    Ss+    0:01 /usr/bin/zsh -i
   9811 pts/2    Ss+    0:00 /usr/bin/zsh -i
  11213 pts/3    Ss     0:06 /usr/bin/zsh -i
  13863 pts/3    S      0:00 ./tsh -p
  13864 pts/3    Z      0:00 [echo] <defunct>
  14066 pts/3    S      0:00 ./tsh -p
  14067 pts/3    Z      0:00 [echo] <defunct>
  17696 pts/3    S+     0:00 make test13
  17697 pts/3    S+     0:00 /bin/sh -c ./sdriver.pl -t trace13.txt -s ./tsh -a "-p"
  17698 pts/3    S+     0:00 /usr/bin/perl ./sdriver.pl -t trace13.txt -s ./tsh -a -p
  17699 pts/3    S+     0:00 ./tsh -p
  17701 pts/3    T      0:00 ./mysplit 4
  17702 pts/3    T      0:00 ./mysplit 4
  17726 pts/3    R      0:00 /bin/ps a
tsh> fg %1
tsh> /bin/ps a
    PID TTY      STAT   TIME COMMAND
    782 ttyS0    Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 ttyS0 vt220
    786 tty1     Ss+    0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
   8662 pts/1    Ss+    0:01 /usr/bin/zsh -i
   9811 pts/2    Ss+    0:00 /usr/bin/zsh -i
  11213 pts/3    Ss     0:06 /usr/bin/zsh -i
  13863 pts/3    S      0:00 ./tsh -p
  13864 pts/3    Z      0:00 [echo] <defunct>
  14066 pts/3    S      0:00 ./tsh -p
  14067 pts/3    Z      0:00 [echo] <defunct>
  17696 pts/3    S+     0:00 make test13
  17697 pts/3    S+     0:00 /bin/sh -c ./sdriver.pl -t trace13.txt -s ./tsh -a "-p"
  17698 pts/3    S+     0:00 /usr/bin/perl ./sdriver.pl -t trace13.txt -s ./tsh -a -p
  17699 pts/3    S+     0:00 ./tsh -p
  17768 pts/3    R      0:00 /bin/ps a
```

correct!

## **Trace14**

```
#
# trace14.txt - Simple error handling
#
/bin/echo tsh> ./bogus
./bogus

/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

/bin/echo tsh> fg
fg

/bin/echo tsh> bg
bg

/bin/echo tsh> fg a
fg a

/bin/echo tsh> bg a
bg a

/bin/echo tsh> fg 9999999
fg 9999999

/bin/echo tsh> bg 9999999
bg 9999999

/bin/echo tsh> fg %2
fg %2

/bin/echo tsh> fg %1
fg %1

SLEEP 2
TSTP

/bin/echo tsh> bg %2
bg %2

/bin/echo tsh> bg %1
bg %1

/bin/echo tsh> jobs
jobs

```

the result should be:

```
➜  shlab-handout ./sdriver.pl -t trace14.txt -s ./tshref -a "-p"
#
# trace14.txt - Simple error handling
#
tsh> ./bogus
./bogus: Command not found
tsh> ./myspin 4 &
[1] (17820) ./myspin 4 &
tsh> fg
fg command requires PID or %jobid argument
tsh> bg
bg command requires PID or %jobid argument
tsh> fg a
fg: argument must be a PID or %jobid
tsh> bg a
bg: argument must be a PID or %jobid
tsh> fg 9999999
(9999999): No such process
tsh> bg 9999999
(9999999): No such process
tsh> fg %2
%2: No such job
tsh> fg %1
Job [1] (17820) stopped by signal 20
tsh> bg %2
%2: No such job
tsh> bg %1
[1] (17820) ./myspin 4 &
tsh> jobs
[1] (17820) Running ./myspin 4 &
```

finish the error message:

```c
void Execve(const char *filename, char *const argv[], char *const envp[])
{
    if (execve(filename, argv, envp) < 0)
    {
        fprintf(stdout, "%s: Command not found\n", filename);
        exit(1);
    }
}
```

```c
void do_bgfg(char **argv)
{
    struct job_t *job = NULL;
    int state;

    if (!strcmp(argv[0], "bg"))
        state = BG;
    else if (!strcmp(argv[0], "fg"))
        state = FG;
    else
        return;

    if (argv[1] == NULL)
    {
        printf("%s command requires PID or %%jobid argument\n", argv[0]);
        return;
    }

    if (argv[1][0] == '%')
    {
        int jid = atoi(argv[1] + 1);
        job = getjobjid(jobs, jid);
        if (job == NULL)
        {
            printf("%s: No such job\n", argv[1]);
            return;
        }
    }
    else if (isdigit(argv[1][0]))
    {
        pid_t pid = atoi(argv[1]);
        job = getjobpid(jobs, pid);
        if (job == NULL)
        {
            printf("(%s): No such process\n", argv[1]);
            return;
        }
    }
    else
    {
        printf("%s: argument must be a PID or %%jobid\n", argv[0]);
        return;
    }

    Kill(-(job->pid), SIGCONT);
    job->state = state;
    if (state == BG)
        printf("[%d] (%d) %s", job->jid, job->pid, job->cmdline);
    else if (state == FG)
        waitfg(job->pid);
    return;
}
```

test:

```
./sdriver.pl -t trace14.txt -s ./tsh -a "-p"
#
# trace14.txt - Simple error handling
#
tsh> ./bogus
./bogus: Command not found
tsh> ./myspin 4 &
[1] (19721) ./myspin 4 &
tsh> fg
fg command requires PID or %jobid argument
tsh> bg
bg command requires PID or %jobid argument
tsh> fg a
fg: argument must be a PID or %jobid
tsh> bg a
bg: argument must be a PID or %jobid
tsh> fg 9999999
(9999999): No such process
tsh> bg 9999999
(9999999): No such process
tsh> fg %2
%2: No such job
tsh> fg %1
Job [1] (19721) stopped by signal 20
tsh> bg %2
%2: No such job
tsh> bg %1
[1] (19721) ./myspin 4 &
tsh> jobs
[1] (19721) Running ./myspin 4 &
```

correct!

## **Trace15**

```
#
# trace15.txt - Putting it all together
#

/bin/echo tsh> ./bogus
./bogus

/bin/echo tsh> ./myspin 10
./myspin 10

SLEEP 2
INT

/bin/echo -e tsh> ./myspin 3 \046
./myspin 3 &

/bin/echo -e tsh> ./myspin 4 \046
./myspin 4 &

/bin/echo tsh> jobs
jobs

/bin/echo tsh> fg %1
fg %1

SLEEP 2
TSTP

/bin/echo tsh> jobs
jobs

/bin/echo tsh> bg %3
bg %3

/bin/echo tsh> bg %1
bg %1

/bin/echo tsh> jobs
jobs

/bin/echo tsh> fg %1
fg %1

/bin/echo tsh> quit
quit

```

test:

```
./sdriver.pl -t trace15.txt -s ./tsh -a "-p"
#
# trace15.txt - Putting it all together
#
tsh> ./bogus
./bogus: Command not found
tsh> ./myspin 10
Job [1] (20178) terminated by signal 2
tsh> ./myspin 3 &
[1] (20207) ./myspin 3 &
tsh> ./myspin 4 &
[2] (20209) ./myspin 4 &
tsh> jobs
[1] (20207) Running ./myspin 3 &
[2] (20209) Running ./myspin 4 &
tsh> fg %1
Job [1] (20207) stopped by signal 20
tsh> jobs
[1] (20207) Stopped ./myspin 3 &
[2] (20209) Running ./myspin 4 &
tsh> bg %3
%3: No such job
tsh> bg %1
[1] (20207) ./myspin 3 &
tsh> jobs
[1] (20207) Running ./myspin 3 &
[2] (20209) Running ./myspin 4 &
tsh> fg %1
tsh> quit
```

correct!

## **Trace16**

```
#
# trace16.txt - Tests whether the shell can handle SIGTSTP and SIGINT
#     signals that come from other processes instead of the terminal.
#

/bin/echo tsh> ./mystop 2 
./mystop 2

SLEEP 3

/bin/echo tsh> jobs
jobs

/bin/echo tsh> ./myint 2 
./myint 2

```

result:

```
➜  shlab-handout make test16
./sdriver.pl -t trace16.txt -s ./tsh -a "-p"
#
# trace16.txt - Tests whether the shell can handle SIGTSTP and SIGINT
#     signals that come from other processes instead of the terminal.
#
tsh> ./mystop 2
Job [1] (20290) stopped by signal 20
tsh> jobs
[1] (20290) Stopped ./mystop 2
tsh> ./myint 2
Job [2] (20337) terminated by signal 2
```

correct!