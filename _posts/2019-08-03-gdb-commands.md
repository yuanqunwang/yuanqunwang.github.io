---
title: Useful gdb commands
categories: Tools
tags: [gdb, Tools, Tips]
---
## start execution

* start debug: `gdb executeble_name`
* run: `(gdb) run arg1 "arg2"`
* restart program running in the debugger: `(gdb) kill`
* exit debugger: `(gdb) quit / q`

## jump around

* line by line, jump into call function: `(gdb) step`
* line by line, not jump into call function: `(gbd) next`
* return from calling function: `(gdb) return/ finish`
* examine variable: `(gdb) print x`
* modify variable: `(gdb) set x = 3`
* call function: `(gdb) call abort()`

## stack trace

* show stack trace: `(gdb) backtrace/ bt`
* change into other frame: `(gdb) frame frame_num`
* show frame info: `(gdb) info frame`
* show stack local variables: `(gdb) info locals`
* show function arguments: `(gdb) info args`

## break point

* show breakpoint info: `(gdb) info break`
* place a breakpoint at specified function: `(gdb) break function_name`
* place a breakpoint at specified line: `(gdb) break line_num`
* place a breakpoint at the specified line within the specified source file: `(gdb) file_name:line_num`
* disable breakpoint identified by breaknum: `(gdb) disable breaknum`
* enable breakpoint identified by breaknum: `(gdb) enable breaknum`
* skip a breakpoint a certain number of times: `(gdb) ingore breaknum times_to_skip`, eg: `(gdb) ingore 2 5` will ignore next 5 crossings of breakpoint 2.

## advanced

### examine memory

Use the **x** command to examine memory. The syntax for the x command is x/FMT ADDRESS. The FMT field is a count followed by a format letter and a size letter. There are many options here, use the help command 'help x' to see them all. The ADDRESS argument can either be a symbol name, such as a variable, or a memory address.

If we have `char *s = "Hello World\n"`, some uses of the x command could be:

Examine the variable as a string:

> ``
>
> ```
> (gdb) x/s s
> 0x8048434 <_IO_stdin_used+4>:    "Hello World\n"
> ```

Examine the variable as a character:

> ``
>
> ```
> (gdb) x/c s
> 0x8048434 <_IO_stdin_used+4>:   72 'H'
> ```

Examine the variable as 4 characters:

> ``
>
> ```
> (gdb) x/4c s
> 0x8048434 <_IO_stdin_used+4>:   72 'H'  101 'e' 108 'l' 108 'l'
> ```

Examine the first 32 bits of the variable:

> ``
>
> ```
> (gdb) x/t s
> 0x8048434 <_IO_stdin_used+4>:   01101100011011000110010101001000
> ```

Examine the first 24 bytes of the variable in hex:

> ``
>
> ```
> (gdb) x/3x s
> 0x8048434 <_IO_stdin_used+4>:   0x6c6c6548      0x6f57206f      0x0a646c72
> ```





Use the **list** command to have gdb print out the lines of code above and below the line the program is stopped at. In the example below, the breakpoint is on line 8.

> ``
>
> ```c
> (gdb) list
> 3       int main(int argc, char **argv)
> 4       {
> 5         int x = 30;
> 6         int y = 10;
> 7       
> 8         x = y;
> 9       
> 10        return 0;
> 11      }
> ```







http://www.unknownroad.com/rtfm/gdbtut/gdbtoc.html
