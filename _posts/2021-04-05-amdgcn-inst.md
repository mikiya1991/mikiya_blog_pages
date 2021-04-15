---
layout: markdown
title: "AMD MI100 Instruction Set 详解"
comments: true
tags: "MI100 ISA" VALU
---

本文是对AMD MI100 ISA的中比较迷惑地方的详细解释。需要首先大概了解下AMD 指令集分类每个分类作用，等等。请先参考指令的ISA文档。

## 1 Vector ALU ##

![vop指令](/assets/vops.png)

大部分valu指令同时支持两种格式：vop3 64bits格式有最全的功能; 另外一种32bits的格式，能力有限。

推荐尽量使用非64bits的格式。

> 例如： v_add_u32指令有两种编码格式vop2编码和vop3编码。
vop2编码的是v_add_u32_e32，vop3编码的是v_add_u32指令。v_add_u32_e32对操作数有限制，如src1操作数只能是vgpr不能是constant或sgpr，但是其指令较短可以后面跟一个literal constant(src0 = literal_contant标志)。vop3则对src0， src1没什么限制（可以是sgpr，vgpr，constant）。

vop2、vop1、vopc 都是32bits。指令32bits编码后可以带一个32bits, litral constant。

vop2是2个inputs， 并且vector dst（vdst目的是vgpr）。带carry-out的指令（例如v_addc）会implicit写vcc。

vopc是比较指令。

![vop3指令](/assets/vop3.png)

VOP3 是有3个输入参数和input modifiers、 output modifiers。有两种类型的vop3： 一种以scalar为dst是vop3b类型，其他为vop3a类型。

vop3p 是用来“packed math”的，同时作用在输入pair上，例如vgpr 32bits会看作两个16bits数据。


## 1.1 vop operators ##

vop指令根据不同的指令编码类型会对操作数有不同的限制。

输入operator：

- 最多可读一个sgpr， 但是可以读入sgpr值可以多用
- 最多能用一个literal constant，但是使用constant时候不能使用sgpr和M0
- LDS只能用在SRC0，而且必须在所有thread中broadcast。
- ADDC、SUBB、CNDMASK implicit读vcc所以不能使用sgpr。(addc_co 不能使用sgpr！！add_co可以)
- vop3形式指令可以使用abs、neg field处理在input上。

输出operator：

- 一般VALU只能写入结果到VDST中vgpr，只有EXEC-mask=1时候才写结果。
- VCMPX写结果到sgpr（or vcc）并写exec mask。
- Inst输出carry-out的，vop2写入vcc，vop3 写入sgpr-pair。

![operator-1](/assets/operator.png)
![operator-2](/assets/operator2.png)


**上图的理解非常重要：**

根据指令编码不同，不同操作数会写为VSRC、VDST、SSRC、SRC。一般VSRC仅代表vector gpr， SSRC仅表示scalar gpr， SRC则是通用的既可是sgpr又可是vgpr。

- 9bits的SRC9 可以使用所有的code值。即SRC9可即可表示sgpr又可以是vgpr。
- SSRC8 可以用到0,256表示sgpr。
- VSRC8，VDST8 只能表示vgpr[0,255]。
- SSRC7 只用0~127的scalar src。

### **例如上面的vop2的指令结构图**

- vop2只能是vgpr <- （src0）(9bits可表示任意值:sgpr/vgpr/constant/硬件寄存器) + （src1）vgpr
- vop1： vgpr <- （src0）任意值
- vopc： vcc <- src0 任意 + src1 vgpr
- vop3a： vgpr <- src0，1，2 任意值
- vop3b： sgpr（0-127）, vgpr <- src0，1，2 任意值


## 2 Scalar ALU OP
![sop](/assets/sop.png)

同理看sop的指令编排：

SOP1：
- 单操作数
- src0 任意sgpr
- dst 只能是0-127的gpr，不能特殊sgpr

SOP2:
- 双操作数
- src0， src1 任意sgpr
- sdst7 0-127的sgpr

SOPK： 输入是立即数
SOPC:  src0 src1 都是全功能的sgpr

另外：
- SALU 不能使用VGPR。
- SALU 不能使用LDS。
- SALU可以带32bits的litral constant。
- SALU如果使用64bits的sgpr对，必须偶对齐。


指令类型了解后，具体指令看第12章。


## 2 Instruction Sample routines

举例如下指令：
- v_add_u32_e32 v41, 0x400, v41
- v_add_u32_e32 v13, v4, v2
    - v_add_u32_e32 是vop2
    - 立即数只能放在操作数2

- v_add_co_u32_e32 v1, vcc, s0, v1
- v_addc_co_u32_e32 v1, vcc, v1, v2, vcc
    - 带co的inst, 要将vcc写下，做第二个output
    - addc_co 最后要有vcc，5参
    - vop2 指令最多读一个sgpr, 只能在src0。
    - e32 表示是vopc，vop2，vop1的非vop3版本。vop3版本有多的控制位，但是vop3版本后面不能带literal constant。

- s_waitcnt lgmkcnt(3)
    - 意思是等到前面同类型读指令，还有3个未完成的时候



SGPR 64bits加法  `s[0:1] + s[2:3]`
```asm
s_add_u32 s0, s2, s0
s_addc_u32 s1, s3, s1
```
VGPR 64bits加 `v[0:1] + v[2:3]`
```asm
v_add_co_u32_e32 v0, vcc, v2, v0
v_addc_co_u32_e32 v1, vcc, v3, v1, vcc
```
某些valu指令需要指令操作是vgpr，而数据可能是constant或者sgpr
```asm
s_mov_b32_e32 v0, s0
s_mov_b32_e32 v1, s1
```

valu 左右常数位移，因为constant操作数必须是src0，所以常用方式为
```asm
v_lshlrev_u32_e32 v1, 2, v1
v_ashrrev_i32_e32 v2, 2, v2
```
32位带符号数左移为64位
```asm
v_ashrrev_i32_e32 v1,31,v0 //arthmic shift，相当于将i32的符号位extend给v1, 并置v1为0
v_lshlrev_b64 v[3:4], 2, v[0:1]
```
