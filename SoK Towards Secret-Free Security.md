[TOC]

## SoK: Towards Secret-Free Security

作者：Ulrich Rührmair



### Abstract

- second generation of physical primitives 
  - Complex PUFs
  - SIMPLs/PPUFs
  - related techniques enable completely *“secret-free” systems*
- 攻击者可以可以知晓任何一个bit或atom，可以了解从硬件中获取的任何信息，而不能破坏其安全性

### 1 Introduction

- Kerckhoffs‘ principle：加密安全依赖于密钥安全
- challenge：有效保护硬件中的密钥
- physical unclonable functions(PUF)：用复杂的物理结构代替密钥，其challenge-response行为依赖于其独特的manufacturing variations
  - 防止在硬件（易受攻击）中存储密钥
  - standard silicon PUFs仍会泄漏一些秘密信息：
    1. the power-up states of SRAM PUFs
    2. the individual runtime delays in Arbiter PUFs
- contributions：
  - 是第一份综述，提出第一个(半)正式定义，设计硬件秘密信息的第一个分类法
  - 引入了Complex PUF
  - 提出了针对无秘密原语的半正式框架，利用了硬件*internal state*的概念
  - 重新构建和分类PUF和物理加密，提出一个新的目标：完全避免硬件中的任何秘密
- 文章结构：
  - 第2节提出*secrets*的正式定义、分类，*secret-free*硬件的定义
  - 第3节分析了典型的弱和强PUF方案中的*secrets* 
  - 第4节和第5节详细介绍了两种中央*secret-free*安全原语：Complex PUFs 和SIMPLs/PPUFs
  - 第6节描述了*secret-free*安全技术的可能扩展
  - 第15页的表1概述了主要结果和分类

### 2 Defining Secret-free Security

#### 2.1 Hardware Secrets

- **==Definition 1==**: "standard" security scheme $S$ 

  - $S$ 由有限的硬件系统 $\Eta_1,...,\Eta_k$ 实现
  - $S$ 由初始设置阶段 $R_0$，后续运行阶段 $R_1,...R_n$ 组成
  - $S$ 及 $R_j$ 都具有与攻击者 $\Alpha_S$ 有关的安全特征

- **==Definition 2==**:  (hardware) secret with respect to $S$ (A finite number or bitstring ***sec***)

  - *sec* 在 $S$ 的某一硬件系统 $\Eta_i$ 某一运行阶段 $R_j$ (包括 $R_0$ )以某种形式物理呈现

    > 包括但不限于：1. （非）易失存储器的内容；2. 电路组件的物理特性；3. 瞬态信号的物理特征（信号时间、形状、强度）；4. 硬件内部物理状态、结构、配置

  - 在 $R_j$ 阶段结束时暴露于 $\Alpha_S$ 的*sec*，可以使 $\Alpha_S$ 去破坏 $R_1,...R_n$ 中的某些安全特征

#### 2.2 A Taxonomy of Secrets in Hardware

- **==Definition 3==**: a taxonomy of *sec* 

  1. Permanent digital secret：出现在所有 $R_1,...R_n$，永久存储在 $\Eta_i$ 的数字存储器

  2. Non-volatile digital secret：位于 $\Eta_i$ 的非易失数字存储器至少一次

  3. Volatile digital secret：位于 $\Eta_i$ 的易失数字存储器至少一次

  4. Transient digital secret：呈现为 $\Eta_i$ 中某些瞬态数字信号的数字值

  5. **Digital secret**：易失、非易失、瞬时的数字秘密信息

     **physical secret**：易失、非易失的非数字秘密信息

  6. Permanent physical secret：出现在所有 $R_1,...R_n$，出现在 $\Eta_i$ 中至少一次的物理秘密信息

  7. Non-volatile physical secret：出现在 $\Eta_i$ 中至少一次的物理秘密信息，断电断能仍存在

  8. Volatile physical secret：出现在 $\Eta_i$ 中至少一次的物理秘密信息，断电断能则丢失

     > 数字还是物理？是否永久？易失、非易失还是瞬态（针对数字）的？
     >
     > - 数字比物理更易受攻击
     > - 永久更容易被攻击
     > - 非易失性更容易受到攻击，瞬态最不容易被攻击者获得
     >
     > 1到8：最脆弱的机密到最具攻击弹性的机密

#### 2.3 Secret-Free Hardware and Its Advantages

- ==**Definition 4**==: Secret-Free Hardware
  - A single hardware system $H_i$ is called*secret-free* if it contains no secrets
- 优势：见Abstract

### 3 Secrets in Standard PUFs

- SRAM PUF：由k个SRAM单元组成的（标准）阵列

  - *PUF-challenge*：the power-up command，整个PUF只有一个challenge
  - *PUF-response*：the concatenated string of the *k* power-up values of the *k* SRAM cells
  - 有时被称作 Weak PUFs 或者 Physically Obfuscated Keys (POKs)

- XOR Arbiter PUF：由k个并行的Arbiter PUF组成

  - *PUF-challenge* $C_i$：长度为m的比特串（$2^m$种可能）

    > 将其分发至k个并行仲裁器PUF，产生k个子响应 $R^1_i,...,R^k_i$ ，子响应暂时保存在每个Aribiter PUF的非易失性仲裁器元件/锁存器中

  - *PUF-reponse*：$R_i=R^1_i\bigoplus...\bigoplus R^k_i$，单个比特

  - 称为 Strong PUFs 

#### 3.1 SRAM PUF

- **==Scheme 5==**：Identification with SRAM PUFs

  - **Set-up Phase**($R_0$)：
    1. prover的硬件 $\Eta_P$ 测量k个SRAM单元的通电状态
    2. $\Eta_P$ 从通电状态中获得密钥$K$，用于纠错的辅助数据 $HD$ ($HD$永久存储在非易失数字存储器中)
    3. $\Eta_P$ 向verifier的硬件 $\Eta_V$ 提供$K$ ($K$永久存储在非易失性数字存储器中)
    4. $\Eta_P$ 向外部传递密钥的功能被禁用

  - **Execution/Identification Phase** ($R_i$ , 1 ≤ *i* ≤ *n*)：
    1. verifier的硬件 $\Eta_V$ 远程发送一个新随机数 $N_i$ 给prover的硬件 $\Eta_P$ 
    2. $\Eta_P$ 触发SRAM单元的通电操作，并使用 $HD$ 从通电状态中获得密钥 $K$ 
    3. $\Eta_P$ 使用伪随机函数 $PRF$ 计算值$PRF_K(N_i)$，并将其远程发送给verifier的硬件，此后，密钥 $K$ 和SRAM单元的通电状态被擦除
    4. verifier验证器的硬件使用 $K$ 检查接收值的正确性，正确则接受标识

- secrets in Scheme 5
  - hardware $\Eta_P$：
    - volatile digital secret：k个SRAM单元的通电状态
    - manifold transient digital secrets： 
      - $R_0$ Step 3：$\Eta_P$ 到 $\Eta_V$的 $K$ 
      - $R_i$ Step 3：$\Eta_P$ 到 $PRF_K(N_i)$ 计算电路的 $K$
      - $R_0、R_i$ Step 2：将SRAM单元的通电状态传达给 $\Eta_P$ 的纠错/密钥推导机制的数字信号 
    - permanent physical secrets、non-volatile physical secrets：physical manufacturing variations
  - hardware $\Eta_V$：
    -  permanent digital secret、non-volatile digital secret：$K$ 
    - transient digital secrets：
      - $R_0$ Step 3：把 $K$ 从与 $\Eta_P$ 的通信接口转移到 $\Eta_V$ 的非易失存储器产生的瞬态数字信号
      - $R_i$ Step 4： $K$ 从存储位置转移到校验 $PRF_K(N_i)$ 模块

#### 3.2 XOR Arbiter PUF

略

### 4 Secret-free Security by Complex PUFs
#### 4.1 Definition of Complex PUFs

- **==Definition 7==**: $(\epsilon,t_{Pred})- Secure\ Complex\ PUF$ 

  - $P$ 是一个内部状态为 $IntSta(P)$ 的 $PUF$ 

    > $IntSta(P)$: including any manufacturing variations and random internal parameters, such as electrical, optical, or magnetic features, positions and states of atoms, etc., of P

  - 攻击者 $\Alpha$ 最高以概率 $\epsilon$ 去赢得game：$FastPredGame(P,IntSta(P),t_{Pred})$ 

    > $\Alpha$ 可以在充足的时间（一年）里准备，包括对 $P$ 进行物理测量（包括确定CRPs），进行计算（包括机器学习攻击或预计算），制造物理系统（包括特殊模拟设备和P的物理克隆）；给定*challeng* $C_i$ 需在 $t_{Pred}$ 内确定*response prediction* $\hat{R_i}$ 
    >
    > $\Alpha$ 赢，如果 $\hat{R_i}=R_i$ 

#### 4.2 Implementation of Complex PUFs

> optical PUF of Pappu et al.——(13.3%, 10s)-Complex PUFs

- $t_{Pred}$：至少 $10^{26}$ 次计算操作，设为10s

- $\epsilon$ ：2400-bit的密钥，随机猜测概率概率为 $2^{-2400}$，不可行。最好的方法：
  - 收集尽可能多的CRPs，希望给的challenge $C_i$ 属于我们已知的CRPs
  - 不相关和独立的CRPs数量为 $2.37·10^{10}$，可以假设攻击者每秒可以测量 $10^2$ 个CRPs，一年能测所有的13.3%。设 $\epsilon$ 为 13.3%

#### 4.3 An Example Scheme and Its Properties

- **==Scheme 8==**：Identification with $(\epsilon,t_{Pred})- Secure\ Complex\ PUF$ 

  - **Set-up Phase**($R_0$)：
    1. verifier的硬件 $\Eta_V$ 选取 $\lambda·n$ 个challenge $C_1,...,C_{\lambda·n}$ 
    2. $\Eta_V$ 将这些challeng用 $(\epsilon,t_{Pred})-Complex\ optical\ PUF$ 获取相关的response $R_1,...,R_{\lambda·n}$ 
    3. CRP-List $(C_1,R_1),...,(C_{\lambda·n},R_{\lambda·n})$ 永久存储在 $\Eta_V$ 的非易失存储器中
  - **Execution Phase** ($R_i$ , 1 ≤ *i* ≤ *n*)：
    1. verifier的硬件 $\Eta_V$ 随机选取 $\lambda$ 个新CRPs $(C_1^i,R_1^i),...,(C_{\lambda}^i,R_{\lambda}^i)$ 
    2. $\Eta_V$ 发送 $C_1^i,...,C_{\lambda}^i$ 到prover的硬件 $\Eta_P$ 
    3. $\Eta_P$ 将这些challeng用 optical PUF 获取相关的response $\overline{R}_1^i,...,\overline{R}_{\lambda}^i$，并发送到 $\Eta_V$ 
    4. $\Eta_V$ 测量从发送 $C_1^i,...,C_{\lambda}^i$ 到收到 $\overline{R}_1^i,...,\overline{R}_{\lambda}^i$ 的时间 $t^*$ 
    5. 如果 $t^*\leq t_{Pred}$ 且 $\overline{R}_j^i=R_j^i\ j=1,...,\lambda$，$\Eta_V$ 接受 identification，否则不接受
    6. CRPs $(C_1^i,R_1^i),...,(C_{\lambda}^i,R_{\lambda}^i)$ 从CRP-List中移除

- Error-tolerance:  relaxing the decision rule of $\Eta_V$ 

- Scheme 8 工作安全当 $t_{Pred}\geq 2·t_{Lat}+\lambda ·t_{Meas}$  (1)

  > $t_{Lat}$: prover和verifier之间通信和处理的延迟，<=1s
  >
  > $t_{Meas}$: the prover’s time for measuring a response of the employed Complex PUF，0.1s
  >
  > $\lambda$ = 35
  >
  > Scheme 8 is secure, for these values

#### 4.4 Secrets in Scheme 8

- prover的硬件 $\Eta_P$ 
  - 在 Execution Phase 不包含任何secret
  - 在 Set-up Phase包含volatile physical secrets：physical and analog optical challenges $C_i$ (激光束位置和入射角) ？？？

- verifier的硬件 $\Eta_V$ 包含permanent digital secrets和non-volatile digital secrets：CRP-List
- Scheme 8实现了远程识别的执行阶段prover的secret-free

### 5 Secret-free Security by SIMPLs

#### 5.1 Definition of SIMPLs

SIMPLs("SIMulation Possible, but Laborious") / Public PUF(PPUF)

- **==Definition 9==**: SIMPL $S$ 

  - 随机且单独被构造，无序的，物理上不可复制
  - 匹配 $C_i$ 和 $S_i$ 
  - 拥有大量的challeng，ideally某些参数是指数的
  - 有公开的 $Des(S)$ 去描述 $S$ 的个体性，唯一无序性
  - 有公开的仿真算法 $SIM$，对给定的 $C_i$ 能得到对应的 $R_i$ ($R_i=SIM(C_i,Des(S))$)

  > 仿真比真实更慢
  >
  > 由于3D制造工艺的限制，$S$ 是unclonable即使 $Des(S)$ 已知

- **==Definition 10==**: $(\epsilon,t_{Pred})- Secure\ SIMPLs$ 

  - $S$ 是一个内部状态为 $IntSta(S)$ 、仿真算法为 $SIM$ 的 $SIMPL$ 

  - 攻击者 $\Alpha$ 最高以概率 $\epsilon$ 去赢得game：$FastPredGame(P,\Alpha,IntSta(S),Des(S),SIM,t_{Pred})$ 

    > $\Alpha$ 获得 $IntSta(S),Des(S),SIM$ 甚至可以物理上访问 $S$，可以在充足的时间（一年）里准备，包括对 $S$ 进行物理测量（包括确定CRPs），进行计算（包括机器学习攻击或预计算），制造物理系统（包括特殊模拟设备和P的物理克隆）；给定*challeng* $C_i$ 需在 $t_{Pred}$ 内确定*response prediction* $\hat{R_i}$ 
    >
    > $\Alpha$ 赢，如果 $\hat{R_i}=R_i$ 

    > 任何 $(\epsilon,t_{Pred})-SIMPLs$ 都是$(\epsilon,t_{Pred})- Complex\ PUFs$ 

#### 5.2 Implementation of SIMPLs

> 仿真复杂性不能简单的设为最大，应被调整为足够大去防止quick仿真，但仍足以进行实际中的response认证或者slow仿真

> Recommended starting points for further reading on SIMPL/PPUF implementations are [29, 47, 53, 54, 72]

#### 5.3 An Example Scheme and Its Properties

- **==Scheme 11==**：Identification with $(\epsilon,t_{Pred})-SIMPLs$ 

  - **Set-up Phase**($R_0$)：
    1. SIMPLs $S$，$SIM$ 产生
    2.  SIM 永久存储在 $\Eta_V$ 的非易失存储器中
  - **Execution Phase** ($R_i$ , 1 ≤ *i* ≤ *n*)：
    1. verifier的硬件 $\Eta_V$ 选取 $\lambda$ 个challenge $C_1^i,...,C_{\lambda}^i$ 
    2. $\Eta_V$ 发送 $C_1^i,...,C_{\lambda}^i$ 至prover的硬件 $\Eta_P$ 
    3. $\Eta_P$ 将这些challeng用 $SIMPL$ 获取相关的response $\overline{R}_1^i,...,\overline{R}_{\lambda}^i$，并发送至 $\Eta_V$ 
    4. $\Eta_V$ 测量从发送 $C_1^i,...,C_{\lambda}^i$ 到收到 $\overline{R}_1^i,...,\overline{R}_{\lambda}^i$ 的时间 $t^*$ 
    5. 如果 $t^*\leq t_{Pred}$ 且 $\overline{R}_j^i=R_j^i\ j=1,...,\lambda$，$\Eta_V$ 接受 identification，否则不接受

- Error-tolerance:  relaxing the decision rule of $\Eta_V$ in Step 5

- Scheme 8 工作安全当 $t_{Pred}\geq 2·t_{Lat}+\lambda ·t_{Meas}$  (2)

- verifier可能需要花较多的时间去仿真验证来自prover的response的正确性

  > Pre-simulating：不可行
  >
  > SIMPLs：从零仿开始真很难，但去证实很快，见5.2

#### 5.4 Secrets in Scheme 11

- prover的硬件 $\Eta_P$ 是 *secret-free* 
- verifier的硬件 $\Eta_V$ 是 *secret-free* 

### 6 Extension to other Secret-free Schemes

1. Secret-Free Message Authentication
   - protocols similar to Schemes 11

2. Secret-Free Tamper Detection
   - 一个对其CRPs结构完整敏感的PUF内置于硬件中，通过测算CRP和a similar protocol as Scheme 6，可以检查该硬件的完整性和未被篡改性
3. Secret-Free Sensing and Virtual Proofs
   - 假设Complex PUF或SIMPL/PPUF能够依赖于某些环境变量（如温度、湿度、压力等）的响应，其CRP可以远程prove这些变量的实际值
   - protocols similar to Schemes 8 and 11
   - 每个PUF-response是环境变量的函数
4. Secret-Free Unforgeable Item Tags/Labels
   - 可以将唯一、随机结构且不可复制的物理对象附加到对象
   - 这个独特的物理特性是制造商使用其密钥 $SK_{Man}$ 签名的数字签名
   - testing apparatuses
5. Secret-Free Digital Rights Management
   - 像CD/DVD这样的存储介质在亚数字模拟级别上具有独特的物理特性，即使它们存储的数字内容完全相同
6.  Secret-Free Encryption and Signatures

### 7 SUMMARY AND OUTLOOK





