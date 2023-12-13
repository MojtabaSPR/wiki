# Redirecting Standard IO

In Bash and other Linux shells, when a program is executed, it uses three standard I/O streams. Each stream is represented by a numeric file descriptor:

0. stdin, the standard input stream.
1. stdout, the standard output stream.
2. stderr, the standard error stream.

| Operator | Usage |
| -------- | ----- |
| > | Redirect STDOUT to a file; Overwrite if exists |
| >> | Redirect STDOUT to a file; Append if exists |
| 2> | Redirect STDERR to a file; Overwrite if exists |
| 2>> | Redirect STDERR to a file; Append if exists |
| &> | Redirect both STDOUT and STDERR; Overwrite if exists |
| &>> | Redirect both STDOUT and STDERR; Append if exists |
| < | Redirect STDIN from a file |
| <> |Redirect STDIN from the file and send the STDOUT to it |

Streams can be redirected using the n> operator, where n is the file descriptor number.

When n is omitted, it defaults to 1, the standard output stream. For example, the following two commands are the same; both will redirect the command output (stdout) to the file.

```bash
~: ls > output.txt
~: ls 1> output.txt
```

```bash
# redirect the standard error(overwrite)
~: ls 2> output.txt

# redirect the standard error(append)
~: ls 2>> output.txt

# redirect stderr and stdout to two separate files
~: command 2> error.txt 1> output.txt

# redirect stderr and stdout to same file(overwrite)
~: command &> ouput.txt

# redirect stderr and stdout to same file(append)
~: command &>> ouput.txt
```

It is also possible to use **&1** and **&2** and **&0** to refer to the target of STDOUT, STDERR & STDIN.

In this format **&1** refers to stdout and **&2** to stderr and **&0** to stdin.

We can use &1,&2 after > to refer the specified standard io.

```bash
# redirect stderr to stdout (as stdout is redirected to output.txt, so stderr will also be redirected to output.txt)
# &1 work as a pointer to stdout
~: command  > output.txt  2>&1    ## same as: command &> output.txt

# redirect stdout to stderr (as stderr is redirected to output.txt, so stdout will also be redirected to output.txt)
# &2 work as a pointer to stderr
~: command  2> output.txt  >&2    ## same as: command &> output.txt

```

**/dev/null:**

In Linux /dev/null device is like a blackhole.
To suppress the error messages from being displayed on the screen, redirect stderr to /dev/null:

```bash
~: command > output.txt 2> /dev/null
```
