# RISC-V 汇编程序员手册

# 版权和许可信息

RISC-V 汇编程序员手册

© 2017 Palmer Dabbelt [palmer@dabbelt.com](mailto:palmer@dabbelt.com) © 2017 Michael Clark [michaeljclark@mac.com](mailto:michaeljclark@mac.com) © 2017 Alex Bradbury [asb@lowrisc.org](mailto:asb@lowrisc.org)

根据知识共享署名 4.0 国际许可 (CC-BY 4.0) 许可。完整的许可文本可在 https://creativecommons.org/licenses/by/4.0/ 获得。

# 命令行参数

我认为加强 binutils 文档而不是在这里复制它可能会更好。

# 寄存器

寄存器是任何处理器中最重要的部分。 RISC-V 定义了多种类型，具体取决于包含哪些扩展：通用寄存器（与程序计数器）、控制寄存器、浮点寄存器（F 扩展）和向量寄存器（V 扩展）。

## 通用寄存器

RV32I 基本整数 ISA 包括 32 个寄存器，命名为`x0`到`x31` 。与 ARM-32 等其他处理器不同，程序计数器`PC`与这些寄存器是分开的。第一个寄存器`x0`有一个特殊的功能：读取它总是返回 0 并且写入它被忽略。正如我们稍后将看到的，这允许各种技巧和简化。

实际使用中，程序员不会使用这种表示法描述寄存器。虽然`x1`到`x31`就处理器而言都是同等的通用寄存器，但按照惯例，某些寄存器用于特殊任务。在汇编程序中，它们被赋予标准化名称作为 RISC-V**应用程序二进制接口**(ABI) 的一部分。这是您通常会在代码中看到的内容。如果您真的想查看数字式的寄存器名称，objdump 的`-M`参数将提供它们。

登记 | ABI | 按惯例使用 | 保留值？
:-- | :-- | :-- | ---
x0 | zero | 硬件横零，忽略写入 | *n/a*
x1 | ra | 跳转的返回地址 | 否
x2 | sp | 栈指针 | 是
x3 | gp | 全局指针 | *n/a*
x4 | tp | 线程指针 | *n/a*
x5 | t0 | 临时寄存器 0 | 否
x6 | t1 | 临时寄存器 1 | 否
x7 | t2 | 临时寄存器 2 | 否
x8 | s0*或*fp | 保存用寄存器 0*或*帧指针 | 是
x9 | s1 | 保存用寄存器 1 | 是
x10 | a0 | 返回值*或*函数参数 0 | 否
x11 | a1 | 返回值*或*函数参数 1 | 否
x12 | a2 | 函数参数 2 | 否
x13 | a3 | 函数参数 3 | 否
x14 | a4 | 函数参数 4 | 否
x15 | a5 | 函数参数 5 | 否
x16 | a6 | 函数参数 6 | 否
x17 | a7 | 函数参数 7 | 否
x18 | s2 | 保存用寄存器 2 | 是
x19 | s3 | 保存用寄存器 3 | 是
x20 | s4 | 保存用寄存器 4 | 是
x21 | s5 | 保存用寄存器 5 | 是
x22 | s6 | 保存用寄存器 6 | 是
x23 | s7 | 保存用寄存器 7 | 是
x24 | s8 | 保存用寄存器 8 | 是
x25 | s9 | 保存用寄存器 9 | 是
x26 | s10 | 保存用寄存器 10 | 是
x27 | s11 | 保存用寄存器 11 | 是
x28 | t3 | 临时寄存器 3 | 否
x29 | t4 | 临时寄存器 4 | 否
x30 | t5 | 临时寄存器 5 | 否
x31 | t6 | 临时寄存器 6 | 否
pc | *无* | 程序计数器 | *n/a*

*RV32I 的寄存器。基于 RISC-V 文档以及 Patterson 和 Waterman “The RISC-V Reader” (2017)*

作为一般规则，**保存用寄存器**`s0`到`s11`在函数调用中保留，而**参数寄存器**`a0`到`a7`和**临时寄存器**`t0`到`t6`则不保留。稍后将更详细地讨论各种专用寄存器的使用，例如按照惯例使用`sp` 。

## 控制寄存器

(待定)

## 浮点寄存器 (RV32F)

(待定)

## 矢量寄存器 (RV32V)

(待定)

# 寻址

像 %pcrel_lo() 这样的寻址格式。我们可以链接到 RISC-V PS ABI 文档来描述重定位的实际作用。

# 指令系统

官方规范页面：

- https://riscv.org/specifications/

最新规范草案仓库：

- https://github.com/riscv/riscv-isa-manual

## 指令

# RISC-V ISA 规范

https://riscv.org/specifications/

## 指令别名

来自 opcodes/riscv-opc.c 的别名

为了更好地诊断程序流到达意外位置的情况，您可能希望在那里发出一条已知会陷入陷阱的指令。您可以使用`UNIMP`伪指令，它应该会在几乎所有系统中捕获。该指令的*事实上的*标准实现是：

- `C.UNIMP` ： `0000` 。全零不是有效指令。任何陷入无效指令的系统都会因此陷入这种`UNIMP`指令形式。尽管不是有效指令，但它仍然符合 16 位（压缩）指令格式，因此`0000 0000`被解释为两条 16 位`UNIMP`指令。

- `UNIMP` ： `C0001073` 。这是`CSRRW x0, cycle, x0`的别名。因为`cycle`是一个只读的CSR，那么（无论这个CSR是否存在）尝试写入它都会产生一个非法指令异常。当针对没有 C 扩展的系统或使用`.option norvc`指令时，会生成这种 32 位形式的`UNIMP` 。

## 伪操作

RISC-V-specific 和 GNU .-prefixed 选项。

下表列出了汇编器指示：

指示 | 参数 | 描述
:-- | :-- | :--
.align | integer | align to power of 2 (alias for .p2align)
.file | "filename" | emit filename FILE LOCAL symbol table
.globl | symbol_name | emit symbol_name to symbol table (scope GLOBAL)
.local | symbol_name | emit symbol_name to symbol table (scope LOCAL)
.comm | symbol_name,size,align | emit common object to .bss section
.common | symbol_name,size,align | emit common object to .bss section
.ident | "string" | accepted for source compatibility
.section | [{.text,.data,.rodata,.bss}] | emit section (if not present, default .text) and make current
.size | symbol, symbol | accepted for source compatibility
.text |  | emit .text section (if not present) and make current
.data |  | emit .data section (if not present) and make current
.rodata |  | emit .rodata section (if not present) and make current
.bss |  | emit .bss section (if not present) and make current
.string | "string" | emit string
.asciz | "string" | emit string (alias for .string)
.equ | name, value | constant definition
.macro | name arg1 [, argn] | begin macro definition \argname to substitute
.endm |  | end macro definition
.type | symbol, @function | accepted for source compatibility
.option | {rvc,norvc,pic,nopic,relax,norelax,push,pop} | RISC-V options. Refer to [.option](#.option) for a more detailed description.
.byte | expression [, expression]* | 8-bit comma separated words
.2byte | expression [, expression]* | 16-bit comma separated words
.half | expression [, expression]* | 16-bit comma separated words
.short | expression [, expression]* | 16-bit comma separated words
.4byte | expression [, expression]* | 32-bit comma separated words
.word | expression [, expression]* | 32-bit comma separated words
.long | expression [, expression]* | 32-bit comma separated words
.8byte | expression [, expression]* | 64-bit comma separated words
.dword | expression [, expression]* | 64-bit comma separated words
.quad | expression [, expression]* | 64-bit comma separated words
.dtprelword | expression [, expression]* | 32-bit thread local word
.dtpreldword | expression [, expression]* | 64-bit thread local word
.sleb128 | expression | signed little endian base 128, DWARF
.uleb128 | expression | unsigned little endian base 128, DWARF
.p2align | p2,[pad_val=0],max | align to power of 2
.balign | b,[pad_val=0] | byte align
.zero | integer | zero bytes
.variant_cc | symbol_name | annotate the symbol with variant calling convention

### <a name=".option"></a> `.option`

#### `rvc` / `norvc`

启用/禁用以下代码区域的 C 扩展。

#### `pic` / `nopic`

将代码模型设置为 PIC（位置无关代码）或非 PIC。这会影响`la`伪指令的扩展，参考[标准 RISC-V 伪指令列表](#pseudoinstructions)。

#### `relax` / `norelax`

为以下代码区域启用/禁用链接器松弛。

注意： `.option relax`的代码区域将生成`R_RISCV_RELAX` / `R_RISCV_ALIGN`即使链接器不支持放松。如果要禁用特定代码区域的链接器松弛，建议使用`.option norelax`和`.option push` / `.option pop` 。

注意：禁用特定代码区域的链接器松弛的推荐方法是使用`.option push` 、 `.option norelax`和`.option pop` ，如果用户已经禁用了链接器松弛，这可以防止意外启用的链接器松弛。

#### `push` / `pop`

将当前选项推入/弹出选项栈。

## 汇编器重定位函数

下表列出了汇编器重定位扩展：

汇编符号 | 描述 | 指令/宏
:-- | :-- | :--
%hi(symbol) | Absolute (HI20) | lui
%lo(symbol) | Absolute (LO12) | load, store, add
%pcrel_hi(symbol) | PC-relative (HI20) | auipc
%pcrel_lo(label) | PC-relative (LO12) | load, store, add
%tprel_hi(symbol) | TLS LE "Local Exec" | lui
%tprel_lo(symbol) | TLS LE "Local Exec" | load, store, add
%tprel_add(symbol) | TLS LE "Local Exec" | add
%tls_ie_pcrel_hi(symbol) * | TLS IE "Initial Exec" (HI20) | auipc
%tls_gd_pcrel_hi(symbol) * | TLS GD "Global Dynamic" (HI20) | auipc
%got_pcrel_hi(symbol) * | GOT PC-relative (HI20) | auipc

* 这些重用 %pcrel_lo(label) 的低半部分

## 标签

文本标签用作分支、无条件跳转目标和符号偏移。文本标签被添加到编译模块的符号表中。

```assembly
loop:
        j loop
```

数字标签用于本地引用。对本地标签的引用后缀为“f”表示前向引用或后缀为“b”表示后向引用。

```assembly
1:
        j 1b
```

## 绝对寻址

以下示例显示如何加载绝对地址：

```assembly
	lui	a0, %hi(msg + 1)
	addi	a0, a0, %lo(msg + 1)
```

如`objdump`所见，它生成以下汇编器输出和重定位：

```assembly
0000000000000000 <.text>:
   0:	00000537          	lui	a0,0x0
			0: R_RISCV_HI20	msg+0x1
   4:	00150513          	addi	a0,a0,1 # 0x1
			4: R_RISCV_LO12_I	msg+0x1
```

## 相对寻址

以下示例显示如何加载相对 PC 的地址：

```assembly
1:
	auipc	a0, %pcrel_hi(msg + 1)
	addi	a0, a0, %pcrel_lo(1b)
```

如`objdump`所见，它生成以下汇编器输出和重定位：

```assembly
0000000000000000 <.text>:
   0:	00000517          	auipc	a0,0x0
			0: R_RISCV_PCREL_HI20	msg+0x1
   4:	00050513          	mv	a0,a0
			4: R_RISCV_PCREL_LO12_I	.L1
```

## GOT-间接寻址

以下示例显示如何从 GOT 加载地址：

```assembly
1:
	auipc	a0, %got_pcrel_hi(msg + 1)
	ld	a0, %pcrel_lo(1b)(a0)
```

如`objdump`所见，它生成以下汇编器输出和重定位：

```assembly
0000000000000000 <.text>:
   0:	00000517          	auipc	a0,0x0
			0: R_RISCV_GOT_HI20	msg+0x1
   4:	00050513          	mv	a0,a0
			4: R_RISCV_PCREL_LO12_I	.L1
```

## 加载立即数

以下示例显示了用于加载立即值的`li`伪指令：

```assembly
	.equ	CONSTANT, 0xdeadbeef

	li	a0, CONSTANT
```

对于 RV32I，生成以下汇编程序输出，如`objdump`所示：

```assembly
00000000 <.text>:
   0:	deadc537          	lui	a0,0xdeadc
   4:	eef50513          	addi	a0,a0,-273 # deadbeef <CONSTANT+0x0>
```

## 加载立即数的上半部分

`lui`的直接参数是区间 [0x0, 0xfffff] 中的整数。它的压缩形式`c.lui`只接受子区间 [0x1, 0x1f] 和 [0xfffe0, 0xfffff] 中的那些。

## 加载地址

以下示例显示了`la`伪指令，该指令用于根据代码是否被汇编为 PIC，使用正确的顺序加载符号地址：

```assembly
	la	a0, msg + 1
```

对于非 PIC，这是下面记录的`lla`伪指令的别名。

对于 PIC，这是下面记录的`lga`伪指令的别名。

`la`伪指令是在汇编中获取变量地址的首选方法，除非需要显式控制 PC 相对或 GOT 间接寻址。

## 加载局部地址

以下示例显示了用于加载局部符号地址的`lla`伪指令：

```assembly
	lla	a0, msg + 1
```

如`objdump`所示，这会生成以下指令和重定位：

```assembly
0000000000000000 <.text>:
   0:	00000517          	auipc	a0,0x0
			0: R_RISCV_PCREL_HI20	msg+0x1
   4:	00050513          	mv	a0,a0
			4: R_RISCV_PCREL_LO12_I	.L0
```

## 加载全局地址

以下示例显示了用于加载全局符号地址的`lga`伪指令：

```assembly
	lga	a0, msg + 1
```

这会生成`objdump`所示的以下指令和重定位（RV32 将使用`lw`而不是`ld` ）：

```assembly
0000000000000000 <.text>:
   0:	00000517          	auipc	a0,0x0
			0: R_RISCV_GOT_HI20	msg+0x1
   4:	00053503          	ld	a0,0(a0) # 0 <.text>
			4: R_RISCV_PCREL_LO12_I	.L0
```

## 全局加载和存储

以下伪指令可用于从全局对象加载和存储：

- `l{b|h|w|d} <rd>, <symbol>` : 从全局加载字节、半字、字或双字[^1]
- `s{b|h|w|d} <rd>, <symbol>, <rt>` : 将字节、半字、字或双字存储到全局[^2]
- `fl{h|w|d|q} <rd>, <symbol>, <rt>` : 从全局加载半精度、单精度、双精度或四精度 [^2]
- `fs{h|w|d|q} <rd>, <symbol>, <rt>` : 将半精度、单精度、双精度或四精度存储到全局[^2]

[^1]：第一个操作数隐式用作暂存寄存器。 [^2]：最后一个操作数指定要使用的暂存寄存器。

以下示例显示了如何使用这些伪指令：

```assembly
	lw	a0, var1
	fld	fa0, var2, t0
	sw	a0, var3, t0
	fsd	fa0, var4, t0
```

如`objdump`所示，它生成以下汇编器输出和重定位：

```
0000000000000000 <.text>:
   0:	00000517          	auipc	a0,0x0
			0: R_RISCV_PCREL_HI20	var1
   4:	00052503          	lw	a0,0(a0) # 0 <.text>
			4: R_RISCV_PCREL_LO12_I	.L0
   8:	00000297          	auipc	t0,0x0
			8: R_RISCV_PCREL_HI20	var2
   c:	0002b507          	fld	fa0,0(t0) # 8 <.text+0x8>
			c: R_RISCV_PCREL_LO12_I	.L0
  10:	00000297          	auipc	t0,0x0
			10: R_RISCV_PCREL_HI20	var3
  14:	00a2a023          	sw	a0,0(t0) # 10 <.text+0x10>
			14: R_RISCV_PCREL_LO12_S	.L0
  18:	00000297          	auipc	t0,0x0
			18: R_RISCV_PCREL_HI20	var4
  1c:	00a2b027          	fsd	fa0,0(t0) # 18 <.text+0x18>
			1c: R_RISCV_PCREL_LO12_S	.L0
```

## 常数

以下示例显示了使用`%hi`和`%lo`汇编函数加载常量。

```assembly
	.equ	UART_BASE, 0x40003080

	lui	a0, %hi(UART_BASE)
	addi	a0, a0, %lo(UART_BASE)
```

如`objdump`所示，它生成以下汇编程序输出：

```assembly
0000000000000000 <.text>:
   0:	40003537          	lui	a0,0x40003
   4:	08050513          	addi	a0,a0,128 # 40003080 <UART_BASE>
```

## 函数调用

以下伪指令可用于调用远离当前位置的子程序：

- `call <symbol>` : 调用子程序[^1]
- `call <rd>, <symbol>` : 调用子程序[^2]
- `tail <symbol>` : tail 调用子程序[^3]
- `jump <symbol>, <rt>` : 跳转到远处程序[^4]

[^1]: `ra`隐式用于保存返回地址。 [^2]：类似于`call <symbol>` ，但`<rd>`用于保存返回地址。 [^3]: `t1`隐式用作暂存寄存器。 [^4]：类似于`tail <symbol>` ，但`<rt>`被用作暂存寄存器。

以下示例显示了如何使用这些伪指令：

```assembly
	call	func1
	tail	func2
	jump	func3, t0
```

如`objdump`所示，它生成以下汇编器输出和重定位：

```
0000000000000000 <.text>:
   0:	00000097          	auipc	ra,0x0
			0: R_RISCV_CALL	func1
   4:	000080e7          	jalr	ra # 0x0
   8:	00000317          	auipc	t1,0x0
			8: R_RISCV_CALL	func2
   c:	00030067          	jr	t1 # 0x8
  10:	00000297          	auipc	t0,0x0
			10: R_RISCV_CALL	func3
  14:	00028067          	jr	t0 # 0x10
```

## 浮点舍入模式

对于带有舍入模式字段的浮点指令，可以通过添加一个附加操作数来指定舍入模式。例如，四舍五入的`fcvt.ws`可以写为`fcvt.ws a0, fa0, rtz` 。如果未指定，将使用默认的`dyn`舍入模式。

支持的舍入方式如下（必须用小写指定）：

- `rne` ：四舍五入到最近，与偶数相关
- `rtz` ：向零舍入
- `rdn` ：向下舍入
- `rup` ：四舍五入
- `rmm` ：四舍五入到最近，与最大值相关
- `dyn` ：动态舍入模式（使用`fcsr`寄存器的`frm`字段中指定的舍入模式）

## 控制和状态寄存器

以下代码示例显示了如何启用定时器中断、设置和等待定时器中断发生：

```assembly
.equ RTC_BASE,      0x40000000
.equ TIMER_BASE,    0x40004000

# setup machine trap vector
1:      auipc   t0, %pcrel_hi(mtvec)        # load mtvec(hi)
        addi    t0, t0, %pcrel_lo(1b)       # load mtvec(lo)
        csrrw   zero, mtvec, t0

# set mstatus.MIE=1 (enable M mode interrupt)
        li      t0, 8
        csrrs   zero, mstatus, t0

# set mie.MTIE=1 (enable M mode timer interrupts)
        li      t0, 128
        csrrs   zero, mie, t0

# read from mtime
        li      a0, RTC_BASE
        ld      a1, 0(a0)

# write to mtimecmp
        li      a0, TIMER_BASE
        li      t0, 1000000000
        add     a1, a1, t0
        sd      a1, 0(a0)

# loop
loop:
        wfi
        j loop

# break on interrupt
mtvec:
        csrrc  t0, mcause, zero
        bgez t0, fail       # interrupt causes are less than zero
        slli t0, t0, 1      # shift off high bit
        srli t0, t0, 1
        li t1, 7            # check this is an m_timer interrupt
        bne t0, t1, fail
        j pass

pass:
        la a0, pass_msg
        jal puts
        j shutdown

fail:
        la a0, fail_msg
        jal puts
        j shutdown

.section .rodata

pass_msg:
        .string "PASS\n"

fail_msg:
        .string "FAIL\n"
```

## <a name="pseudoinstructions"></a>标准 RISC-V 伪指令列表

伪指令 | 基本指令 | 意义 | 注释
:-- | :-- | :-- | :--
la rd, symbol | auipc rd, symbol[31:12]; addi rd, rd, symbol[11:0] | Load address | With `.option nopic` (Default)
la rd, symbol | auipc rd, symbol@GOT[31:12]; l{w|d} rd, symbol@GOT[11:0](rd) | Load address | With `.option pic`
lla rd, symbol | auipc rd, symbol[31:12]; addi rd, rd, symbol[11:0] | Load local address |
lga rd, symbol | auipc rd, symbol@GOT[31:12]; l{w|d} rd, symbol@GOT[11:0](rd) | Load global address |
l{b|h|w|d} rd, symbol | auipc rd, symbol[31:12]; l{b|h|w|d} rd, symbol[11:0](rd) | Load global |
s{b|h|w|d} rd, symbol, rt | auipc rt, symbol[31:12]; s{b|h|w|d} rd, symbol[11:0](rt) | Store global |
fl{w|d} rd, symbol, rt | auipc rt, symbol[31:12]; fl{w|d} rd, symbol[11:0](rt) | Floating-point load global |
fs{w|d} rd, symbol, rt | auipc rt, symbol[31:12]; fs{w|d} rd, symbol[11:0](rt) | Floating-point store global |
nop | addi x0, x0, 0 | No operation |
li rd, immediate | *Myriad sequences* | Load immediate |
mv rd, rs | addi rd, rs, 0 | Copy register |
not rd, rs | xori rd, rs, -1 | Ones’ complement |
neg rd, rs | sub rd, x0, rs | Two’s complement |
negw rd, rs | subw rd, x0, rs | Two’s complement word |
sext.b rd, rs | slli rd, rs, XLEN - 8; srai rd, rd, XLEN - 8 | Sign extend byte | It will expand to another instruction sequence when B extension is available*[1]
sext.h rd, rs | slli rd, rs, XLEN - 16; srai rd, rd, XLEN - 16 | Sign extend half word | It will expand to another instruction sequence when B extension is available*[1]
sext.w rd, rs | addiw rd, rs, 0 | Sign extend word |
zext.b rd, rs | andi rd, rs, 255 | Zero extend byte |
zext.h rd, rs | slli rd, rs, XLEN - 16; srli rd, rd, XLEN - 16 | Zero extend half word | It will expand to another instruction sequence when B extension is available*[1]
zext.w rd, rs | slli rd, rs, XLEN - 32; srli rd, rd, XLEN - 32 | Zero extend word | It will expand to another instruction sequence when B extension is available*[1]
seqz rd, rs | sltiu rd, rs, 1 | Set if = zero |
snez rd, rs | sltu rd, x0, rs | Set if != zero |
sltz rd, rs | slt rd, rs, x0 | Set if &lt; zero |
sgtz rd, rs | slt rd, x0, rs | Set if &gt; zero |
fmv.s rd, rs | fsgnj.s rd, rs, rs | Copy single-precision register |
fabs.s rd, rs | fsgnjx.s rd, rs, rs | Single-precision absolute value |
fneg.s rd, rs | fsgnjn.s rd, rs, rs | Single-precision negate |
fmv.d rd, rs | fsgnj.d rd, rs, rs | Copy double-precision register |
fabs.d rd, rs | fsgnjx.d rd, rs, rs | Double-precision absolute value |
fneg.d rd, rs | fsgnjn.d rd, rs, rs | Double-precision negate |
beqz rs, offset | beq rs, x0, offset | Branch if = zero |
bnez rs, offset | bne rs, x0, offset | Branch if != zero |
blez rs, offset | bge x0, rs, offset | Branch if ≤ zero |
bgez rs, offset | bge rs, x0, offset | Branch if ≥ zero |
bltz rs, offset | blt rs, x0, offset | Branch if &lt; zero |
bgtz rs, offset | blt x0, rs, offset | Branch if &gt; zero |
bgt rs, rt, offset | blt rt, rs, offset | Branch if &gt; |
ble rs, rt, offset | bge rt, rs, offset | Branch if ≤ |
bgtu rs, rt, offset | bltu rt, rs, offset | Branch if &gt;, unsigned |
bleu rs, rt, offset | bgeu rt, rs, offset | Branch if ≤, unsigned |
j offset | jal x0, offset | Jump |
jal offset | jal x1, offset | Jump and link |
jr rs | jalr x0, rs, 0 | Jump register |
jalr rs | jalr x1, rs, 0 | Jump and link register |
ret | jalr x0, x1, 0 | Return from subroutine |
call offset | auipc x6, offset[31:12]; jalr x1, x6, offset[11:0] | Call far-away subroutine |
tail offset | auipc x6, offset[31:12]; jalr x0, x6, offset[11:0] | Tail call far-away subroutine |
fence | fence iorw, iorw | Fence on all memory and I/O |

- [1] 当 B 扩展存在时，我们没有指定代码序列，因为 B 扩展仍然没有被批准或冻结。一旦冻结，我们将指定扩展顺序。

## 访问控制和状态寄存器的伪指令

伪指令 | 基本指令 | 意义
:-- | :-- | :--
rdinstret[h] rd | csrrs rd, instret[h], x0 | Read instructions-retired counter
rdcycle[h] rd | csrrs rd, cycle[h], x0 | 读取周期计数器
rdtime[h] rd | csrrs rd, time[h], x0 | 读取实时时钟
csrr rd, csr | csrrs rd, csr, x0 | Read CSR
csrw csr, rs | csrrw x0, csr, rs | Write CSR
csrs csr, rs | csrrs x0, csr, rs | 在 CSR 中设置位
csrc csr, rs | csrrc x0, csr, rs | 清除 CSR 中的位
csrwi csr, imm | csrrwi x0, csr, imm | Write CSR, immediate
csrsi csr, imm | csrrsi x0, csr, imm | Set bits in CSR, immediate
csrci csr, imm | csrrci x0, csr, imm | Clear bits in CSR, immediate
frcsr rd | csrrs rd, fcsr, x0 | 读取 FP 控制/状态寄存器
fscsr rd, rs | csrrw rd, fcsr, rs | 交换 FP 控制/状态寄存器
fscsr | csrrw x0, fcsr, rs | 写FP控制/状态寄存器
FRRMRD | csrrs rd，frm，x0 | 读取 FP 舍入模式
fsrm rd, rs | csrrw rd、frm、rs | 交换 FP 舍入模式
fsrm rs | csrrw x0, frm, rs | 写FP舍入模式
fsrmi rd, imm | csrrwi rd、frm、imm | Swap FP rounding mode, immediate
fsrmi | csrrwi x0, frm, imm | Write FP rounding mode, immediate
frflags rd | csrrs rd, fflags, x0 | 读取 FP 异常标志
fsflags rd, rs | csrrw rd, fflags, rs | 交换 FP 异常标志
fsflags rs | csrrw x0, fflags, rs | 写入 FP 异常标志
fsflagsi rd, imm | csrrwi rd, fflags, imm | Swap FP exception flags, immediate
fsflagsi imm | csrrwi x0, fflags, imm | Write FP exception flags, immediate


