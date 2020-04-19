---
title: MakeFile Rules
categories: Tools
tags: [MakeFile, Tools, Tips]
---
# makefile rule

## general rule

```shell
target: component \
           component
   Tab ↹command ;          \
   Tab ↹command |          \
   Tab ↹piped-command
```

## sufix rule

Suffix rules have "targets" with names in the form `.FROM.TO` and are used to launch actions based on file extension. In the command lines of suffix rules, POSIX specifies[[37\]](https://en.wikipedia.org/wiki/Make_(software)#cite_note-posix-macros-37) that the internal macro `$<` refers to the first prerequisite and `$@` refers to the target. In this example, which converts any HTML file into text, the shell redirection token `>` is part of the command line whereas `$<` is a macro referring to the HTML file:

```shell
.SUFFIXES: .txt .html

# From .html to .txt
.html.txt:
	lynx -dump $<   >   $@
```

When called from the command line, the above example expands.

```shell
$ make -n file.txt
lynx -dump file.html > file.txt
```

## pattern rule

Suffix rules cannot have any prerequisites of their own.[[38\]](https://en.wikipedia.org/wiki/Make_(software)#cite_note-38) If they have any, they are treated as normal files with unusual names, not as suffix rules. GNU Make supports suffix rules for compatibility with old makefiles but otherwise encourages usage of *pattern rules*.[[39\]](https://en.wikipedia.org/wiki/Make_(software)#cite_note-39)

A pattern rule looks like an ordinary rule, except that its target contains exactly one character '%'. The target is considered a pattern for matching file names: the '%' can match any substring of zero or more characters,[[40\]](https://en.wikipedia.org/wiki/Make_(software)#cite_note-40) while other characters match only themselves. The prerequisites likewise use '%' to show how their names relate to the target name.

The above example of a suffix rule would look like the following pattern rule:

```shell
# From %.html to %.txt
%.txt : %.html 
	lynx -dump $< > $@
```

