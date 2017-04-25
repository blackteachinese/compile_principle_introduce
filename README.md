# 编译原理入门

-------

# 编译器结构
编译器由两个部分组成：分析 (analysis)部分和综合 (synthesis)部分。

* 分析部分也称前端 (front end)，它把源程序分解成为多个组成要素，并在这些要素之上加上语法结构，然后使用这个结构来创建该源程序的一个中间表示。
* 分析部分还会收集有关源程序的信息，放在一个称为符号表 (symbol table)的数据结构中。符号表将和中间表示形式一起传送给综合部分。
* 综合部分也称后端 (back end)，根据中间表示和符号表中的信息来构造用户期待的目标程序。

# 编译步骤
* 词法分析
    * 词法分析器读入组成源程序的字符流，将他们切分组织成有意义的词法单元 (token)的序列作为输出。
* 语法分析
    * 语法分析器根据各个词法单元，创建树形中间表示，常用语法树 (syntax tree)。语法分析只判断源程序在结构上是否正确。
* 语义分析
    * 使用语法树和符号表中的信息来检查源程序是否和语言定义的语义一致。语义分析是判断结构正确的源程序所表达的意义是否正确，其中一个重要部分是类型检查 (type checking)。
* 中间代码生成
    * 编译器根据语法树和符号表，生成一个明确的低级的或类机器语言的中间表示 (intermediate representation)。三地址代码 (three-address code)是一种常见的中间表示形式。 
* 代码优化（机器无关）
    * 机器无关的代码优化步骤试图改进中间代码，以便生成更好的目标代码。 
* 代码生成器
    * 代码生成器以源程序的中间表示形式作为输入，并把它映射到目标语言。如果目标语言是机器代码，就必须为程序使用的每个变量选择寄存器或内存位置。然后，中间指令被翻译成为能够完成相同任务的机器指令序列 
* 代码优化（机器相关）
    * 优化目标机器语言 

## 查看编译步骤
创建文件 compileText.c

```
  #include <stdio.h>
    int main (int argc, char *argv[]) {
    printf("Hello workd");
    return 0;
  }
```

使用clnag/llvm,输入下面命令查看编译步骤：

```
clang -ccc-print-phases compileTest.c
```

输出得到编译步骤：

```
0: input, "compileTest.c", c
1: preprocessor, {0}, cpp-output
2: compiler, {1}, ir
3: backend, {2}, assembler
4: assembler, {3}, object
5: linker, {4}, image
6: bind-arch, "x86_64", {5}, image
```


# 符号表
符号表记录源程序中使用的变量的名字，并收集和每个名字的各种属性有关的信息。如一个名字的存储分配、类型、作用域、参数数量和类型、返回类型等。
符号表为每个变量名字创建一个记录条目。编译器可以向记录中快速存放和获取数据。

# Example
## 词法分析
创建文件 compileText.c

```
  #include <stdio.h>
    int main (int argc, char *argv[]) {
    printf("Hello workd");
    return 0;
  }
```

输入以下命令：

```
clang -fmodules -fsyntax-only -Xclang -dump-tokens compileTest.c
```

输出token序列：

```
annot_module_include '#include <s'		Loc=<compileTest.c:1:1>
int 'int'		Loc=<compileTest.c:2:1>
identifier 'main'	 [LeadingSpace]	Loc=<compileTest.c:2:5>
l_paren '('	 [LeadingSpace]	Loc=<compileTest.c:2:10>
int 'int'		Loc=<compileTest.c:2:11>
identifier 'argc'	 [LeadingSpace]	Loc=<compileTest.c:2:15>
comma ','		Loc=<compileTest.c:2:19>
char 'char'	 [LeadingSpace]	Loc=<compileTest.c:2:21>
star '*'	 [LeadingSpace]	Loc=<compileTest.c:2:26>
identifier 'argv'		Loc=<compileTest.c:2:27>
l_square '['		Loc=<compileTest.c:2:31>
r_square ']'		Loc=<compileTest.c:2:32>
r_paren ')'		Loc=<compileTest.c:2:33>
l_brace '{'	 [LeadingSpace]	Loc=<compileTest.c:2:35>
identifier 'printf'	 [StartOfLine] [LeadingSpace]	Loc=<compileTest.c:3:5>
l_paren '('		Loc=<compileTest.c:3:11>
string_literal '"Hello workd"'		Loc=<compileTest.c:3:12>
r_paren ')'		Loc=<compileTest.c:3:25>
semi ';'		Loc=<compileTest.c:3:26>
return 'return'	 [StartOfLine] [LeadingSpace]	Loc=<compileTest.c:4:5>
numeric_constant '0'	 [LeadingSpace]	Loc=<compileTest.c:4:12>
semi ';'		Loc=<compileTest.c:4:13>
r_brace '}'	 [StartOfLine]	Loc=<compileTest.c:5:1>
eof ''		Loc=<compileTest.c:5:2>
```

## 语法分析
输入以下命令：

```
clang -fmodules -fsyntax-only -Xclang -ast-dump compileTest.c
```

输出如下：

```
TranslationUnitDecl 0x7f8e4d81fad0 <<invalid sloc>> <invalid sloc>
|-TypedefDecl 0x7f8e4d820018 <<invalid sloc>> <invalid sloc> implicit __int128_t '__int128'
| `-BuiltinType 0x7f8e4d81fd40 '__int128'
|-TypedefDecl 0x7f8e4d820078 <<invalid sloc>> <invalid sloc> implicit __uint128_t 'unsigned __int128'
| `-BuiltinType 0x7f8e4d81fd60 'unsigned __int128'
|-TypedefDecl 0x7f8e4d820368 <<invalid sloc>> <invalid sloc> implicit __NSConstantString 'struct __NSConstantString_tag'
| `-RecordType 0x7f8e4d820170 'struct __NSConstantString_tag'
|   `-Record 0x7f8e4d8200c8 '__NSConstantString_tag'
|-TypedefDecl 0x7f8e4d8203f8 <<invalid sloc>> <invalid sloc> implicit __builtin_ms_va_list 'char *'
| `-PointerType 0x7f8e4d8203c0 'char *'
|   `-BuiltinType 0x7f8e4d81fb60 'char'
|-TypedefDecl 0x7f8e4d8206d8 <<invalid sloc>> <invalid sloc> implicit __builtin_va_list 'struct __va_list_tag [1]'
| `-ConstantArrayType 0x7f8e4d820680 'struct __va_list_tag [1]' 1
|   `-RecordType 0x7f8e4d8204f0 'struct __va_list_tag'
|     `-Record 0x7f8e4d820448 '__va_list_tag'
|-ImportDecl 0x7f8e4d98ada0 <compileTest.c:1:1> col:1 implicit Darwin.C.stdio
|-FunctionDecl 0x7f8e4d98b040 <line:2:1, line:5:1> line:2:5 main 'int (int, char **)'
| |-ParmVarDecl 0x7f8e4d98ade8 <col:11, col:15> col:15 argc 'int'
| |-ParmVarDecl 0x7f8e4d98af00 <col:21, col:32> col:27 argv 'char **':'char **'
| `-CompoundStmt 0x7f8e4d9e9ab0 <col:35, line:5:1>
|   |-CallExpr 0x7f8e4d9e9a18 <line:3:5, col:25> 'int'
|   | |-ImplicitCastExpr 0x7f8e4d9e9a00 <col:5> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
|   | | `-DeclRefExpr 0x7f8e4d98b538 <col:5> 'int (const char *, ...)' Function 0x7f8e4d98b158 'printf' 'int (const char *, ...)'
|   | `-ImplicitCastExpr 0x7f8e4d9e9a60 <col:12> 'const char *' <BitCast>
|   |   `-ImplicitCastExpr 0x7f8e4d9e9a48 <col:12> 'char *' <ArrayToPointerDecay>
|   |     `-StringLiteral 0x7f8e4d98b598 <col:12> 'char [12]' lvalue "Hello workd"
|   `-ReturnStmt 0x7f8e4d9e9a98 <line:4:5, col:12>
|     `-IntegerLiteral 0x7f8e4d9e9a78 <col:12> 'int' 0
`-<undeserialized declarations>
```

## 中间代码生成
输入命令：

```
clang -S -emit-llvm compileTest.c -o compileTest.ll
```

得到compileTest.ll文件：

```
; ModuleID = 'compileTest.c'
source_filename = "compileTest.c"
target datalayout = "e-m:o-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-apple-macosx10.12.0"

@.str = private unnamed_addr constant [12 x i8] c"Hello workd\00", align 1

; Function Attrs: nounwind ssp uwtable
define i32 @main(i32, i8**) #0 {
  %3 = alloca i32, align 4
  %4 = alloca i32, align 4
  %5 = alloca i8**, align 8
  store i32 0, i32* %3, align 4
  store i32 %0, i32* %4, align 4
  store i8** %1, i8*** %5, align 8
  %6 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([12 x i8], [12 x i8]* @.str, i32 0, i32 0))
  ret i32 0
}

declare i32 @printf(i8*, ...) #1

attributes #0 = { nounwind ssp uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="penryn" "target-features"="+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #1 = { "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="penryn" "target-features"="+cx16,+fxsr,+mmx,+sse,+sse2,+sse3,+sse4.1,+ssse3,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0}
!llvm.ident = !{!1}

!0 = !{i32 1, !"PIC Level", i32 2}
!1 = !{!"Apple LLVM version 8.1.0 (clang-802.0.41)"}

```

## 生成汇编

输入命令：

```
clang -S compileTest.c -o compileTest.s
```

得到compileTest.s:

```
	.section	__TEXT,__text,regular,pure_instructions
	.macosx_version_min 10, 12
	.globl	_main
	.p2align	4, 0x90
_main:                                  ## @main
	.cfi_startproc
    ## BB#0:
	pushq	%rbp
Ltmp0:
	.cfi_def_cfa_offset 16
Ltmp1:
	.cfi_offset %rbp, -16
	movq	%rsp, %rbp
Ltmp2:
	.cfi_def_cfa_register %rbp
	subq	$32, %rsp
	leaq	L_.str(%rip), %rax
	movl	$0, -4(%rbp)
	movl	%edi, -8(%rbp)
	movq	%rsi, -16(%rbp)
	movq	%rax, %rdi
	movb	$0, %al
	callq	_printf
	xorl	%ecx, %ecx
	movl	%eax, -20(%rbp)         ## 4-byte Spill
	movl	%ecx, %eax
	addq	$32, %rsp
	popq	%rbp
	retq
	.cfi_endproc

	.section	__TEXT,__cstring,cstring_literals
L_.str:                                 ## @.str
	.asciz	"Hello workd"


.subsections_via_symbols

```

## 生成目标文件

输入命令：

```
clang -fmodules -c compileTest.c -o compileTest.o
```

输出文件compileTest.o:

```
cffa edfe 0700 0001 0300 0000 0100 0000
0400 0000 0002 0000 0020 0000 0000 0000
1900 0000 8801 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
a000 0000 0000 0000 2002 0000 0000 0000
a000 0000 0000 0000 0700 0000 0700 0000
0400 0000 0000 0000 5f5f 7465 7874 0000
0000 0000 0000 0000 5f5f 5445 5854 0000
0000 0000 0000 0000 0000 0000 0000 0000
3400 0000 0000 0000 2002 0000 0400 0000
c002 0000 0200 0000 0004 0080 0000 0000
0000 0000 0000 0000 5f5f 6373 7472 696e
6700 0000 0000 0000 5f5f 5445 5854 0000
0000 0000 0000 0000 3400 0000 0000 0000
0c00 0000 0000 0000 5402 0000 0000 0000
0000 0000 0000 0000 0200 0000 0000 0000
0000 0000 0000 0000 5f5f 636f 6d70 6163
745f 756e 7769 6e64 5f5f 4c44 0000 0000
0000 0000 0000 0000 4000 0000 0000 0000
2000 0000 0000 0000 6002 0000 0300 0000
d002 0000 0100 0000 0000 0002 0000 0000
0000 0000 0000 0000 5f5f 6568 5f66 7261
6d65 0000 0000 0000 5f5f 5445 5854 0000
0000 0000 0000 0000 6000 0000 0000 0000
4000 0000 0000 0000 8002 0000 0300 0000
0000 0000 0000 0000 0b00 0068 0000 0000
0000 0000 0000 0000 2400 0000 1000 0000
000c 0a00 0000 0000 0200 0000 1800 0000
d802 0000 0200 0000 f802 0000 1000 0000
0b00 0000 5000 0000 0000 0000 0000 0000
0000 0000 0100 0000 0100 0000 0100 0000
0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
5548 89e5 4883 ec20 488d 0525 0000 00c7
45fc 0000 0000 897d f848 8975 f048 89c7
b000 e800 0000 0031 c989 45ec 89c8 4883
c420 5dc3 4865 6c6c 6f20 776f 726b 6400
0000 0000 0000 0000 3400 0000 0000 0001
0000 0000 0000 0000 0000 0000 0000 0000
1400 0000 0000 0000 017a 5200 0178 1001
100c 0708 9001 0000 2400 0000 1c00 0000
80ff ffff ffff ffff 3400 0000 0000 0000
0041 0e10 8602 430d 0600 0000 0000 0000
2300 0000 0100 002d 0b00 0000 0200 0015
0000 0000 0100 0006 0100 0000 0f01 0000
0000 0000 0000 0000 0700 0000 0100 0000
0000 0000 0000 0000 005f 6d61 696e 005f
7072 696e 7466 0000
```

## 生成可执行文件
输入命令：

```
clang compileTest.o -o compileTest
```

输出可执行文件compileTest,执行如下:

```
./compileTest
```

```
输出
Hello world
```



# 编译器 (compiler)

编译器是一个程序，可以阅读以某一种语言(源语言)编写的程序，并把该程序翻译成为一个等价的、用另一种语言(目标语言)编写的程序。

# 解释器 (interpreter)

解释器不通过翻译的方式生成目标程序，直接利用用户提供的输入执行源程序中指定的操作。

## 编译器和解析器的区别

编译型语言在编译过程中生成目标平台的指令，解释型语言在运行过程中才生成目标平台的指令。
虚拟机的任务是在运行过程中将中间代码翻译成目标平台的指令。

# 语言处理流程
预处理器 (preprocessor)
编译器 (compiler)
汇编器 (assembler)
链接器 (linker)/加载器 (loader)

# 相关名词
* 词法单元 (token)
* 抽象语法树 (AST - abstract syntax tree)
* 类型检查 (type checking)
* 三地址代码 (three-address code)
* 中间表示形式 (IR - intermediate representation)
    * 中间语言 (IL - intermediate language)
    * 字节码 (bytecode)

