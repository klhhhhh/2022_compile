# NKUER4
2022 National College Student Computer System Ability Training Competition Compilation System Design Competition Project
Team School: Nankai University
Team Name: NKUER4
Team members: Shi Haoming, Yan Shihui, Lin Kunmei, Junyi
Guidance Teacher: Wang Gang, Li Zhongwei

## Project Introduction
A single front-end and single back-end compiler that compiles SysY language and generates arm architecture assembly.
There are three parts: front-end, mid-range, and back-end. The front-end is implemented based on lex and yacc, and the intermediate code is compatible with llvm IR. The back-end generates arm assembly code.
Implemented some optimization algorithms.

## Project construction
Execute at the root directory of the project
```
make app
```
## Project operation
```
Usage: build/compiler [options] file
Options: 
    -a              Generate ast.
    -i              Generate IR.
    -o              Specify output file.
    -O              Generate optimized code.
    -S              Generate Arm assemble code.
```

## Project Properties
### Intermediate representation optimization
*Localization of global variables

*Mem2Reg

*Function inlining

*Constant propagation

*Rare condition constant propagation

*Common subexpression deletion

*Dead code deletion

*Global Value Number

*Loop optimization

*Instruction scheduling

*Control flow optimization

### Target code optimization

*Dead code deletion
*Peephole optimization
*Local value number
*Control flow optimization
*Multiplication and division optimization
*Register allocation

For detailed introduction, please refer to[文档](./NKUER4_final.pdf)

## Project demonstration
For SysY code:
``` c
// test if-else
int ifElse() {
  int a;
  a = 5;
  if (a == 5) {
    a = 25;
  } else {
    a = a * 2;
  }
  return (a);
}


int main() {
  return (ifElse());
}
```

The syntax tree generated by the compiler is as follows:
```
program
    FunctionDefine	function name: @ifElse	type: i32()
        CompoundStmt
            DeclStmt
                Id	name: a	scope: 2	type: i32
            AssignStmt
                Id	name: a	scope: 2	type: i32
                IntegerLiteral	value: 5	type: i32
            IfElseStmt
                BinaryExpr	op: equal	type: i1
                    Id	name: a	scope: 2	type: i32
                    IntegerLiteral	value: 5	type: i32
                CompoundStmt
                    AssignStmt
                        Id	name: a	scope: 2	type: i32
                        IntegerLiteral	value: 25	type: i32
                CompoundStmt
                    AssignStmt
                        Id	name: a	scope: 2	type: i32
                        BinaryExpr	op: mul	type: i32
                            Id	name: a	scope: 2	type: i32
                            IntegerLiteral	value: 2	type: i32
            ReturnStmt
                Id	name: a	scope: 2	type: i32
    FunctionDefine	function name: @main	type: i32()
        CompoundStmt
            ReturnStmt
                CallExpr	function name: @ifElse	scope: 0	type: i32()

```
Intermediate code generated:
``` llvm
define i32 @ifElse() {
B10:
  %t11 = alloca i32, align 4
  store i32 5, i32* %t11, align 4
  %t2 = load i32, i32* %t11, align 4
  %t3 = icmp eq i32 %t2, 5
  br i1 %t3, label %B12, label %B17
B12:                               	; preds = %B10
  store i32 25, i32* %t11, align 4
  br label %B14
B14:                               	; preds = %B12, %B13
  %t8 = load i32, i32* %t11, align 4
  ret i32 %t8
B17:                               	; preds = %B10
  br label %B13
B13:                               	; preds = %B17
  %t6 = load i32, i32* %t11, align 4
  %t7 = mul i32 %t6, 2
  store i32 %t7, i32* %t11, align 4
  br label %B14
}
define i32 @main() {
B18:
  %t9 = call i32 @ifElse()
  ret i32 %t9
}

```

Unoptimized assembly code generated:
``` arm
	.cpu cortex-a72
	.arch armv8-a
	.fpu vfpv3-d16
	.arch_extension crc
	.text
	.global ifElse
	.type ifElse , %function
ifElse:
	push {r3, r4, r5, r6, fp, lr}
	mov fp, sp
	sub sp, sp, #8
.L10:
	ldr r4, =5
	str r4, [fp, #-4]
	ldr r4, [fp, #-4]
	cmp r4, #5
	moveq r4, #1
	movne r4, #0
	beq .L12
	b .L17
.L12:
	ldr r4, =25
	str r4, [fp, #-4]
	b .L14
.L14:
	ldr r4, [fp, #-4]
	mov r0, r4
	add sp, sp, #8
	pop {r3, r4, r5, r6, fp, lr}
	bx lr
.L17:
	b .L13
.L13:
	ldr r4, [fp, #-4]
	ldr r5, =2
	mul r6, r4, r5
	str r6, [fp, #-4]
	b .L14

	.global main
	.type main , %function
main:
	push {r3, r4, fp, lr}
	mov fp, sp
.L18:
	bl ifElse
	mov r4, r0
	mov r0, r4
	pop {r3, r4, fp, lr}
	bx lr

	.ident "NKUER4"
```

After optimization, generate:
``` arm
	.cpu cortex-a72
	.arch armv8-a
	.fpu vfpv3-d16
	.arch_extension crc
	.text
	.global main
	.type main , %function
main:
	push {fp, lr}
	mov fp, sp
.L18:
	mov r0, #25
	pop {fp, lr}
	bx lr

	.ident "NKUER4"

```

## Project framework
The basic framework originates from[Emanual20/2021NKUCS-Compilers-Lab: For 2021Fall NKUCS, the course of the principle of compilers. (github.com)](https://github.com/Emanual20/2021NKUCS-Compilers-Lab)
