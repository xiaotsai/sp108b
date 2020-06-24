# Compiler

## 語法

```
PROG = STMTS
BLOCK = { STMTS }
STMTS = STMT*
STMT = WHILE | BLOCK | ASSIGN
WHILE = while (E) STMT
ASSIGN = id '=' E;
E = F (op E)*
F = (E) | Number | Id
```

## 執行結果

```
user@DESKTOP-96FRN6B MINGW64 /d/ccc/book/sp/code/c/02-compiler/03-compiler
$ make clean
rm -f *.o *.exe

user@DESKTOP-96FRN6B MINGW64 /d/ccc/book/sp/code/c/02-compiler/03-compiler
$ make
gcc -std=c99 -O0 lexer.c compiler.c main.c -o compiler

user@DESKTOP-96FRN6B MINGW64 /d/ccc/book/sp/code/c/02-compiler/03-compiler
$ ./compiler test/while.c
while (i<10) i = i + 1;

========== lex ==============
token=while
token=(
token=i
token=<
token=10
token=)
token=i
token==
token=i
token=+
token=1
token=;
========== dump ==============
0:while
1:(
2:i
3:<
4:10
5:)
6:i
7:=
8:i
9:+
10:1
11:;
============ parse =============
(L0)
t0 = i
t1 = 10
t2 = t0 < t1
goto L1 if T2
t3 = i
t4 = 1
t5 = t3 + t4
i = t5
goto L0
(L1)
``` 


```
PS C:\ccc\sp108b\homeWork\03-compiler> ./compiler test/if.c
a = 3;
b = 5;
if (a > b)
    t = a;
else
   t = b;
========== lex ==============
token=a
token==
token=3
token=;
token=b
token==
token=5
token=;
token=t
token==
token=0
token=;
token=if
token=(
token=a
token=>
token=b
token=)
token=t
token==
token=a
token=;
token=else
token=t
token==
token=b
token=;
========== dump ==============
0:a
1:=
2:3
3:;
4:b
5:=
6:5
7:;
8:t
9:=
10:0
11:;
12:if
13:(
14:a
15:>
16:b
17:)
18:t
19:=
20:a
21:;
22:else
23:t
24:=
25:b
26:;
============ parse =============
t0 = 3
b = t1
t2 = 0
t3 = a
t4 = b
t5 = t3 > t4
if not T5 goto L0
t6 = a
t = t6
goto L1
(L0)
t7 = b
t = t7

```