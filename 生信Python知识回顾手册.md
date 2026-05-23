# 生信 Python 知识回顾手册

> 专为"学过但忘了"的生信人设计 —— 每个知识点都附带**逐行解释**和**为什么这么做**，帮你从"好像学过"到"真的会用"。

---

## 目录

1. [环境搭建速查](#1-环境搭建速查)
2. [Python 语法急救箱](#2-python-语法急救箱)
3. [生信核心数据结构](#3-生信核心数据结构)
4. [文件读写：生信格式大全](#4-文件读写生信格式大全)
5. [字符串与序列操作](#5-字符串与序列操作)
6. [NumPy：基因表达矩阵的基石](#6-numpy基因表达矩阵的基石)
7. [Pandas：生信数据的万能工具箱](#7-pandas生信数据的万能工具箱)
8. [数据可视化](#8-数据可视化)
9. [统计学与假设检验](#9-统计学与假设检验)
10. [Biopython 精要](#10-biopython-精要)
11. [机器学习入门：scikit-learn 在生信中的应用](#11-机器学习入门scikit-learn-在生信中的应用)
12. [单细胞分析：Scanpy 入门](#12-单细胞分析scanpy-入门)
13. [综合实战：三条完整分析流水线](#13-综合实战三条完整分析流水线)
14. [速查卡片合集](#14-速查卡片合集)

---

## 1. 环境搭建速查

### 1.1 什么是 Conda？为什么需要它？

想象你有两个生信项目：项目 A 需要 `numpy==1.20`，项目 B 需要 `numpy==1.24`。如果把它们都装在系统 Python 里，版本冲突会让你痛不欲生。

**Conda** 是一个"包管理器 + 环境管理器"。它让你为每个项目创建**独立的 Python 环境**，就像给每个项目分配了一台独立的电脑，互相完全隔离。

> 🎯 **类比**：Conda 环境就像一个个独立的实验台。你在台 A 上放试剂 A，台 B 上放试剂 B，它们不会互相污染。`conda activate bio` 就是"走到 bio 实验台前开始工作"。

### 1.2 名词解释

| 名词 | 解释 | 生信中的具体含义 |
|------|------|-----------------|
| **包 (package)** | 别人写好的代码库 | `numpy`、`pandas`、`scanpy` 都是包 |
| **环境 (environment)** | 一套独立的 Python + 包的组合 | `bio` 环境里装了生信相关的所有包 |
| **conda install** | 从 conda 仓库下载并安装包 | 类似手机上的"应用商店" |
| **pip install** | 从 PyPI 仓库下载并安装包 | 另一个应用商店，有些包只在 PyPI 有 |
| **conda-forge** | conda 的社区频道 | `-c conda-forge` 表示从这个频道下载 |
| **bioconda** | 专门收录生信软件的 conda 频道 | `-c bioconda` 表示下载生信专用包 |

### 1.3 搭建步骤

```powershell
# 第一步：下载安装 Miniconda（选一个版本即可）
# 官网：https://docs.conda.io/en/latest/miniconda.html
# 下载 Windows 64-bit 安装包，双击安装，一路默认即可

# 第二步：打开 PowerShell 或 Anaconda Prompt，创建环境
conda create -n bio python=3.10
# 解释：
# conda create  → 创建一个新环境
# -n bio        → 环境的名字叫 "bio"（你可以随便起名，比如 rnaseq_env）
# python=3.10   → 在这个环境里装 Python 3.10（生信生态兼容性最好的版本）
# 执行后它会列出即将安装的包，输入 y 确认

# 第三步：激活环境（进入这个"实验台"）
conda activate bio
# 激活后，你的终端提示符前面会出现 (bio)，说明你现在在 bio 环境里了
# 此后你做的所有 pip install 和 conda install 都只会影响 bio 环境

# 第四步：安装生信核心包
conda install -c conda-forge numpy pandas matplotlib seaborn jupyter scipy scikit-learn
# -c conda-forge → 从 conda-forge 频道下载（社区维护的包最全最稳定）
# 一次性安装 7 个包，conda 会自动处理它们之间的依赖关系

conda install -c bioconda biopython
# bioconda 频道专门收录生物信息学软件

pip install scanpy anndata xgboost umap-learn
# 有些包 conda 里没有或者版本旧，用 pip 装
# ⚠️ pip 和 conda 是两套系统，但都在当前环境里工作，不会冲突
```

### 1.4 日常使用

```powershell
# 查看所有环境
conda env list

# 激活环境（每个新终端窗口都需要做一次）
conda activate bio

# 查看当前环境装了哪些包
conda list

# 退出当前环境
conda deactivate

# 删除环境（如果环境坏了想重建）
conda remove -n bio --all
```

### 1.5 Jupyter Notebook / Lab

**Jupyter** 是"交互式编程笔记本"——你可以在里面逐段写代码、逐段运行、立刻看到结果，非常适合数据分析探索。

```python
# 在终端（确保已 conda activate bio）启动 Jupyter Lab
jupyter lab
# 浏览器会自动打开，你会看到一个文件管理器界面
# 点击 "Notebook" → "Python 3" 就可以开始写代码了

# ⚠️ 使用规则：
# - 数据分析、画图、探索数据 → 用 Jupyter Notebook
# - 写可复用的工具脚本、处理大批量文件 → 用 .py 文件
```

---

## 2. Python 语法急救箱

> 如果你只能复习一章，就复习这一章。这里覆盖了生信脚本中 90% 的基础语法。

### 2.1 变量与基本类型

#### 变量是什么？

变量就是一个**有名字的盒子**，你把数据放进去，之后可以通过名字取出数据。

```python
gene_name = "TP53"
# 意思是：创建一个盒子，贴上标签 "gene_name"，把字符串 "TP53" 放进去
# = 是"赋值"的意思（不是数学中的"等于"！）
# 右边的东西放进左边的盒子里
```

#### 四种基本"数据类型"

在 Python 中，每个数据都有"类型"。类型决定了你能对这个数据做什么操作。

```python
# 1. 字符串 str —— 用引号括起来的内容，代表文本
gene_name = "TP53"                    # 基因名、DNA序列、染色体号 都是字符串
dna_seq = "ATGGAGGAGCCGCAG"
chrom = "chr17"
# 本质：字符串是一个"字符序列"，可以按位置取、可以切片、可以拼接
# 标志：单引号 '' 或双引号 "" 都可以，但必须成对出现

# 2. 浮点数 float —— 带小数点的数字，代表连续数值
expression_level = 15.73              # 基因表达量
p_value = 0.0001                      # P 值
log2fc = -2.35                        # log2 Fold Change
# 为什么叫"浮点数"？因为小数点可以在数字中"浮动"
# 15.73  = 1.573 × 10^1 = 0.1573 × 10^2

# 3. 整数 int —— 不带小数点的数字，代表计数
exon_count = 12                       # 外显子数量
read_count = 4829153                  # 测序读段数
n_samples = 47                        # 样本数
# 生信中计数类数据都是 int：外显子数、突变数、读段数

# 4. 布尔值 bool —— 只有 True 和 False 两个值，代表"是/否"
is_oncogene = True                    # 是不是癌基因
is_significant = False                # 是否统计显著
# 布尔值来自条件判断的结果，如 x > 5 的结果就是 True 或 False
```

#### 类型转换 —— 为什么重要？

从文件读出来的数据**全是字符串**。你要做计算，就必须把字符串转成数字。

```python
# 场景：从 TSV 文件中读到了这些字符串
count_str = "42"           # 注意：有引号，这是字符串！
p_str = "0.0001"           # 这也是字符串，不是数字！

# 不转换就想做数学运算会报错：
# count_str + 1    →   TypeError: can only concatenate str (not "int") to str
# 这个报错的意思是：你不能把字符串和数字直接相加

# 正确的做法：先转换类型
count = int(count_str)          # int("42") → 42，现在可以做数学了
p_value = float("0.0001")       # float("0.0001") → 0.0001

# 生信中的典型用法
count = int(count_str) + 1              # 42 + 1 = 43
is_high = float(expr_str) > 10.0        # 把字符串转成 float 后才能比较

# 数字也可以转回字符串（写文件时需要）
result_str = str(42)                    # "42"
label = "Gene_" + str(5)               # "Gene_5"，+ 只能拼接字符串
```

### 2.2 条件判断

> 条件判断就是"如果……就……否则就……"。它让程序有了"做决定"的能力。

#### if / elif / else 的语法结构

```python
# 基本结构（冒号和缩进是 Python 的语法要求，不能省略！）
# if 条件:
#     条件成立时执行的代码    ← 注意：这里必须缩进（通常 4 个空格）
# elif 另一个条件:            ← elif = "else if" 的缩写，可以有很多个
#     另一个条件成立时执行的代码
# else:                       ← 以上条件都不成立时执行
#     以上都不成立时执行的代码

# ═══════════════════════════════════════════
# 生信场景 1：筛选差异表达基因
# ═══════════════════════════════════════════
# 假设已经算出了某个基因的 p_value 和 log2fc（差异倍数）
if p_value < 0.05 and abs(log2fc) > 1:
    print(f"{gene} is differentially expressed")

# 逐行解释：
# if                             → 如果……
# p_value < 0.05                 → P 值小于 0.05（统计显著）
# and                            → 并且（两个条件必须同时满足）
# abs(log2fc) > 1                → log2 差异倍数的绝对值大于 1（生物学显著）
# abs()                          → 取绝对值，abs(-2.5) = 2.5, abs(3.1) = 3.1
#                                   因为差异可能是上调（正值）或下调（负值）
#                                   我们关心的是"变化幅度"，不管方向
# :                              → 条件写完了，后面是满足条件时做的事
# print(f"...")                  → f 字符串：f"..." 里的 {变量名} 会被替换成变量的值
#                                   比如 gene="TP53"，输出就是 "TP53 is differentially expressed"

# ═══════════════════════════════════════════
# 生信场景 2：判断突变类型并打分
# ═══════════════════════════════════════════
if variant_type == "missense":
    impact = "moderate"
elif variant_type == "nonsense":
    impact = "high"
elif variant_type in ["synonymous", "intron"]:
    impact = "low"
else:
    impact = "unknown"

# 逐行解释：
# variant_type == "missense"
#   == 是"比较"（判断左右是否相等），结果是 True 或 False
#   区别于单个 =（赋值）
#   记忆口诀：一个 = 是"放入"，两个 == 是"是否相等"
#
# variant_type in ["synonymous", "intron"]
#   in 运算符：检查左边的值是否"在"右边的列表里
#   "synonymous" in ["synonymous", "intron"] → True
#   "missense"   in ["synonymous", "intron"] → False
#   这比写 variant_type == "synonymous" or variant_type == "intron" 简洁得多
#
# else:
#   当前面的 if 和所有 elif 都不成立时，执行 else 下的代码
#   这里表示"其他类型的突变，影响未知"
```

#### 在生信中最常用的运算符

```python
# ═══════ 比较运算符（结果是 True 或 False） ═══════
p_value < 0.05            # 小于
log2fc >= 1               # 大于等于
gene == "TP53"            # 等于（注意是双等号！）
gene != "TP53"            # 不等于
gene in gene_list         # 在列表中存在
base not in bases         # 在集合中不存在

# ═══════ 逻辑运算符（组合多个条件） ═══════
p < 0.05 and fc > 1       # 并且：两个条件都满足才为 True
p < 0.05 or fc > 1        # 或者：至少一个条件满足就为 True
not is_control             # 取反：如果 is_control 是 True，结果为 False

# ═══════ 生信实际例子 ═══════
# 检查序列是否合法
bases = set(["A", "T", "C", "G"])
if base not in bases:                 # 如果某个碱基不是 A/T/C/G
    print("Non-standard base found!") # 报告异常碱基

# 检查基因名是否在已知癌基因列表中
oncogenes = ["TP53", "EGFR", "KRAS", "MYC", "BRCA1"]
if gene_name in oncogenes:
    print(f"{gene_name} is a known oncogene")
```

#### 关于缩进（Indentation）

Python 用**缩进**（行首的空格）来表示代码块，这和其他语言用 `{}` 完全不同。

```python
# ✅ 正确：统一缩进
if condition:
    action1()
    action2()        # action1 和 action2 对齐，都属于 if 块

# ❌ 错误：缩进不一致
if condition:
    action1()
  action2()          # 缩进不对，Python 会报 IndentationError

# ❌ 错误：忘记缩进
if condition:
action1()            # 没有缩进，Python 会报 IndentationError
```

> 🔑 **关键规则**：`if` / `elif` / `else` / `for` / `while` / `def` / `with` 之后的行**必须**比它们缩进一级（通常 4 空格或 1 Tab，选一个后全文统一）。同一个块的所有行缩进必须一致。

#### 真值判断 —— `if x:` 的秘密

```python
# 在 Python 中，任何值都可以放在 if 后面
# Python 会把它们自动转换为 True 或 False

# 以下值视为 False（"假值"或"空值"）：
# None     → 空，表示"什么都没有"
# 0        → 数字零
# ""       → 空字符串
# []       → 空列表
# ()       → 空元组
# {}       → 空字典
# set()    → 空集合

# 其他所有值都视为 True（包括负数、空格字符串等）

# ═══════ 生信中的实际使用 ═══════
# 场景 1：判断一个基因列表是否有内容
if gene_list:                    # 等价于 if len(gene_list) > 0
    print("Found genes to process")
else:
    print("No genes found")

# 场景 2：安全地访问可能不存在的字典键
result = gene_dict.get("TP53")   # 如果 "TP53" 不在字典中，get() 返回 None
if result:                       # result 不是 None 也不是空，说明取到了
    process(result)
else:
    print("Gene not found in database")
```

### 2.3 循环

> 循环就是"把一件事重复做很多遍"。生信中最常见的：遍历每个基因、遍历文件的每一行、遍历每个样本。

#### for 循环的基本结构

```python
# for 临时变量 in 可迭代对象:
#     重复执行的代码
#
# "可迭代对象"就是"可以逐个取出来"的东西——列表、字典、文件的每一行、字符串的每个字符

# ═══════════════════════════════════════════
# 最简单：遍历基因列表
# ═══════════════════════════════════════════
genes = ["TP53", "BRCA1", "EGFR", "MYC"]

for gene in genes:
    print(f"Processing {gene}...")

# 这段代码的执行过程：
# 第 1 轮：gene = "TP53"， 打印 "Processing TP53..."
# 第 2 轮：gene = "BRCA1"，打印 "Processing BRCA1..."
# 第 3 轮：gene = "EGFR"， 打印 "Processing EGFR..."
# 第 4 轮：gene = "MYC"，  打印 "Processing MYC..."
# 循环结束，继续执行后面的代码
#
# 关键理解：gene 是一个"临时变量"，
# 每一轮循环它被自动赋值为列表中的下一个元素
# 变量名可以随便起，for g in genes: 也完全一样

# ═══════════════════════════════════════════
# 遍历字典：key 和 value 一起拿
# ═══════════════════════════════════════════
expression = {"TP53": 12.5, "BRCA1": 8.2, "EGFR": 45.1}

for gene, expr in expression.items():
    # expression.items() 把字典拆成 (键, 值) 对
    # 第 1 轮：gene="TP53", expr=12.5
    # 第 2 轮：gene="BRCA1", expr=8.2
    # 第 3 轮：gene="EGFR", expr=45.1
    if expr > 10:
        print(f"{gene} is highly expressed: {expr}")

# ═══════════════════════════════════════════
# 遍历文件每一行（生信超高频操作）
# ═══════════════════════════════════════════
with open("genes.txt") as f:
    for line in f:                    # f 是一个"可迭代对象"，每次给出一行
        line = line.strip()           # 去掉行末的换行符 \n（详见第 4 章）
        if line.startswith("#"):      # 如果这一行以 # 开头（注释行）
            continue                  # 跳过本轮循环，直接进入下一轮
        if not line:                  # 如果是空行
            continue                  # 也跳过
        print(line)

# ═══════════════════════════════════════════
# continue 和 break
# ═══════════════════════════════════════════
for gene in genes:
    if gene == "EGFR":
        continue   # 跳过本轮：不打印 EGFR，但继续处理后面的基因
    if gene == "MYC":
        break      # 终止整个循环：遇到 MYC 就停止，后面不再处理
    print(gene)
# 输出：TP53 → BRCA1 → (遇到 EGFR 跳过) → (遇到 MYC 终止)
```

#### enumerate() —— 需要"编号"的时候用

```python
# 场景：打印基因列表时，想在前面加上序号
genes = ["TP53", "BRCA1", "EGFR"]

# enumerate() 的作用：把 [a, b, c] 变成 [(0,a), (1,b), (2,c)]
# 每一轮给你 (索引, 元素)

for i, gene in enumerate(genes):
    # 第 1 轮：i=0, gene="TP53"
    # 第 2 轮：i=1, gene="BRCA1"
    # 第 3 轮：i=2, gene="EGFR"
    print(f"Gene {i+1}: {gene}")
# 输出：
# Gene 1: TP53
# Gene 2: BRCA1
# Gene 3: EGFR

# enumerate 还可以指定起始编号
for i, gene in enumerate(genes, start=1):
    print(f"Gene {i}: {gene}")       # i 从 1 开始，而不是 0
```

#### zip() —— 并行遍历多个列表

```python
# 场景：把基因名和表达量一一对应打印出来
genes = ["TP53", "BRCA1", "EGFR"]
exprs = [12.5, 8.2, 45.1]

# zip() 的作用：把多个列表"拉链"在一起
# 第 1 轮：(genes[0], exprs[0]) → ("TP53", 12.5)
# 第 2 轮：(genes[1], exprs[1]) → ("BRCA1", 8.2)
# 第 3 轮：(genes[2], exprs[2]) → ("EGFR", 45.1)

for gene, expr in zip(genes, exprs):
    print(f"{gene}: {expr}")

# 如果列表长度不一样，zip 会在最短的列表结束时停止
```

#### 一段代码看懂所有循环模式

```python
# 准备数据
genes = ["TP53", "BRCA1", "EGFR", "KRAS", "PTEN"]
expression = {"TP53": 12.5, "BRCA1": 8.2, "EGFR": 45.1, "KRAS": 3.2, "PTEN": 15.3}
pathways = {"TP53": "apoptosis", "BRCA1": "DNA repair", "EGFR": "proliferation",
            "KRAS": "proliferation", "PTEN": "PI3K"}

# 1. 普通遍历 —— 逐个处理基因
for gene in genes:
    print(gene)

# 2. 遍历字典获得键值对 —— 打印基因和表达量
for gene, expr in expression.items():
    print(f"{gene}: {expr}")

# 3. 带索引遍历 —— 打印排名
for rank, gene in enumerate(genes, start=1):
    print(f"#{rank}: {gene}")

# 4. 并行遍历 —— 合并两个列表的信息
for gene, path in zip(genes, pathways):
    print(f"{gene} belongs to {path} pathway")
```

### 2.4 函数

> 函数就是"把一段代码打包起来，起个名字，以后随时调用"。相当于一个生信实验的 SOP（标准操作流程）。

#### 为什么要写函数？

```python
# ❌ 不用函数：重复写同样的代码
seq1 = "ATGCGTACG"
gc1 = (seq1.count("G") + seq1.count("C")) / len(seq1)

seq2 = "CCGGAATT"
gc2 = (seq2.count("G") + seq2.count("C")) / len(seq2)

seq3 = "TTTTAAAAGGGGCCCC"
gc3 = (seq3.count("G") + seq3.count("C")) / len(seq3)
# 问题：代码重复、容易出错、修改时三处都要改

# ✅ 用函数：写一次，反复用
def gc_content(sequence):
    """返回序列的 GC 含量（0~1 之间的小数）"""
    seq = sequence.upper()              # 先统一转成大写（防止用户输入小写）
    gc = seq.count("G") + seq.count("C")
    return gc / len(seq)

gc1 = gc_content("ATGCGTACG")           # 0.666...
gc2 = gc_content("CCGGAATT")            # 0.5
gc3 = gc_content("TTTTAAAAGGGGCCCC")    # 0.5
```

#### 函数的解剖结构

```python
def gc_content(sequence):          # ← def 关键字 + 函数名 + (参数列表) + 冒号
    """返回序列的 GC 含量"""       # ← 文档字符串（docstring），说明函数做什么
    seq = sequence.upper()         # ← 函数体：实际执行的代码（必须缩进！）
    gc = seq.count("G") + seq.count("C")
    return gc / len(seq)           # ← 返回值：函数执行完毕后"交出来"的结果

# def            → define 的缩写，告诉 Python "我要定义一个函数"
# gc_content     → 函数名：动词或动词短语，小写加下划线（Python 命名习惯）
# (sequence)     → 参数：调用函数时传入的数据，是函数的"输入"
# return         → 函数的"输出"，把结果传回调用处
#                  return 之后函数就结束了，后面的代码不会执行
```

#### 参数、默认值和返回值

```python
# 1. 有默认值的参数
def filter_genes(expr_matrix, min_expr=1.0, min_cells=3):
    """
    筛选低表达基因

    参数:
        expr_matrix: 表达矩阵
        min_expr: 最低表达阈值（默认 1.0）
        min_cells: 最少需要几个细胞表达（默认 3）
    """
    # 这里写具体筛选逻辑...
    pass  # pass 是"占位符"，表示什么也不做（等后面用 Pandas 实现）

# 调用方式
filter_genes(data)                              # 使用默认值 min_expr=1.0, min_cells=3
filter_genes(data, min_expr=2.0)                # 只覆盖 min_expr
filter_genes(data, min_expr=2.0, min_cells=5)   # 全部指定

# ⚠️ 注意事项：有默认值的参数必须放在没有默认值的参数后面
# ❌ def f(a=1, b): 这是错误的
# ✅ def f(b, a=1): 这是正确的

# 2. 返回多个值
def calc_gene_stats(expressions):
    """计算基因表达量的三个统计量"""
    mean = sum(expressions) / len(expressions)
    # sum([1,2,3]) = 6, len([1,2,3]) = 3, mean = 2

    # 方差 = 每个值与均值之差的平方的平均
    variance = sum((x - mean) ** 2 for x in expressions) / len(expressions)
    # ** 2 是平方运算：3 ** 2 = 9
    # for x in expressions → 遍历每个表达值
    # (x - mean) ** 2 → 计算离均差的平方
    # sum(...) / len(...) → 求平均

    std = variance ** 0.5    # ** 0.5 就是开根号：标准差 = √方差
    cv = std / mean          # 变异系数 (CV)：标准差除以均值

    return mean, std, cv     # 返回多个值时，实际返回的是一个元组

# 接收多个返回值
avg, stdev, cv = calc_gene_stats([12.5, 8.2, 45.1, 0.0, 3.7])
# avg = 13.9, stdev = ..., cv = ...
# 这就是"元组解包"，变量个数必须和返回值的个数一致
```

#### lambda 函数 —— 一行函数

```python
# 有时候只需要一个简单的"一次性"函数，写完整的 def 太啰嗦
# lambda 就是"匿名函数"，语法：lambda 参数: 表达式

# 普通函数写法
def square(x):
    return x ** 2

# lambda 写法（两者完全等价）
square = lambda x: x ** 2

# 生信中 lambda 最常用于 Pandas 的 apply() 和 sort() 的 key 参数
genes = ["TP53", "BRCA1", "EGFR"]
genes.sort(key=lambda g: len(g))   # 按基因名长度排序
# key 参数：sort 会对每个元素先用 key 函数转换一下，再按转换后的值排序
# lambda g: len(g)：输入基因名 g，返回它的长度
# TP53→4, BRCA1→5, EGFR→4 → 但不能只按长度排，原顺序也有影响
```

### 2.5 列表推导式（生信利器）

> 列表推导式是 Python 最有特色的语法之一。它用一行代码替代 for 循环 + append，既快又易读。

#### 从 for 循环到列表推导式

```python
genes = ["TP53", "BRCA1", "egfr", "MYC", "brca2"]

# 需求：把所有基因名转成大写
# ❌ 传统写法（4 行）
genes_upper = []
for g in genes:
    genes_upper.append(g.upper())

# ✅ 列表推导式（1 行）
genes_upper = [g.upper() for g in genes]
# 结果：["TP53", "BRCA1", "EGFR", "MYC", "BRCA2"]

# ═══════ 语法分解 ═══════
# [g.upper() for g in genes]
#  ├──┬──┘  ├──┬─┘  ├──┬──┘
#  │   │      │      └── 从哪个可迭代对象中取元素
#  │   │      └── 循环变量（临时名字，每一轮自动赋值）
#  │   └── 对每个元素做什么操作（表达式）
#  └── 最终结果存放在一个新列表里

# 读法："对 genes 里的每个 g，计算 g.upper()，把所有结果收集成一个新列表"
```

#### 带条件的列表推导式

```python
# 需求：从 P 值列表中找出显著的
p_values = [0.001, 0.04, 0.08, 0.0001, 0.15]
sig_p = [p for p in p_values if p < 0.05]
# 结果：[0.001, 0.04, 0.0001]

# 读法："对 p_values 里的每个 p，如果 p < 0.05，就把它放进新列表"

# ═══════ 同时做变换和过滤 ═══════
genes = ["TP53", "BRCA1", "EGFR", "KRAS"]
result = [g.lower() for g in genes if len(g) > 4]
# 第 1 步过滤：只保留长度 > 4 的（BRCA1、EGFR、KRAS）
# 第 2 步变换：对保留的转小写（"brca1", "egfr", "kras"）

# ═══════ 字典推导式 ═══════
# 语法：{键: 值 for 变量 in 可迭代对象 if 条件}
expr_dict = {"TP53": 12.5, "BRCA1": 8.2, "EGFR": 45.1}
log_expr = {gene: round(val, 1) for gene, val in expr_dict.items() if val > 0}
# round(val, 1) → 保留 1 位小数
# 作用：创建一个新字典，只包含表达量 > 0 的基因，并且值保留 1 位小数
```

#### 列表推导式 vs for 循环：什么时候用哪个？

```python
# 用列表推导式：简单的数据转换和筛选
genes_upper = [g.upper() for g in genes]           # ✅ 一行，意图明确

# 用 for 循环：逻辑复杂、多步骤操作
for gene in genes:
    result = complex_analysis(gene)                 # 复杂逻辑
    if result.needs_correction():
        result = correct(result)                    # 多步骤处理
    save_to_database(result)                        # 有副作用（写入文件、数据库）
    print(f"Processed {gene}: status={result.status}")  # 多行输出
# 这种情况下用列表推导式反而难读
```

### 2.6 异常处理

> 异常就是"程序运行时出了意想不到的状况"。生信数据常常不完美：文件缺失、格式错误、网络中断……异常处理让程序在这些情况下不会直接崩溃，而是优雅地处理。

```python
# ═══════ try / except 的基本语法 ═══════
try:
    # 尝试执行这里的代码（"试试看"）
    with open("rnaseq_counts.txt") as f:
        header = f.readline()
except FileNotFoundError:
    # 如果文件不存在，就执行这里的代码
    print("Can't find the count file. Check your path.")
except PermissionError:
    # 如果没有读取权限，就执行这里的代码
    print("No permission to read the file.")
except Exception as e:
    # Exception 是所有异常的"父类"，会捕获所有其他类型的异常
    # as e 把异常对象保存到变量 e 里，可以查看具体错误信息
    print(f"Unexpected error: {e}")
# 程序继续执行（不会崩溃！）

# ═══════ 类比理解 ═══════
# try     → "我试试打开这个瓶盖"
# except  → "如果打不开：可能是盖子太紧（FileNotFoundError），
#            可能是手滑（PermissionError），
#            或者碰到其他意外情况（Exception）"
# 不管哪种情况，你都知道该怎么做，而不是把瓶子摔了

# ═══════ 生信中的防御性编程 ═══════
def safe_gc(seq):
    """安全地计算 GC 含量，处理各种异常输入"""
    try:
        gc_count = seq.count("G") + seq.count("C")  # 统计 G 和 C
        return gc_count / len(seq)                    # 计算比例
    except ZeroDivisionError:
        # 如果 len(seq) == 0，除法会触发 ZeroDivisionError
        return 0.0            # 空序列的 GC 含量记为 0
    except TypeError:
        # 如果 seq 不是字符串（比如传入了 None 或数字），
        # .count() 方法会触发 TypeError
        print(f"Expected a string, got {type(seq)}")
        return None           # 返回 None 表示计算失败

# 测试
print(safe_gc("ATGCGTACG"))   # 0.666...    正常
print(safe_gc(""))            # 0.0         空序列
print(safe_gc(None))          # None        错误输入，打印提示并返回 None
```

---

## 3. 生信核心数据结构

> Python 提供了四种内置的"容器"：list、tuple、dict、set。选择正确的容器能让代码简洁 3 倍。
>
> 回忆口诀：**列表存基因，字典建映射，集合做取交，元组不改动。**

### 3.1 什么是"数据结构"？

数据结构 = 数据在内存中的"摆放方式"。就像你要整理实验室 —— 试剂可以随便扔在桌上（混乱），也可以分类放在架子上（有序）。Python 的四种数据结构就是四种不同的"整理方式"，各有擅长的事。

```
场景                                → 最适合的数据结构
──────────────────────────────────────────────────────
要分析的前 10 个基因名              → list（有序，可增删）
一个基因组位点的坐标 (chr, 起点, 终点) → tuple（无需修改）
基因名查找表达量                    → dict（键→值映射）
取两组差异基因的交集                → set（去重，集合运算）
统计每个染色体上有多少个突变        → defaultdict(int)（自动计数）
```

### 3.2 列表 list —— 最常用的容器

```python
# 创建一个列表：用方括号 []，元素用逗号分隔
genes = ["TP53", "BRCA1", "EGFR", "MYC"]

# ═══════ 列表的特点 ═══════
# 1. 有序：元素的顺序是固定的，第一个是 genes[0]
# 2. 可修改：可以增、删、改元素
# 3. 可重复：同一个值可以出现多次
# 4. 可混合类型（但不推荐）：["TP53", 12.5, True] 语法上合法但不建议

# ═══════ 访问元素：用方括号 + 索引 ═══════
genes[0]     # "TP53"   ← 索引从 0 开始！（这是 Python 的铁律）
genes[1]     # "BRCA1"
genes[-1]    # "MYC"    ← 负数索引从末尾倒着数：-1 是最后一个
genes[-2]    # "EGFR"   ← -2 是倒数第二个

# 为什么从 0 开始？这是历史惯例，大部分编程语言都从 0 开始。
# 记忆技巧：索引 = "从开头跳过的元素个数"
# genes[0] = 跳过 0 个 = 第一个
# genes[2] = 跳过 2 个 = 第三个
```

#### 列表的增删改查

```python
genes = ["TP53", "BRCA1", "EGFR"]

# ─── 增加 ───
genes.append("PTEN")       # 在末尾添加一个元素
# genes → ["TP53", "BRCA1", "EGFR", "PTEN"]

genes.insert(0, "KIT")     # 在指定位置插入：在第 0 个位置插入 "KIT"
# genes → ["KIT", "TP53", "BRCA1", "EGFR", "PTEN"]
# insert(索引, 元素)：在索引位置之前插入，原有元素往后移

# ─── 删除 ───
genes.remove("MYC")        # 删除指定值的元素（删除第一个匹配项）
# 如果 "MYC" 不在列表中，会抛出 ValueError

last = genes.pop()         # 弹出最后一个元素（取出 + 删除）
# last = "PTEN", genes → ["KIT", "TP53", "BRCA1", "EGFR"]
# pop(索引) 可以从指定位置弹出,不写索引默认最后一个

# ─── 查找 ───
idx = genes.index("EGFR")  # 查找元素的位置，返回索引号
# idx = 3
# 如果找不到，会抛出 ValueError

count = genes.count("KIT") # 统计元素出现次数
# count = 1

# ─── 修改 ───
genes[1] = "BRCA2"         # 直接修改指定位置的元素
# genes → ["KIT", "BRCA2", "BRCA1", "EGFR"]
```

#### 切片（Slicing）—— 列表的灵魂操作

```python
# 切片的完整语法：list[起始:终止:步长]
# 记忆口诀："从起始开始，到终止之前结束，每次走步长步"

genes = ["KIT", "TP53", "BRCA1", "EGFR", "PTEN"]

# ─── 基本切片 ───
genes[0:3]    # ["KIT", "TP53", "BRCA1"]
# 读法：取索引 0、1、2（终止索引 3 不包含！）
# 这是 Python 的"左闭右开"原则：包括 start，不包括 end

genes[:3]     # ["KIT", "TP53", "BRCA1"]
# 省略起始：从开头开始

genes[2:]     # ["BRCA1", "EGFR", "PTEN"]
# 省略终止：到末尾结束

genes[-2:]    # ["EGFR", "PTEN"]
# 从倒数第 2 个到末尾

# ─── 带步长的切片 ───
genes[::2]    # ["KIT", "BRCA1", "PTEN"]
# 每隔 1 个取 1 个（步长为 2）

genes[::-1]   # ["PTEN", "EGFR", "BRCA1", "TP53", "KIT"]
# 步长为 -1 = 反转列表！这条在序列反转中经常用到（见 5.2 节）

# ═══════ 生信中的切片应用 ═══════
# 取表达量最高的前 10 个基因
sorted_genes = sorted(genes, key=lambda g: expression.get(g, 0), reverse=True)
top10 = sorted_genes[:10]          # 前 10 个

# 把基因列表等分成训练集和测试集
mid = len(genes) // 2              # // 是整除：7 // 2 = 3
train = genes[:mid]                # 前一半
test = genes[mid:]                 # 后一半
```

#### 列表排序

```python
genes = ["TP53", "brca1", "EGFR", "myc", "ALK"]

# sort() → 原地排序，直接修改原列表
genes.sort()                              # 按字母顺序
# ['ALK', 'EGFR', 'TP53', 'brca1', 'myc']
# 注意：大写字母排在小写字母前面（ASCII 码顺序）

genes.sort(key=str.lower)                 # 忽略大小写排序
# ['ALK', 'brca1', 'EGFR', 'myc', 'TP53']
# key 参数：先用 str.lower 把每个基因转小写，再按转换后的值排序

genes.sort(key=lambda g: len(g))          # 按基因名长度排序
# ['myc', 'ALK', 'TP53', 'EGFR', 'brca1']

genes.sort(key=lambda g: len(g), reverse=True)  # 从长到短
# ['brca1', 'EGFR', 'TP53', 'ALK', 'myc']

# sorted() → 返回新列表，不修改原列表
sorted_genes = sorted(genes, key=str.lower)  # 推荐：不改变原数据
```

#### 列表去重

```python
genes_with_dup = ["TP53", "BRCA1", "TP53", "EGFR", "BRCA1", "EGFR"]

# 方法：转成 set（自动去重），再转回 list
unique_genes = list(set(genes_with_dup))
# 顺序可能会变！因为 set 是无序的

# 如果要保留原始顺序的同时去重：
seen = set()
unique_ordered = []
for g in genes_with_dup:
    if g not in seen:
        unique_ordered.append(g)
        seen.add(g)
# unique_ordered = ["TP53", "BRCA1", "EGFR"]
```

### 3.3 元组 tuple —— 不可变的数据打包

```python
# 创建元组：用圆括号 ()，元素逗号分隔
snp_position = ("chr17", 7676594, 7676595)

# ═══════ 元组 vs 列表 ═══════
#          列表 list        元组 tuple
# 符号     [ ]              ( )
# 可否修改 可增删改         不可修改（"不可变"）
# 用途     动态集合          固定数据打包
# 速度     稍慢             稍快

# ═══════ 为什么生信中需要元组？ ═══════
# 基因组坐标是固定的，不应该被意外修改
# 如果代码中有 snp_position[0] = "chr18"，用元组会直接报错，避免 bug
# 用列表则可能被悄悄改动，后续分析全错了

# ═══════ 元组解包（unpacking） ═══════
chrom, start, end = snp_position
# chrom = "chr17", start = 7676594, end = 7676595
# 左边变量个数必须和元组元素个数一致

# 如果只关心部分元素，用 _ 占位
chrom, _, _ = snp_position     # 只要染色体号
_, start, end = snp_position   # 只要起止位置

# ═══════ 函数返回多个值 = 返回元组 ═══════
def analyze_gene(gene_name):
    """模拟基因分析，返回多个结果"""
    expr = 15.3
    is_oncogene = True
    pathway = "apoptosis"
    return expr, is_oncogene, pathway   # 实际返回的是 (15.3, True, "apoptosis")

# 解包接收
expression, oncogene, path = analyze_gene("TP53")
```

### 3.4 字典 dict —— 生信中最强大的数据结构

> 字典是**键值对**的容器。你可以通过"键"快速查找"值"，就像用基因名查找它的表达量。

```python
# 创建字典：用花括号 {}，格式 {键: 值, 键: 值, ...}
expression = {
    "TP53": 12.5,
    "BRCA1": 8.2,
    "EGFR": 45.1
}
# 键 = 基因名 (str)，值 = 表达量 (float)

# ═══════ 字典的核心操作 ═══════
# 取值
expr_tp53 = expression["TP53"]        # 12.5 —— 直接取
# ⚠️ 如果键不存在，会抛出 KeyError

expr_kras = expression.get("KRAS", 0)  # 0 —— 安全取，不存在返回默认值
# get(键, 默认值) 是生信中最常用安全的取值方式！
# 很多基因可能不在字典里，用 get 避免程序崩溃

# 添加 / 修改
expression["PTEN"] = 15.3             # 添加新键值对（PTEN 之前不存在）
expression["TP53"] = 13.0             # 修改已有键的值

# 删除
del expression["EGFR"]                # 删除键值对

# 检查键是否存在
if "TP53" in expression:              # in 检查的是键，不是值
    print("TP53 found")

# 获取所有键 / 所有值
gene_names = list(expression.keys())      # ["TP53", "BRCA1", "PTEN"]
expr_values = list(expression.values())   # [13.0, 8.2, 15.3]
```

#### 字典的三种生信应用场景

```python
# ═══════ 场景 1：简单的查询表 ═══════
# 用途：基因名 → 表达量、基因 ID → 基因名、转录本 ID → 基因 ID
id_to_name = {}
with open("gene_annotations.tsv") as f:
    for line in f:
        cols = line.strip().split("\t")
        ensembl_id = cols[0]           # 第一列：Ensembl ID，如 ENSG00000141510
        gene_name = cols[1]            # 第二列：基因名，如 TP53
        id_to_name[ensembl_id] = gene_name
# 之后就可以通过 Ensembl ID 快速查找基因名
# id_to_name["ENSG00000141510"] → "TP53"

# ═══════ 场景 2：嵌套字典（结构化信息） ═══════
# 用途：存储每个基因的多项注释信息
gene_annot = {
    "TP53": {
        "chrom": "17p13.1",
        "type": "tumor_suppressor",
        "length": 19149,
        "exon_count": 11
    },
    "EGFR": {
        "chrom": "7p11.2",
        "type": "oncogene",
        "length": 188564,
        "exon_count": 28
    },
}

# 访问嵌套信息
print(gene_annot["TP53"]["chrom"])       # "17p13.1"
print(gene_annot["EGFR"]["exon_count"])  # 28

# ═══════ 场景 3：用 defaultdict 做计数和分组 ═══════
from collections import defaultdict

# defaultdict 和普通 dict 的唯一区别：
# 当你访问一个不存在的键时，它自动创建默认值，而不是报错

# ── 计数模式 ──
chr_mutations = defaultdict(int)        # int() 的默认值是 0
for mutation in mutation_list:
    chrom = mutation["chrom"]           # 突变所在的染色体
    chr_mutations[chrom] += 1           # 不需要先判断键是否存在！
# chr_mutations → {"chr1": 342, "chr2": 281, "chr17": 563, ...}

# 对比：如果用普通 dict，你需要这样写：
# if chrom not in chr_mutations:
#     chr_mutations[chrom] = 0
# chr_mutations[chrom] += 1

# ── 分组模式 ──
chr_genes = defaultdict(list)           # list() 的默认值是空列表 []
for gene in all_genes:
    chr_genes[gene["chrom"]].append(gene["name"])
# 结果：{"chr1": ["TP73", "MTHFR", ...], "chr2": ["ALK", ...], ...}
# 当 chr_genes["chr1"] 第一次被访问时，自动变成 []，然后 gene["name"] 被 append 进去
```

#### 字典遍历高级技巧

```python
expression = {"TP53": 12.5, "BRCA1": 8.2, "EGFR": 45.1}

# ── 基本遍历 ──
for gene in expression:                    # 默认遍历键
    print(gene)

for gene, expr in expression.items():      # 同时遍历键和值（最常用）
    print(f"{gene}: {expr}")

# ── 按值排序后遍历 ──
# 需求：按表达量从高到低打印基因
sorted_genes = sorted(expression.items(),
                      key=lambda item: item[1],    # item = ("基因名", 表达量)
                      reverse=True)               # item[0]=基因名, item[1]=表达量
for gene, expr in sorted_genes:
    print(f"{gene}: {expr}")
# 输出：EGFR: 45.1 → TP53: 12.5 → BRCA1: 8.2
```

### 3.5 集合 set —— 去重和集合运算

```python
# 创建集合：用花括号 {} 或 set()
oncogenes = {"TP53", "EGFR", "KRAS", "MYC"}
# 注意：空集合必须用 set()，不能用 {}（{} 是空字典！）

# ═══════ 集合的两大特点 ═══════
# 1. 无序：元素没有固定顺序，不能用索引访问
# 2. 不重复：相同的元素自动只保留一份

# ═══════ 集合运算 —— 韦恩图的底层实现 ═══════
deg_cancer = {"TP53", "BRCA1", "EGFR", "MYC", "KRAS"}      # 癌症中的差异基因
deg_drug_treated = {"TP53", "EGFR", "PTEN", "AKT1", "KRAS"} # 药物处理后的差异基因

# 交集 &：两种条件下都差异表达的基因（可能是核心靶点）
common = deg_cancer & deg_drug_treated
# {"TP53", "EGFR", "KRAS"}

# 并集 |：至少在一个条件下差异表达的基因
all_deg = deg_cancer | deg_drug_treated
# {"TP53", "BRCA1", "EGFR", "MYC", "KRAS", "PTEN", "AKT1"}

# 差集 -：只在癌症中差异、药物不影响（癌症特异性靶点）
cancer_only = deg_cancer - deg_drug_treated
# {"BRCA1", "MYC"}

# 对称差集 ^：只在一个条件中差异的（非共有靶点）
unique = deg_cancer ^ deg_drug_treated
# {"BRCA1", "MYC", "PTEN", "AKT1"}

# ═══════ 集合的其他常用操作 ═══════
genes = {"TP53", "EGFR", "KRAS"}
genes.add("PTEN")                        # 添加一个元素
genes.remove("KRAS")                     # 删除一个元素（不存在会报错）
genes.discard("MYC")                     # 安全删除（不存在也不报错）
"TP53" in genes                          # True —— 检查元素是否在集合中
len(genes)                               # 3 —— 集合大小
```

---

## 4. 文件读写：生信格式大全

### 4.1 基础概念：文件路径、模式、with 语句

#### 什么是"文件路径"？

```
绝对路径：从盘符开始的完整路径
  Windows: C:\Users\y9435\Desktop\data\genes.txt
  Linux:   /home/y9435/data/genes.txt

相对路径：从当前工作目录开始的路径
  如果你在 C:\Users\y9435\Desktop 目录下工作：
  data\genes.txt   →   实际指向 C:\Users\y9435\Desktop\data\genes.txt
  .\genes.txt      →   . 表示当前目录
  ..\data\genes.txt →  .. 表示上一级目录 (C:\Users\y9435\data\genes.txt)
```

#### 文件打开模式

| 模式 | 含义 | 何时使用 |
|------|------|----------|
| `"r"` | 只读 (read) | 读取已有文件（默认模式，不写也行） |
| `"w"` | 只写 (write) | 创建新文件 / **覆盖已有文件**（谨慎！） |
| `"a"` | 追加 (append) | 在文件末尾添加内容，不覆盖已有内容 |

#### `with` 语句 —— 为什么必须用它？

```python
# ❌ 不推荐：手动 open 和 close
f = open("genes.txt")
content = f.read()
f.close()                    # 如果第 2 行出错，close() 不会执行，文件句柄泄漏

# ✅ 推荐：with 语句
with open("genes.txt") as f:
    content = f.read()
# 不管代码块里发生了什么（正常结束 or 抛出异常），
# Python 都会自动关闭文件。安全、干净。

# with 的语法：
# with open(文件路径, 模式) as 文件对象:
#     使用文件对象进行操作
# 缩进块结束后，文件被自动关闭
```

### 4.2 基础文件读写

```python
# ═══════ 读取整个文件 ═══════
with open("gene_list.txt") as f:   # 不写模式默认 "r"
    content = f.read()             # read() 把整个文件读成一个字符串
# 适用场景：小文件（如基因列表、配置文件）
# 不适用：几百 MB 的表达矩阵——会把内存撑爆

# ═══════ 逐行读取（推荐！不占内存） ═══════
with open("expression_matrix.tsv") as f:
    for line in f:                 # f 是"可迭代对象"，每次给出一行
        line = line.strip()        # ⚠️ 必须去换行符！（原因见下）
        if not line:               # 跳过空行
            continue
        if line.startswith("#"):   # 跳过注释行（VCF/GFF 文件中常见）
            continue
        # 现在可以安全地处理这一行了
        cols = line.split("\t")
        # ...

# ═══════ 写入文件 ═══════
with open("output.txt", "w") as f: # "w" 模式会覆盖已有文件！
    f.write("gene\texpression\n")  # write() 不会自动加换行符，要手动加 \n
    for gene, expr in sorted_expr:
        f.write(f"{gene}\t{expr}\n")
```

#### 为什么要 strip()？

```python
# 从文件读出的每一行，末尾都有一个看不见的换行符 \n
line = "TP53\t12.5\n"     # 实际内容
#               ↑ 换行符，代表"这一行结束了"

# 如果不 strip()：
cols = line.split("\t")    # ["TP53", "12.5\n"]  ← 第二项带 \n！
expr = float("12.5\n")     # ValueError: could not convert string to float

# strip() 去掉首尾的空白字符（空格、\n、\t、\r 等）
clean = line.strip()       # "TP53\t12.5"
cols = clean.split("\t")   # ["TP53", "12.5"]  ← 干净了！
expr = float(cols[1])      # 12.5 ← 成功！
```

### 4.3 FASTA 文件（序列文件）

```
FASTA 文件的结构：
>序列名称 描述信息    ← 标题行，以 > 开头
ATGGAGGAGCCGCAG...   ← 序列行，可以有多行
>序列名称 描述信息    ← 下一个序列的标题行
ATGGATTTATCTGCT...   ← 下一个序列的序列行
```

```python
def read_fasta(filepath):
    """
    读取 FASTA 文件，返回 {序列名: 序列字符串} 的字典

    算法思路（跟着走一遍就懂了）：
    1. 逐行读文件
    2. 遇到 > 开头的行 = 新序列开始了
       → 先把上一条序列存进字典
       → 开始记录新序列
    3. 不是 > 开头的行 = 序列数据
       → 追加到当前序列后面
    4. 文件读完记得存最后一条序列
    """
    sequences = {}             # 最终结果
    current_name = None        # 当前在处理的序列名（None 表示还没开始）
    current_seq = []           # 当前序列的各个片段（用列表避免反复拼接字符串）

    with open(filepath) as f:
        for line in f:
            line = line.strip()
            if not line:       # 跳过空行
                continue

            if line.startswith(">"):
                # 遇到新序列头，先保存上一个序列
                if current_name is not None:
                    sequences[current_name] = "".join(current_seq)
                    # "".join(["ATG", "GAG", "CAG"]) → "ATGGAGCAG"

                # 开始新序列
                current_name = line[1:]  # line[1:] 跳过头部的 >
                # 如果标题行是 ">TP53 transcript 1"
                # line[1:] = "TP53 transcript 1"
                current_seq = []         # 重置序列缓存

            else:
                # 序列数据行
                current_seq.append(line)

        # 文件读完了，别忘了保存最后一条序列！
        if current_name is not None:
            sequences[current_name] = "".join(current_seq)

    return sequences

# 使用示例
seqs = read_fasta("genome.fasta")
for name, seq in seqs.items():
    print(f"{name}: {len(seq)} bp, GC content={gc_content(seq):.2%}")
    # :.2% 是格式说明符：把小数格式化成百分比，保留 2 位小数
    # 0.666 → "66.67%"
```

### 4.4 CSV / TSV 文件

```python
# TSV = Tab-Separated Values（制表符分隔）
# CSV = Comma-Separated Values（逗号分隔）
# 生信中 TSV 远比 CSV 常见，因为基因名和表达量不会包含制表符

# 手动解析 TSV（理解原理，实际用 Pandas 更方便）
with open("expression.tsv") as f:
    header = f.readline().strip().split("\t")
    # readline()   → 读第一行
    # strip()      → 去换行符
    # split("\t")  → 按 Tab 分割，得到一个列表
    # header = ["gene_id", "Sample1", "Sample2", "Sample3", ...]

    for line in f:
        cols = line.strip().split("\t")
        gene = cols[0]                     # 第一列：基因名
        values = [float(v) for v in cols[1:]]  # 其余列：表达量（转 float）
        # 列表推导式：对 cols[1:] 中的每个字符串 v，转换成 float
```

### 4.5 BED 文件（基因组区间）

BED 格式存储基因组上的区间，如基因位置、SNP 位置、peak 位置等。字段用 Tab 分隔：

```
染色体  起始  终止  名称  分数  链方向
chr1    11873 14409 DDX11L1 0 +
chr1    14362 29806 WASH7P 0 -
```

```python
def read_bed(filepath):
    """读取 BED 文件，返回区间列表。每个区间是一个字典。"""
    intervals = []
    with open(filepath) as f:
        for line in f:
            # 跳过注释行（以 # 开头）和 track 定义行
            if line.startswith("#") or line.startswith("track"):
                continue

            cols = line.strip().split("\t")

            # BED 格式有 3-12 列，前三列是必须的，后面是可选的
            # 用条件表达式安全取值：如果列不够多就用默认值
            intervals.append({
                "chrom":  cols[0],
                "start":  int(cols[1]),    # BED 坐标是 0-based（起点算 0）
                "end":    int(cols[2]),
                "name":   cols[3] if len(cols) > 3 else ".",   # 三元表达式
                "strand": cols[5] if len(cols) > 5 else ".",
            })
            # 三元表达式语法：值A if 条件 else 值B
            # 如果条件成立用 A，不成立用 B
            # 等价于：
            # if len(cols) > 3:
            #     name = cols[3]
            # else:
            #     name = "."

    return intervals

# 判断某个 SNP 是否落在基因区域内的工具函数
def is_in_gene(snp_chrom, snp_pos, gene_interval):
    """snp_pos 是否在 gene_interval 内部？"""
    return (snp_chrom == gene_interval["chrom"] and
            gene_interval["start"] <= snp_pos < gene_interval["end"])
    # 注意：右边界是 < 不是 <=，因为 BED 坐标是 0-based，end 不包含在内
```

### 4.6 文件路径处理

```python
import os
import glob

# ── 路径拼接（跨平台兼容） ──
# ❌ 硬编码（只在 Windows 上能用）
path = "data\\rnaseq\\project1"

# ✅ 用 os.path.join（Windows 和 Linux 都能用）
data_dir = os.path.join("data", "rnaseq", "project1")
# Windows → data\rnaseq\project1
# Linux   → data/rnaseq/project1

# ── 路径分解 ──
filepath = "C:\\Users\\y9435\\data\\sample1.fastq.gz"
os.path.basename(filepath)           # "sample1.fastq.gz" —— 文件名
os.path.dirname(filepath)            # "C:\\Users\\y9435\\data" —— 目录
os.path.splitext(filepath)           # ("C:\\Users\\...\\sample1.fastq", ".gz") —— 拆扩展名
os.path.exists(filepath)             # 检查文件/目录是否存在

# ── 批量读取文件 ──
fastq_files = glob.glob("data/*.fastq.gz")
# glob.glob 用通配符匹配文件：
# *       → 匹配任意字符（不含路径分隔符）
# **      → 递归匹配任意路径
# *.gz    → 所有 .gz 文件
# data/*  → data 目录下的所有文件
# **/*.bam → 所有子目录下的 .bam 文件（需设置 recursive=True）

bam_files = glob.glob("**/*.bam", recursive=True)

for fq in fastq_files:
    # 从文件名提取样本名："data/sample1.fastq.gz" → "sample1"
    sample_name = os.path.basename(fq).replace(".fastq.gz", "")
    print(f"Processing {sample_name}...")
```

---

## 5. 字符串与序列操作

> 生物信息学最核心的数据 —— DNA、RNA、蛋白质序列 —— 在 Python 中就是字符串。掌握字符串操作等于掌握了序列操作。

### 5.1 字符串的本质

```python
# Python 中，字符串是一个"字符序列"
seq = "ATGGAGGAGCCGCAG"

# 和列表一样，可以：
seq[0]       # "A" —— 按索引取字符（第 0 个字符）
seq[0:3]     # "ATG" —— 切片
len(seq)     # 15 —— 序列长度
for base in seq:    # 遍历每个碱基
    print(base)

# ⚠️ 但和列表不一样，字符串不可修改！
# seq[0] = "T"  →  TypeError! 不能直接改字符串里的单个字符
# 要修改只能创建新字符串：new_seq = "T" + seq[1:]
```

### 5.2 字符串常用方法速查

```python
seq = "ATGGAGGAGCCGCAG"

# ═══════ 大小写 ═══════
seq.upper()                   # "ATGGAGGAGCCGCAG" —— 全部大写
seq.lower()                   # "atggaggagccgcag" —— 全部小写
# 生信意义：序列分析前统一大小写，因为用户可能输入小写 atcg

# ═══════ 搜索和计数 ═══════
len(seq)                      # 15 —— 序列长度（有多少个字符）
seq.count("G")                # 5 —— 统计 G 出现次数
seq.count("CG")               # 2 —— 统计 CpG 二核苷酸出现次数！
                              # count 可以统计任意长度的子串
seq.find("GAG")               # 3 —— 子串第一次出现的位置（索引）
                              # 找不到返回 -1（而不是报错！）
seq.find("TTT")               # -1 —— 序列中没有 TTT

# ═══════ 判断开头结尾 ═══════
seq.startswith("ATG")         # True —— 以起始密码子开始
seq.startswith(("ATG", "GTG"))# True —— 以 ATG 或 GTG 开始（注意双层括号）
seq.endswith("TAG")           # False —— 不是以终止密码子结尾
# 生信意义：判断 CDS 是否完整（有起始和终止密码子）

# ═══════ 替换 ═══════
seq.replace("T", "U")         # "AUGGAGGAGCCGCAG" —— 转录：DNA → RNA
                              # 把所有的 T 替换为 U
                              # 返回新字符串，原字符串不变！
rna = seq.replace("T", "U").replace("G", "g")  # 可以链式调用

# ═══════ 去除空白 ═══════
line = "  ATGCGTAC\n  "       # 模拟从文件读到的一行
line.strip()                  # "ATGCGTAC" —— 去首尾空白
line.rstrip()                 # "  ATGCGTAC" —— 只去右边空白（rstrip = right strip）
line.lstrip()                 # "ATGCGTAC\n  " —— 只去左边空白

# ═══════ 连接和拆分 ═══════
# split()：把字符串按分隔符拆成列表
"gene1,gene2,gene3".split(",")     # ["gene1", "gene2", "gene3"]
"TP53\t12.5\n".strip().split("\t")  # ["TP53", "12.5"] —— 生信常用模式

# join()：把列表用分隔符粘成一个字符串
"_".join(["TP53", "BRCA1", "EGFR"])    # "TP53_BRCA1_EGFR"
# 注意：join 是字符串的方法，调用者是"分隔符"
# "_".join(列表) = 用 _ 作为胶水，把列表元素粘起来
# 和 split 是互逆操作！
```

### 5.3 生信序列操作实战

```python
# ═══════ 1. 反向互补 ═══════
# str.maketrans 创建一个"翻译表"：把 A 映射到 T，T 映射到 A，C ↔ G
complement_table = str.maketrans("ATCGatcg", "TAGCtagc")
# maketrans 的参数：两个等长字符串
# 第一个字符串的每个字符 → 第二个字符串对应位置的字符
# A→T, T→A, C→G, G→C, a→t, t→a, c→g, g→c

def reverse_complement(seq):
    """返回 DNA 序列的反向互补链"""
    # seq[::-1] 是切片：步长为 -1 = 反转字符串
    # .translate(table) 用翻译表替换每个字符
    return seq[::-1].translate(complement_table)

print(reverse_complement("ATGCGT"))     # "ACGCAT"
# 过程：ATGCGT → 反转 → TGCGTA → 互补 → ACGCAT

# ═══════ 2. DNA → 蛋白质翻译 ═══════
# 遗传密码子表：3 个碱基 → 1 个氨基酸
CODON_TABLE = {
    "ATA":"I", "ATC":"I", "ATT":"I", "ATG":"M",  # M = 甲硫氨酸（起始密码子）
    "ACA":"T", "ACC":"T", "ACG":"T", "ACT":"T",  # T = 苏氨酸
    # ... （完整表格见之前版本，此处省略以节省篇幅）
    "TAA":"*", "TAG":"*", "TGA":"*",             # * = 终止密码子
}

def translate(dna_seq):
    """将 DNA 序列翻译成蛋白质序列"""
    protein = []
    # range(起始, 终止, 步长)
    # 从 0 开始，到 len-2 结束（保证每次取 3 个完整碱基），步长 3
    for i in range(0, len(dna_seq) - 2, 3):
        codon = dna_seq[i:i+3]       # 取出当前的 3 连体
        # dna_seq[i:i+3]：从 i 开始，取到 i+3 之前（即取 i, i+1, i+2 三个字符）
        codon = codon.upper()        # 统一大写
        aa = CODON_TABLE.get(codon, "X")  # 查密码子表，找不到用 X 表示未知
        if aa == "*":                # 遇到终止密码子
            break                    # 翻译结束
        protein.append(aa)           # 把氨基酸加到蛋白质序列中
    return "".join(protein)          # 把氨基酸列表拼成一个字符串

print(translate("ATGGAGGAGCCGCAG"))     # "MEEPQ"
# Met-Glu-Glu-Pro-Gln → MEEPQ

# ═══════ 3. k-mer 提取 ═══════
def get_kmers(seq, k=3):
    """提取序列中所有的 k-mer（长度为 k 的子串）"""
    kmers = []
    for i in range(len(seq) - k + 1):
        # range(len(seq) - k + 1)：
        # 假设 seq="ATGCGT"，k=3
        # len=6, 6-3+1=4 → range(4) → i = 0,1,2,3
        kmer = seq[i:i+k]      # 每次取 k 个字符
        kmers.append(kmer)
    return kmers
    # 等价的一行版：return [seq[i:i+k] for i in range(len(seq)-k+1)]

print(get_kmers("ATGCGT", k=3))
# ["ATG", "TGC", "GCG", "CGT"]

# k-mer 的生信意义：
# - k=3：密码子频率分析（三连体使用偏好）
# - k=6：转录因子结合位点（motif）识别
# - k=20+：用于基因组组装的 de Bruijn 图
```

### 5.4 正则表达式入门

> 正则表达式 (regex) 是一种"模式匹配语言"，当 `find()` 和 `startswith()` 不够灵活时使用。

```python
import re

# ═══════ re 常用函数 ═══════
# re.search(模式, 字符串)     → 找第一个匹配（返回 Match 对象或 None）
# re.findall(模式, 字符串)    → 找所有匹配（返回字符串列表）
# re.finditer(模式, 字符串)   → 找所有匹配（返回迭代器，可获取位置）
# re.match(模式, 字符串)      → 从字符串开头匹配
# re.sub(模式, 替换, 字符串)  → 查找替换

# ═══════ 常用正则符号（记住这些就够 90%） ═══════
# .      任意一个字符
# \d     任意一个数字 (0-9)
# \w     任意一个字母或数字或下划线
# +      前面的东西出现 1 次或多次
# *      前面的东西出现 0 次或多次
# ?      前面的东西出现 0 次或 1 次
# [abc]  a、b、c 中的任意一个
# [^abc] 除了 a、b、c 之外的任意字符
# ^      字符串的开头
# $      字符串的结尾
# |      或者（OR）
# ( )    分组（捕获）

# ═══════ 生信中的正则应用 ═══════

# 1. 从 FASTA 标题行提取基因名
header = ">ENST00000269305.9|ENSG00000141510.17|TP53|protein_coding"
# r"..." = raw string：反斜杠不会被转义，写正则时建议加 r
match = re.search(r"\|([A-Z0-9]+)\|", header)
# 解释：\| → 匹配竖线 |（因为 | 在正则里有特殊含义，需要转义）
#       ([A-Z0-9]+) → 匹配一个或多个大写字母或数字，用括号捕获
if match:
    gene_name = match.group(1)  # 转义符
    print(gene_name)            # "TP53"

# 2. 检查序列是否只含标准核苷酸
def is_valid_dna(seq):
    # ^ 开头, $ 结尾, + 出现一次或多次
    # 整体含义：从头到尾全部是 ATCG（大小写均可）
    # bool() 把 Match 对象或 None 转成 True 或 False
    return bool(re.match(r"^[ATCGNatcgn]+$", seq))

# 3. 找出序列中所有 CG 位置
seq = "ATCGCCGTACG"
cpg_positions = [m.start() for m in re.finditer(r"CG", seq)]
# re.finditer 返回 Match 对象的迭代器
# m.start() 返回匹配的起始位置（索引）
# [1, 4, 8]

# 4. 提取文本中的 GO ID
desc = "This gene is involved in apoptosis (GO:0006915) and cell cycle (GO:0007049)"
go_terms = re.findall(r"GO:\d+", desc)
# GO: 字面匹配
# \d+ 匹配一个或多个数字
# ["GO:0006915", "GO:0007049"]
```

---

## 6. NumPy：基因表达矩阵的基石

> scikit-learn、Scanpy、scVI 等一切生信 ML/DL 库的底层都是 NumPy。基因表达矩阵本质就是一个 NumPy 二维数组。

### 6.1 什么是 NumPy？

NumPy = Numerical Python。它提供了一个核心数据结构 `ndarray`（N-dimensional array，多维数组），能高效地进行数值计算。

```python
import numpy as np    # 约定俗成：numpy 导入为 np

# ═══════ Python 列表 vs NumPy 数组 ═══════
# Python 列表：分散在内存中，元素类型可以混搭
py_list = [12.5, 8.2, 45.1]

# NumPy 数组：连续存储在内存中，所有元素必须是同一类型
np_array = np.array([12.5, 8.2, 45.1])

# 为什么 NumPy 更快？
# 1. 连续内存 = CPU 缓存友好
# 2. 底层用 C 实现，没有 Python 循环的开销
# 3. 向量化运算 = 一条指令操作整块数据（SIMD）
# 结论：同样计算，NumPy 比纯 Python 快 100~1000 倍！
```

### 6.2 创建数组

```python
# ── 从列表创建 ──
# 一维：一个基因在多个样本中的表达
expr = np.array([12.5, 8.2, 45.1, 0.0, 3.7])
# 形状：一维，长度 5

# 二维：5 个基因 × 3 个样本的表达矩阵
matrix = np.array([
    [12.5, 15.2, 10.1],      # 第 0 行 = 基因 0 在 3 个样本中的表达量
    [8.2,  7.8,  9.1],       # 第 1 行 = 基因 1
    [45.1, 42.3, 50.0],      # 第 2 行 = 基因 2
    [0.0,  0.5,  0.2],       # 第 3 行 = 基因 3
    [3.7,  4.1,  3.9],       # 第 4 行 = 基因 4
])
# 形状 (5, 3)：5 行（基因） × 3 列（样本）

# ── 快捷创建特殊数组 ──
np.zeros((100, 3))                   # 100×3 全零矩阵（初始化的表达矩阵）
np.ones((50,))                       # 长度 50 的全 1 向量
np.random.randn(100, 3)              # 100×3 标准正态分布随机数（模拟数据）
np.random.randn(100, 3) * 2 + 5     # 均值为 5, 标准差为 2 的 100×3 矩阵
np.arange(10)                        # [0,1,2,3,4,5,6,7,8,9] —— 类似 range 但返回数组
np.arange(0, 100, 10)               # [0,10,20,30,40,50,60,70,80,90]
```

### 6.3 数组属性和索引

```python
# ── 数组属性（查看数组的结构信息） ──
matrix.shape           # (5, 3) —— 形状：5 行 × 3 列
matrix.ndim            # 2 —— 维度数（几维数组）
matrix.size            # 15 —— 元素总数（5 × 3 = 15）
matrix.dtype           # dtype('float64') —— 元素的数据类型

# ── 索引和切片（和列表一样，但可以多维索引） ──
matrix[0, :]           # 第 0 行，所有列 → 基因 0 在所有样本的表达
# [行, 列]，: 表示"全部"
matrix[:, 1]           # 所有行，第 1 列 → 样本 1 中所有基因的表达
matrix[:3, 1:]         # 前 3 行，第 1 列及之后 → [0:3, 1:3]

# ── 布尔索引（超高频！） ──
# 找出在样本 0 中表达量 > 10 的基因
mask = matrix[:, 0] > 10
# mask = [True, False, True, False, False]
# 对每一行：样本 0 的表达 > 10？
high_expr_genes = matrix[mask, :]
# 只保留 mask 为 True 的行（即基因 0 和基因 2）

# 一行搞定
high_expr_genes = matrix[matrix[:, 0] > 10, :]
```

### 6.4 轴 (axis) —— 理解 NumPy 的核心

```python
# axis 是 NumPy 中最重要的概念之一
# 想象一个二维数组：
#         列0  列1  列2
# 行0 ->  [a,   b,   c]
# 行1 ->  [d,   e,   f]
# 行2 ->  [g,   h,   i]

# axis=0 → 沿行方向操作（"从上往下"）→ 对每一列独立操作 → 结果的行数变化
# axis=1 → 沿列方向操作（"从左往右"）→ 对每一行独立操作 → 结果的列数变化

matrix = np.array([
    [12.5, 15.2, 10.1],
    [8.2,  7.8,  9.1],
    [45.1, 42.3, 50.0],
])

matrix.sum(axis=0)     # [65.8, 65.3, 69.2]
# axis=0：每列求和 → 3 个样本的总表达量
# 12.5+8.2+45.1=65.8, 15.2+7.8+42.3=65.3, 10.1+9.1+50.0=69.2

matrix.mean(axis=1)    # [12.6, 8.37, 45.8]
# axis=1：每行求平均 → 3 个基因的平均表达量
# (12.5+15.2+10.1)/3=12.6, ...

# 生信中的使用：
# axis=0 → 对每个样本做操作（样本在列上）
# axis=1 → 对每个基因做操作（基因在行上）
```

### 6.5 向量化运算 —— NumPy 的灵魂

```python
# ═══════ 逐元素运算 ═══════
# 不需要写 for 循环！直接对数组做运算 = 对每个元素都做

# log2(FPKM + 1) 标准化 —— 生信中最常见的操作
log_matrix = np.log2(matrix + 1)
# matrix + 1：数组每个元素都加 1
# np.log2()：每个元素都取 log2
# 等价于对每个元素执行 x → log2(x+1)，但用 C 实现，极快

# Z-score 标准化（减均值，除标准差）
z_scores = (matrix - matrix.mean(axis=0)) / matrix.std(axis=0)
# matrix.mean(axis=0)：每个样本（每列）的均值，形状 (3,)
# matrix - mean：广播机制！(5,3) - (3,) → (5,3)，每列减去对应的均值
# / std：每个元素除以对应列的标准差

# ═══════ 统计运算 ═══════
mean_per_gene = matrix.mean(axis=1)         # 每个基因在所有样本中的平均表达
max_per_gene = matrix.max(axis=1)           # 每个基因的最大表达值
min_per_gene = matrix.min(axis=1)           # 每个基因的最小表达值
nonzero_per_gene = (matrix > 0).sum(axis=1) # 每个基因在多少样本中有表达
# (matrix > 0) 返回布尔数组，True=1, False=0，sum 就是数 True 的个数

np.corrcoef(matrix)          # 基因间的相关系数矩阵 (5×5)
np.percentile(matrix, 75, axis=0)  # 每个样本 75% 分位数

# ═══════ 数组重塑 ═══════
matrix.ravel()               # 多维 → 一维展平：(5,3) → (15,)
matrix.reshape(3, 5)         # 改变形状：(5,3) → (3,5)（总元素数必须相同！）
matrix.T                     # 转置：(5,3) → (3,5)，行列互换
```

### 6.6 for 循环 vs NumPy 向量化

```python
import time
import numpy as np

# 准备 10,000 个基因 × 100 个样本的模拟数据
genes, samples = 10000, 100
data = np.random.randn(genes, samples)

# ═══════ 方法一：Python for 循环 ═══════
start = time.time()
result = []
for i in range(genes):
    row = []
    for j in range(samples):
        row.append(np.log2(data[i, j] + 1))
    result.append(row)
loop_time = time.time() - start
print(f"For loop: {loop_time:.3f}s")       # 典型结果：~1.5s

# ═══════ 方法二：NumPy 向量化 ═══════
start = time.time()
result = np.log2(data + 1)                 # 一行！
numpy_time = time.time() - start
print(f"NumPy: {numpy_time:.3f}s")         # 典型结果：~0.002s

print(f"NumPy is {loop_time / numpy_time:.0f}x faster")
# 速度差距通常是 100~1000+ 倍！

# ═══════ 为什么差这么多？ ═══════
# for 循环：Python 每轮循环都要做类型检查、变量查找、函数调用...
# NumPy：底层 C 代码直接操作连续内存，没有 Python 开销
# 结论：在生信数据处理中，永远优先用 NumPy 向量化操作！
```

---

## 7. Pandas：生信数据的万能工具箱

> Pandas 的 **DataFrame** 是生信中最核心的数据容器。表达矩阵、临床信息表、基因注释表、差异分析结果表——全部用 DataFrame 存储。Scanpy 的 `adata.obs` 和 `adata.var` 本质上就是 DataFrame。

### 7.1 什么是 DataFrame？

```
DataFrame 就是一个"智能表格"：
- 有行名（index）和列名（columns）
- 每列可以是不同类型（int, float, str 混搭）
- 支持 SQL 式的查询、过滤、分组、合并
- 底层是 NumPy，速度快

和 Excel 的类比：
  Excel 工作表 ≈ DataFrame
  Excel 列     ≈ Series（DataFrame 中的一列）
  Excel 筛选   ≈ df[df["col"] > 5]
  Excel 数据透视表 ≈ df.groupby(...).mean()
```

```python
import pandas as pd
import numpy as np

# 创建 DataFrame：字典的键变成列名，值变成每列的数据
df = pd.DataFrame({
    "Sample_A": [12.5, 8.2, 45.1, 0.0],
    "Sample_B": [15.2, 7.8, 42.3, 0.5],
    "Sample_C": [10.1, 9.1, 50.0, 0.2],
}, index=["TP53", "BRCA1", "EGFR", "KRAS"])
# index 参数指定行名（基因名）

# 查看数据
df.shape           # (4, 3) —— 4 行 × 3 列
df.columns         # Index(['Sample_A', 'Sample_B', 'Sample_C'])
df.index           # Index(['TP53', 'BRCA1', 'EGFR', 'KRAS'])
df.head(2)         # 前两行（快速预览大数据集时常用）
df.describe()      # 每列的统计信息：count, mean, std, min, 25%, 50%, 75%, max

# ═══════ DataFrame vs Series ═══════
# DataFrame：二维表格（多列）
# Series：一维数组（单列），类似"带标签的 NumPy 数组"
sample_a = df["Sample_A"]              # Series 类型
type(sample_a)                         # pandas.core.series.Series
# Series 有 index，可以像字典一样用：sample_a["TP53"] → 12.5
```

### 7.2 `loc` vs `iloc` —— 最容易混淆的两个操作

```python
# ═══════ loc：用标签（名字）选择 ═══════
# loc = "label-based location"

df.loc["TP53"]                         # 选 TP53 这一行（返回 Series）
df.loc["TP53", "Sample_B"]             # 选 TP53 在 B 样本的值（返回标量
df.loc[["TP53", "EGFR"],               # 选多行多列
       ["Sample_A", "Sample_C"]]
df.loc["TP53":"EGFR", :]               # 注意：loc 的切片包含右边界！
# "TP53":"EGFR" 包含 TP53, BRCA1, EGFR 三行

# ═══════ iloc：用数字位置选择 ═══════
# iloc = "integer-based location"
# 和 NumPy/列表的索引完全一样：从 0 开始，左闭右开

df.iloc[0]                             # 第 0 行（TP53）
df.iloc[0, 1]                          # 第 0 行第 1 列（TP53, Sample_B）
df.iloc[:2, 1:]                        # 前 2 行，第 1 列及之后（右边界不包含！）

# ═══════ 什么时候用哪个？ ═══════
# 知道行/列的名字 → loc（如 loc["TP53"]）
# 知道行/列的位置 → iloc（如 iloc[0]）
# 不确定时优先用 loc（更安全，代码可读性更好）
```

### 7.3 布尔过滤 —— 生信最常用的操作

```python
# 筛选表达量 > 10 的基因（在所有样本中）
high_expr = df[df["Sample_A"] > 10]

# 多条件筛选：Sample_A > 5 且 Sample_B > 5
# 注意：每个条件必须用括号括起来！因为 & 的优先级高于 >
df_filtered = df[(df["Sample_A"] > 5) & (df["Sample_B"] > 5)]
# | → 或
# ~ → 非，如 ~(df["Sample_A"] > 10) 表示 "不大于 10"

# .query() 风格：把条件写成字符串，更接近自然语言
df.query("Sample_A > 10 and Sample_B > 5")

# .isin()：检查是否在某个列表中
cancer_genes = ["TP53", "BRCA1", "EGFR", "PTEN"]
cancer_expr = df[df.index.isin(cancer_genes)]
# df.index.isin(list)：检查行名是否在给定的列表中
```

### 7.4 数据处理

```python
# ── 创建新列 ──
df["Mean_Expr"] = df.mean(axis=1)
# axis=1：对每一行求均值（同 NumPy）

df["Log2_FC"] = np.log2(df["Sample_B"] + 1) - np.log2(df["Sample_A"] + 1)
# 计算 Sample_B 相对于 Sample_A 的 log2 差异倍数

# ── apply()：对每行/列应用自定义函数 ──
df["CV"] = df[["Sample_A", "Sample_B", "Sample_C"]].apply(
    lambda row: row.std() / row.mean(), axis=1
)
# apply() 的过程：
# 1. axis=1 → 把每一行传给 lambda
# 2. lambda row: row.std() / row.mean() → 计算该基因的变异系数
# 3. 所有结果组成新的一列

# ── 排序 ──
df.sort_values("Mean_Expr", ascending=False)  # 按平均表达降序
df.sort_index()                                # 按基因名字母顺序

# ── 分组聚合 ──
df["Group"] = ["Tumor", "Normal", "Tumor", "Normal"]
#               TP53     BRCA1    EGFR     KRAS
grouped = df.groupby("Group")["Sample_A"].mean()
# Tumor:  (12.5 + 45.1) / 2 = 28.8
# Normal: (8.2 + 0.0) / 2 = 4.1

# 相当于 SQL: SELECT AVG(Sample_A) FROM df GROUP BY Group

# ── 缺失值处理 ──
df.fillna(0)                      # 把 NaN 换成 0（NaN = "Not a Number"，缺失值）
# 生信中 NaN 通常表示"没有检测到表达" → 视为 0
df.dropna(axis=0, how="all")      # 删除全为 NaN 的行
```

### 7.5 合并数据

```python
# ═══════ merge：横向合并（类似 SQL JOIN） ═══════
expr_df = pd.read_csv("expression.tsv", sep="\t", index_col=0)
# 行=基因，列=样本ID

clinical_df = pd.read_csv("clinical.tsv", sep="\t")
# 列：sample_id, age, sex, stage, survival_months, ...

# 转置表达式矩阵：让样本变成行，基因变成列
expr_T = expr_df.T
# T = transpose, 100 样本 × 20000 基因

# 把临床信息合并进来
combined = expr_T.merge(
    clinical_df,
    left_index=True,        # 用 expr_T 的行名（样本ID）作为连接键
    right_on="sample_id"    # 用 clinical_df 的 sample_id 列作为连接键
)
# 结果：每行是一个样本，列包括所有基因 + 临床指标

# ═══════ concat：纵向拼接（把两个批次堆在一起） ═══════
batch1 = pd.read_csv("batch1.tsv", sep="\t")
batch2 = pd.read_csv("batch2.tsv", sep="\t")
all_data = pd.concat([batch1, batch2], axis=0)  # axis=0 → 纵向堆叠
# 前提：两个 DataFrame 的列名一致

all_columns = pd.concat([batch1, batch2], axis=1)  # axis=1 → 横向拼接
```

### 7.6 生信 Pandas 经典流水线

```python
# 一个完整的"原始表达矩阵 → 标准化后的干净矩阵"流程
# 每个步骤的含义都在注释中解释了

df = pd.read_csv("raw_counts.tsv", sep="\t", index_col=0)
# index_col=0 → 把第一列（基因名）当作行索引，不是数据列

# 步骤 1：去除不需要的基因类型
# ~ 是"取反"：~True = False
# str.startswith("MIR")：检测字符串是否以 "MIR" 开头
df = df[~df.index.str.startswith("MIR")]       # 去掉 microRNA
df = df[~df.index.str.startswith("LINC")]      # 去掉 lincRNA
# df.index.str 让你可以对行名使用字符串方法

# 步骤 2：过滤低表达基因
# (df > 1) → 和 1 比较，生成布尔 DataFrame（True/False）
# .sum(axis=1) → 每行有多少个 True（即该基因在多少样本中表达量 > 1）
# >= 3 → 至少在 3 个样本中达标
keep = (df > 1).sum(axis=1) >= 3
df = df[keep]  # 只保留满足条件的基因行

# 步骤 3：log2(CPM + 1) 标准化
# CPM = Counts Per Million，让不同测序深度的样本可比
lib_sizes = df.sum(axis=0)                    # 每个样本的总读段数
cpm = df.divide(lib_sizes, axis=1) * 1e6      # 除以文库大小，乘以 100 万
# divide(..., axis=1)：每行除以 lib_sizes（广播）
log_cpm = np.log2(cpm + 1)                    # log2 转换，+1 防止 log2(0)
# 为什么要 log 转换？
# 1. 表达量数据通常高度偏态（少数基因超高表达）
# 2. log 转换让数据更接近正态分布
# 3. 让差异倍数变得对称（fold change = 2 和 0.5 的 log2 分别是 1 和 -1）

log_cpm.to_csv("normalized_expression.tsv", sep="\t")
```

---

## 8. 数据可视化

### 8.1 理解 Matplotlib 的"画布"概念

```python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# ═══════ Figure 和 Axes ═══════
# Figure = 整张画布（可以放多张图）
# Axes = 画布上的一个子图（实际画图的地方）
# 一个 Figure 可以包含多个 Axes（subplots）

fig, ax = plt.subplots(figsize=(6, 5))
# figsize=(宽, 高) 单位是英寸
# 返回：fig = 画布对象, ax = 子图对象（我们主要在 ax 上画）

# ═══════ 散点图示例（PCA/t-SNE/UMAP 图就靠它） ═══════
# 模拟 PCA 的前两个主成分坐标
pca_x = np.random.randn(100) * 2   # 100 个样本的 PC1 坐标
pca_y = np.random.randn(100) * 2   # 100 个样本的 PC2 坐标
labels = np.random.choice(["Tumor", "Normal", "Metastasis"], 100)

fig, ax = plt.subplots(figsize=(6, 5))
for label in set(labels):                    # 按分组分别画，每组不同颜色
    mask = labels == label                   # 属于该组的样本
    ax.scatter(pca_x[mask], pca_y[mask],     # x 坐标, y 坐标
               label=label,                   # 图例中的名字
               s=10,                          # s = size：点的大小
               alpha=0.7)                     # alpha = 透明度 (0-1)
    # scatter() 每调用一次就在图上加一种颜色的点

ax.set_xlabel("PC1")                         # x 轴标签
ax.set_ylabel("PC2")                         # y 轴标签
ax.set_title("PCA of Samples")               # 图标题
ax.legend()                                  # 显示图例
plt.tight_layout()                           # 自动调整边距（避免标签被切掉）
plt.savefig("pca_plot.png", dpi=150, bbox_inches="tight")
# dpi=150 → 分辨率 150 dots per inch（论文级清晰度）
# bbox_inches="tight" → 裁掉多余白边
plt.show()                                   # 在 Jupyter 中显示图片
```

### 8.2 生信常用图表速查

```python
# ═══════ 图表类型 × 生信用途 对照 ═══════
# 散点图 (scatter)  → PCA/t-SNE/UMAP, 火山图
# 热图   (heatmap)  → 基因表达矩阵可视化
# 箱线图 (boxplot)  → 某基因在不同组间的表达对比
# 小提琴图(violin)  → 单细胞中基因表达分布（比箱线图更多信息）
# 柱状图 (bar)      → GO/KEGG 富集分析结果
# 直方图 (hist)     → P 值分布，表达量分布
# 折线图 (line)     → 时序表达变化，生存曲线

# ── 热图 ──
data = np.random.randn(20, 10)               # 20 基因 × 10 样本
sns.heatmap(data, cmap="RdBu_r", center=0,   # cmap=色阶, center=中心值
            xticklabels=False, yticklabels=False)
# cmap="RdBu_r"：红-白-蓝色阶，红=高, 白=中, 蓝=低（_r=反转）
# center=0：以 0 为颜色中心，适合展示 log2FC（上调/下调）


# ── 箱线图：看一个基因在分组间的表达差异 ──
sns.boxplot(data=expr_df, x="Group", y="TP53")
# 箱线图解读：
# - 中间的线 = 中位数
# - 箱子上下边 = 第 25 和 75 百分位数
# - 须线 = 1.5 倍四分位距内的范围
# - 须线外的点 = 离群值（outlier）

# ── 小提琴图：单细胞分析中比箱线图更常用 ──
sns.violinplot(data=expr_df, x="CellType", y="GeneX",
               scale="width", inner="quartile")
# 小提琴图的形状 = 数据分布的密度估计
# 越宽的地方 = 该表达水平的细胞越多

# ── 火山图（生信中的必修课） ──
log2fc = np.random.randn(5000)               # 5000 个基因的 log2FC
pval = np.random.exponential(0.05, 5000)     # 对应的 P 值
sig = (pval < 0.05) & (abs(log2fc) > 1)     # 显著差异的基因

fig, ax = plt.subplots(figsize=(8, 6))
# 先画不显著的（灰色背景）
ax.scatter(log2fc[~sig], -np.log10(pval[~sig]),
           c="grey", s=2, alpha=0.5, label="NS")
# ~sig = 取反，即不显著的；-log10(p)：P 越小 Y 越大（越显著越在顶部）
# 再画显著的（红色前景）
ax.scatter(log2fc[sig], -np.log10(pval[sig]),
           c="red", s=3, alpha=0.7, label="Significant")
# 画阈值线
ax.axhline(-np.log10(0.05), ls="--", color="grey", alpha=0.5)  # P=0.05 线
ax.axvline(-1, ls="--", color="grey", alpha=0.5)               # FC=-1 线
ax.axvline(1, ls="--", color="grey", alpha=0.5)                # FC=+1 线
ax.set_xlabel("log2 Fold Change")
ax.set_ylabel("-log10(p-value)")
ax.set_title("Volcano Plot: Tumor vs Normal")
ax.legend()
```

### 8.3 生信可视化 6 条准则

```
1. 每张图必须有三要素：x 轴标签、y 轴标签、标题
2. 热图用 RdBu_r（红-白-蓝），以 0 为中心：红=上调，蓝=下调
3. 全文颜色编码保持一致（Tumor=red, Normal=blue，不要换）
4. 论文用矢量格式 PDF/SVG，日常用 PNG
5. 不要用 jet 色阶，用 viridis / RdBu_r / Set2 等感知均匀的色阶
6. t-SNE/UMAP 图中，簇间距离没有定量意义——只看哪些细胞在一起
```

---

## 9. 统计学与假设检验

### 9.1 核心概念速览

```python
from scipy import stats
import numpy as np

# ═══════ 三个最基础的统计概念 ═══════
data = np.random.randn(100) + 5             # 模拟 100 个基因的表达量

# 均值 (mean)：数据的"重心"
mean = np.mean(data)

# 标准差 (std)：数据的"离散程度"
# 大 = 基因表达在不同样本间波动大；小 = 表达稳定
std = np.std(data, ddof=1)
# ddof=1 → 样本标准差（分母用 n-1），生信中标准做法
# ddof=0 → 总体标准差（分母用 n）

# 标准误 (SEM)：均值估计的"精度"
# SEM = std / sqrt(n)，样本越多，SEM 越小
sem = stats.sem(data)

# ═══════ P 值是什么？（生信中最关键的理解） ═══════
# P 值 = 在"无效假设为真"的前提下，观察到当前结果（或更极端）的概率
#
# 无效假设 H0：两组没有差异
# P = 0.03 → 如果两组真的没有差异，仅有 3% 的概率会看到现在这样的结果
#            → 通常我们认为 P < 0.05 时拒绝 H0，认为"有显著差异"
#
# ⚠️ P 值不告诉你差异有多大！差异大小看 fold change。
#    一个小差异在大样本中可能 P 值极小，但它仍然没什么生物学意义。
```

### 9.2 生信中的常用检验

```python
# ═══════ t 检验：两组比较（最常用） ═══════
# 假设：数据大致正态分布
tumor_expr = np.random.randn(30) * 2 + 8    # 30 个肿瘤样本, 均值≈8
normal_expr = np.random.randn(30) * 2 + 5   # 30 个正常样本, 均值≈5
# np.random.randn(30) → 30 个标准正态随机数
# * 2 → 标准差变为 2
# + 8 → 均值变为 8

t_stat, p_val = stats.ttest_ind(tumor_expr, normal_expr)
# ttest_ind → independent t-test（独立样本 t 检验）
# t_stat：t 统计量，越大说明两组差异越大
# p_val：P 值

# ═══════ Mann-Whitney U 检验（非参数检验） ═══════
# 不假设正态分布，基于秩（rank）进行比较
# 在单细胞差异分析中更常用，因为单细胞数据高度非正态
u_stat, p_val = stats.mannwhitneyu(tumor_expr, normal_expr,
                                    alternative="two-sided")

# ═══════ 相关性分析 ═══════
# Pearson 相关：线性相关（要求正态分布）
r, p = stats.pearsonr(gene_a_expr, gene_b_expr)
# r = 1：完全正相关；r = -1：完全负相关；r = 0：不相关

# Spearman 秩相关：单调相关（不要求正态，更稳健）
rho, p = stats.spearmanr(gene_a_expr, gene_b_expr)
# Spearman 基于值的排名而非原始值，不受离群值影响
```

### 9.3 多重检验校正 —— 生信必须理解的概念

```python
# ═══════ 为什么需要校正？ ═══════
# 如果你对 20,000 个基因各做一次 t 检验（alpha=0.05），
# 即使所有基因都真的没有差异，你也期望得到：
#   20,000 × 0.05 = 1,000 个"显著"的假阳性！
# 这不是方法的问题，是"多次检验"本身的逻辑必然。

from statsmodels.stats.multitest import multipletests

p_values = [0.001, 0.04, 0.0001, 0.5, 0.02, 0.1, 0.8]

# Benjamini-Hochberg (FDR) 校正 —— 生信标准做法
reject, padj, _, _ = multipletests(p_values, method="fdr_bh")
# reject：bool 列表，校正后是否仍显著
# padj：校正后的 P 值
# 忽略的第 3、4 个返回值

for p, adj, r in zip(p_values, padj, reject):
    print(f"raw p={p:.4f} → corrected p={adj:.4f}, significant={r}")

# 输出解读：
# raw p=0.0010 → corrected p=0.0035, significant=True   ← 仍然显著
# raw p=0.0400 → corrected p=0.0700, significant=False  ← 校正后不显著了！
# raw p=0.5000 → corrected p=0.7000, significant=False  ← 本来就不显著

# ═══════ 三种校正方法对比 ═══════
# Bonferroni：最严格，P_adj = P × N（N=检验次数）
#   优点：假阳性极少；缺点：太严格，丢掉太多真阳性
# Benjamini-Hochberg (BH)：控制 False Discovery Rate ≤ 0.05
#   含义：在"显著"的结果中，最多 5% 是假的
#   **这是生信中默认的选择**
# Benjamini-Yekutieli：BH 的改进版，考虑基因间的相关性
```

### 9.4 批量差异分析的简化版实现

```python
# ⚠️ 真实分析请用 DESeq2 / edgeR / MAST
# 这里是理解原理的简化版 Python 实现

results = []
for gene in expr_df.index:                   # 对每个基因
    tumor_vals = expr_df.loc[gene, tumor_samples].astype(float)
    normal_vals = expr_df.loc[gene, normal_samples].astype(float)

    # 跳过在所有样本中都不表达的基因
    if tumor_vals.mean() < 1 and normal_vals.mean() < 1:
        continue

    t_stat, p_val = stats.ttest_ind(tumor_vals, normal_vals)
    # log2(fold change)：+0.01 防止除以 0
    log2fc = np.log2(tumor_vals.mean() + 0.01) - np.log2(normal_vals.mean() + 0.01)
    results.append({"gene": gene, "log2FC": log2fc, "p_value": p_val})

results_df = pd.DataFrame(results)
# 多重检验校正
results_df["padj"] = multipletests(results_df["p_value"],
                                    method="fdr_bh")[1]
# 筛选显著差异基因
sig_genes = results_df[(results_df["padj"] < 0.05) &
                       (abs(results_df["log2FC"]) > 1)]
# padj < 0.05：统计显著
# |log2FC| > 1：生物学显著（差异倍数 > 2 倍或 < 1/2）
```

---

## 10. Biopython 精要

> Biopython 是 Python 的生信工具包。你不需要学它的全部——掌握 SeqIO 和 Seq 就覆盖了 80% 的使用场景。

### 10.1 Seq对象

```python
from Bio.Seq import Seq
from Bio import SeqIO

# Seq 对象 = 增强版字符串
# 它和普通字符串一样可以切片、索引、len()
# 但它还提供了生信专属方法

seq = Seq("ATGGAGGAGCCGCAG")

# 这些方法和我们在第 5 章手写的一样，但 Biopython 已经帮你写好了
seq.complement()             # Seq("TACCTCCTCGGCGTC") —— 互补链
seq.reverse_complement()     # Seq("CTGCGGCTCCTCCAT") —— 反向互补
seq.transcribe()             # Seq("AUGGAGGAGCCGCAG") —— DNA → RNA
seq.translate()              # Seq("MEEPQ")           —— DNA → 蛋白质
# translate() 内部已经内置了标准密码子表！

# Seq 对象可以直接和字符串比较
seq[:3] == "ATG"             # True

# 转换成普通字符串
str(seq)                     # "ATGGAGGAGCCGCAG"
```

### 10.2 SeqIO —— 读/写序列文件

```python
# ═══════ 读取 FASTA ═══════
# SeqIO.parse() 返回一个迭代器，每次给一个 SeqRecord 对象
for record in SeqIO.parse("genome.fasta", "fasta"):
    # record 是一个 SeqRecord，包含序列 + 元信息
    print(f"ID: {record.id}")              # "TP53"
    print(f"完整描述: {record.description}")  # "TP53 transcript variant 1"
    print(f"序列长度: {len(record.seq)}")    # 序列的碱基数
    print(f"序列前 50 bp: {record.seq[:50]}")  # record.seq 是 Seq 对象
    print(f"GC 含量: {gc_content(str(record.seq)):.1%}")

# 对比：手写 FASTA 解析器（第 4 章）需要约 20 行
#       用 SeqIO.parse() 只需要 2 行
# 结论：读取标准格式的序列文件，优先用 Biopython

# ═══════ 筛选并写入 ═══════
long_seqs = []
for record in SeqIO.parse("transcripts.fasta", "fasta"):
    if len(record.seq) > 1000:
        long_seqs.append(record)

SeqIO.write(long_seqs, "long_transcripts.fasta", "fasta")
# SeqIO.write(记录列表, 输出文件名, 格式)

# ═══════ 读 FASTQ ═══════
for record in SeqIO.parse("reads.fastq", "fastq"):
    print(f"读段 ID: {record.id}")
    print(f"序列: {record.seq}")
    print(f"质量分数: {record.letter_annotations['phred_quality'][:10]}")
```

### 10.3 NCBI 数据下载

```python
from Bio import Entrez, Medline

# Entrez 是 NCBI 的 API 接口
# 注意：必须设置邮箱，NCBI 要求告知谁在查询
Entrez.email = "your_email@example.com"

# ═══════ 下载序列 ═══════
handle = Entrez.efetch(db="nucleotide",         # 数据库=核苷酸
                       id="NM_000546",          # RefSeq ID (TP53)
                       rettype="fasta")         # 返回 FASTA 格式
record = SeqIO.read(handle, "fasta")            # SeqIO.read() 读单条
# SeqIO.read vs SeqIO.parse：
#   read()：文件中只有一条序列时使用（如果有多条会报错）
#   parse()：文件中有多条序列时使用（返回迭代器）
handle.close()                                  # 记得关闭！

# ═══════ 搜索 PubMed ═══════
handle = Entrez.esearch(db="pubmed",
                        term="TP53 cancer",    # 搜索词
                        retmax=10)             # 最多返回 10 条
record = Entrez.read(handle)
handle.close()
id_list = record["IdList"]                     # PubMed ID 列表

# ═══════ 获取文献摘要 ═══════
handle = Entrez.efetch(db="pubmed",
                       id=",".join(id_list),   # 用逗号连接多个 ID
                       rettype="medline",
                       retmode="text")
records = Medline.parse(handle)
for rec in records:
    print(f"标题: {rec.get('TI', 'N/A')}")
    print(f"摘要: {rec.get('AB', 'N/A')[:200]}...")
    print(f"PMID: {rec.get('PMID')}")
    print("---")
```

---

## 11. 机器学习入门：scikit-learn 在生信中的应用

> 这部分讲的是"怎么写代码跑 ML"，算法理论请参考桌面的《生信机器学习深度学习入门指南》。

### 11.1 scikit-learn 的统一 API 模式

**所有 sklearn 模型都遵循完全相同的 4 步模式**。记住这一个模式，所有模型都会用：

```python
# 模式：准备 → 划分 → 训练 → 评估
#       prep → split → fit → predict

from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report, roc_auc_score

# ═══════ 第 1 步：准备数据 ═══════
# sklearn 的要求：X 必须是二维数组 (样本数 × 特征数)，y 是一维数组
# 我们的表达矩阵通常是 基因 × 样本，需要转置！
X = expr_df.T.values          # 转置 → 变成 样本数 × 基因数
y = sample_labels             # 如 [1, 1, 0, 0, 1, 1, 0, ...]
# X 形状：(100, 20000) → 100 个样本，20000 个基因特征
# y 形状：(100,)      → 100 个样本的标签

# ═══════ 第 2 步：划分训练集和测试集 ═══════
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,           # 20% 用来测试，80% 用来训练
    random_state=42,         # 随机种子：固定随机性，保证每次运行结果一致
    stratify=y               # 保持训练和测试集中各类别的比例相同
)
# random_state 随便设一个数（42 是传统），目的是结果可复现
# stratify 很重要：如果正负样本不均（如 70 个病人 vs 30 个对照），
# 不设 stratify 可能训练集全是病人、测试集全是对照

# ═══════ 第 3 步：训练模型 ═══════
model = RandomForestClassifier(n_estimators=100, random_state=42)
# n_estimators=100 → 用 100 棵树投票
model.fit(X_train, y_train)
# fit() 是 sklearn 的核心——模型从这里开始"学习"

# ═══════ 第 4 步：预测 + 评估 ═══════
y_pred = model.predict(X_test)           # 预测类别（0 或 1）
y_prob = model.predict_proba(X_test)[:, 1]  # 预测概率
# predict_proba 返回 (n_samples, 2) 的数组：第 1 列是 P(0), 第 2 列是 P(1)
# [:, 1] 取第 2 列：每个样本属于类别 1 的概率

# 评估指标
print(f"Accuracy: {accuracy_score(y_test, y_pred):.3f}")
# 准确率 = 预测对的 / 总数；不太适合类别不均衡的数据

print(f"AUC: {roc_auc_score(y_test, y_prob):.3f}")
# AUC = Area Under the ROC Curve
# 衡量模型区分正负样本的能力，0.5=随机, 1.0=完美, >0.8=不错

print(classification_report(y_test, y_pred))
# 精确率 (Precision)：预测为正的样本中，真的为正的比例
# 召回率 (Recall)：真的为正的样本中，被正确找出的比例
# F1-score：精确率和召回率的调和平均
```

### 11.2 常用模型速查

```python
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.ensemble import RandomForestClassifier
from xgboost import XGBClassifier
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# ── 逻辑回归：可解释性最强 ──
# 每个基因有一个权重，权重正负 = 该基因是危险还是保护因素
lr = LogisticRegression(penalty="l1", solver="saga", C=1.0, max_iter=1000)
# penalty="l1"：L1 正则化 = 自动做特征选择（把不重要基因的权重压到 0）
# C=1.0：正则化强度（越小正则化越强）

# ── 随机森林：生信万金油 ──
rf = RandomForestClassifier(n_estimators=500,    # 树的数量（越多越稳定但越慢）
                            max_depth=10,        # 每棵树的最大深度（防止过拟合）
                            random_state=42)

# ── SVM：适合小样本 ──
svm = SVC(kernel="rbf", probability=True, C=1.0)
# kernel="rbf"：用 RBF 核处理非线性关系
# probability=True：启用概率输出（需要的话）

# ── XGBoost：表格数据王者 ──
xgb = XGBClassifier(n_estimators=200, max_depth=6, learning_rate=0.05)

# ── PCA：降维 ──
pca = PCA(n_components=50)             # 降到 50 维
X_pca = pca.fit_transform(X)           # 同时完成 fit 和 transform

# ── K-Means：聚类 ──
kmeans = KMeans(n_clusters=3, random_state=42)
clusters = kmeans.fit_predict(X)       # 返回每个样本属于哪个簇（0,1,2）

# ── StandardScaler：标准化 ──
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)     # 每列变成均值为 0，标准差为 1
```

### 11.3 特征选择和交叉验证

```python
from sklearn.feature_selection import SelectKBest, f_classif
from sklearn.model_selection import cross_val_score, StratifiedKFold

# ── 特征选择：从 20000 个基因中选出 top 100 ──
selector = SelectKBest(f_classif, k=100)
# f_classif = ANOVA F 值（方差分析）：检测每个基因和标签之间的线性关系
X_selected = selector.fit_transform(X, y)
selected_gene_indices = selector.get_support()
# get_support() 返回 bool 数组，标记哪些特征被选中
selected_gene_names = gene_names[selected_gene_indices]

# ── 交叉验证：更可靠的模型评估 ──
# 不要把"用一次 train_test_split 得到的测试集准确率"当真
# 那可能只是你运气好分到了好的测试集
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
# 把数据分成 5 份，每次用 4 份训练、1 份测试，循环 5 次
scores = cross_val_score(model, X, y, cv=cv, scoring="roc_auc")
print(f"AUC: {scores.mean():.3f} ± {scores.std():.3f}")
# 报告的是 5 次的均值 ± 标准差，比单次更可靠
```

---

## 12. 单细胞分析：Scanpy 入门

### 12.1 AnnData —— 单细胞数据的标准容器

```python
import scanpy as sc

# ═══════ AnnData 的五个组成部分 ═══════
# AnnData 是围绕"表达矩阵 X"的一整套数据集合
#
# adata.X      → 表达矩阵 (细胞数 × 基因数)，NumPy 数组或稀疏矩阵
# adata.obs    → 细胞元数据 DataFrame（行=细胞）
#                每一列是细胞的一个属性：细胞类型、样本来源、UMAP 坐标...
# adata.var    → 基因元数据 DataFrame（行=基因）
#                每一列是基因的一个属性：高变异标签、线粒体标签...
# adata.obsm   → 降维坐标字典
#                如 adata.obsm["X_umap"] = UMAP 坐标数组
# adata.uns    → 非结构化字典
#                存颜色映射、marker 基因排序结果等杂项

adata = sc.datasets.pbmc3k()          # 2700 个 PBMC × 13714 个基因
print(adata)                           # 会显示 AnnData 的概要信息
print(adata.obs.columns)               # 查看细胞的注释列名
# AnnData 支持类似 Pandas 的切片操作
# adata[:100, :] → 前 100 个细胞
# adata[:, adata.var_names.str.startswith("CD")] → 所有以 CD 开头的基因
```

### 12.2 标准单细胞分析流水线（逐行解释）

```python
import scanpy as sc

# ── 读入 10X Genomics 格式的数据 ──
adata = sc.read_10x_h5("filtered_feature_bc_matrix.h5")
# 10X Genomics 输出一个 HDF5 文件，包含了所有细胞的 UMI 计数矩阵
adata.var_names_make_unique()
# 确保基因名唯一（有些基因可能在不同染色体上有拷贝）

# ═══════ 1. 质量控制 (QC) ═══════
# 标记线粒体基因（高线粒体比例 = 细胞损伤/死亡）
adata.var["mt"] = adata.var_names.str.startswith("MT-")
# var_names 是基因名，str.startswith("MT-") 判断是否以 MT- 开头
# 返回 True/False 数组，赋值给 var 表的新列 "mt"

# 计算每个细胞的线粒体比例
# adata[:, adata.var["mt"]]：选所有细胞 + 线粒体基因 → 子集
# .X.toarray()：稀疏矩阵 → 密集数组（因为后续要 sum）
# .sum(1)：每行（每个细胞）求和 → 该细胞的总线粒体表达量
adata.obs["pct_counts_mt"] = (
    adata[:, adata.var["mt"]].X.toarray().sum(axis=1) /
    adata.X.toarray().sum(axis=1) * 100
)
# 简化方式（推荐）：
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"],
                           percent_top=None, inplace=True)

# 过滤
sc.pp.filter_cells(adata, min_genes=200)       # 至少表达 200 个基因（去除空液滴）
sc.pp.filter_genes(adata, min_cells=3)         # 至少在 3 个细胞中表达（去除噪音基因）
adata = adata[adata.obs["pct_counts_mt"] < 20, :]  # 线粒体 < 20%（去除死细胞）

# ═══════ 2. 标准化 ═══════
# 保存原始计数（后续差异分析可能用到）
adata.raw = adata.copy()
# 每细胞归一化：把每个细胞的总表达量缩放到 10,000
sc.pp.normalize_total(adata, target_sum=1e4)
# log(x+1) 转换
sc.pp.log1p(adata)

# ═══════ 3. 高变异基因筛选 ═══════
# 选 2000 个"变化最大"的基因，用于后续降维和聚类
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
# 为什么只选 2000 个？
# - 大多数基因在所有细胞中表达稳定，不提供分组信息
# - 只保留"高变异"基因 = 保留能区分细胞类型的基因
# - 同时大幅减少计算量
adata = adata[:, adata.var.highly_variable]  # 只保留高变异基因

# ═══════ 4. PCA 降维 ═══════
sc.tl.pca(adata, n_comps=50, svd_solver="arpack")
# 2000 维 → 50 维，保留最主要的变异信息
# 肘部图：看每个主成分解释的方差，前多少 PC 累积解释 > 80%

# ═══════ 5. 构建邻居图 + UMAP ═══════
sc.pp.neighbors(adata, n_pcs=30, n_neighbors=15)
# 在 PCA 空间中找每个细胞的 15 个最近邻居
# n_pcs=30 → 只用前 30 个主成分（后面的主成分主要是噪音）

sc.tl.umap(adata, min_dist=0.3)
# UMAP 把高维邻居关系"画"到 2D 平面上
# min_dist=0.3 → 点之间的最小距离（越小簇越紧）

# ═══════ 6. 聚类 (Leiden 算法) ═══════
sc.tl.leiden(adata, resolution=0.5)
# resolution：分辨率参数
#   0.1 → 非常粗（几个大簇）
#   1.0 → 非常细（很多小簇）
#   0.5 → 这是一个中庸的起点，之后可以调整
# 聚类结果存在 adata.obs["leiden"]

# ═══════ 7. 找 marker 基因 ═══════
sc.tl.rank_genes_groups(adata, groupby="leiden", method="wilcoxon")
# 对每个 cluster，找出它在所有 cluster 中最高表达的基因
# method="wilcoxon" → Wilcoxon 秩和检验（非参数）

# ═══════ 8. 细胞类型注释 ═══════
# 这个步骤需要生物学知识！根据 marker 基因判断每个 cluster 是什么细胞类型
marker_dict = {
    "T cells":    ["CD3D", "CD3E", "CD8A", "CD4"],
    "B cells":    ["CD79A", "CD79B", "MS4A1"],
    "Monocytes":  ["CD14", "LYZ", "FCGR3A"],
    "NK cells":   ["NKG7", "GNLY"],
}

# Dotplot：每一行是一种细胞类型特征基因，每一列是一个 cluster
# 点的大小 = 表达比例，点的颜色深浅 = 平均表达水平
sc.pl.dotplot(adata, var_names=marker_dict, groupby="leiden")

# 根据 dotplot 结果人工标注
cluster_to_celltype = {
    "0": "CD4+ T cells",
    "1": "CD14+ Monocytes",
    "2": "B cells",
    # ... 需要根据实际 dotplot 来填！
}
adata.obs["cell_type"] = adata.obs["leiden"].map(cluster_to_celltype)
# .map()：用字典映射，把 cluster 编号转换为细胞类型名称

# 最后的可视化
sc.pl.umap(adata, color="cell_type", legend_loc="right margin")
```

### 12.3 批次校正（scVI）

```python
import scvi

# scVI = single-cell Variational Inference
# 用深度生成模型（VAE）学习细胞的潜在表示，同时校正批次效应

# 第一步：告诉 scVI 哪一列是批次标签
scvi.model.SCVI.setup_anndata(adata, batch_key="batch")

# 第二步：创建并训练模型
model = scvi.model.SCVI(adata,
                        n_layers=2,          # 神经网络的隐藏层数
                        n_latent=30,         # 潜在空间维数
                        gene_likelihood="nb") # 用负二项分布建模表达量
model.train()

# 第三步：提取潜在表示
adata.obsm["X_scVI"] = model.get_latent_representation()

# 用 scVI 潜在空间替代 PCA 做后续分析
sc.pp.neighbors(adata, use_rep="X_scVI")  # 在 scVI 空间而非 PCA 空间找邻居
sc.tl.umap(adata)
sc.tl.leiden(adata)
```

---

## 13. 综合实战：三条完整分析流水线

> 这三条流水线涵盖了生信 Python 分析最常见的工作模式。每条流水线从头到尾读一遍，理解每一行在干什么。

### 13.1 流水线一：TCGA 差异表达分析 + 火山图

```python
import pandas as pd
import numpy as np
from scipy import stats
from statsmodels.stats.multitest import multipletests
import matplotlib.pyplot as plt

# ── 1. 读取数据 ──
expr = pd.read_csv("tcga_brca_expr.tsv", sep="\t", index_col=0)
# 行=基因, 列=样本, 值=log2(FPKM+1)
clinical = pd.read_csv("tcga_brca_clinical.tsv", sep="\t")

# ── 2. 根据临床信息分出 Tumor 和 Normal 组 ──
tumor_samples = clinical[clinical["type"] == "Tumor"]["sample_id"].tolist()
normal_samples = clinical[clinical["type"] == "Normal"]["sample_id"].tolist()
# .tolist()：把 Pandas Series 转成 Python 列表

# ── 3. 对每个基因循环做 t 检验 ──
results = []
for gene in expr.index:            # 对每个基因
    tumor = expr.loc[gene, tumor_samples].astype(float)
    normal = expr.loc[gene, normal_samples].astype(float)

    # 跳过在所有样本都不表达的基因（节省计算时间 + 减少噪音）
    if tumor.mean() < 1 and normal.mean() < 1:
        continue

    t_stat, p_val = stats.ttest_ind(tumor, normal)
    log2fc = np.log2(tumor.mean() + 0.01) - np.log2(normal.mean() + 0.01)
    # + 0.01：防止 log2(0) 出现 -inf

    results.append({"gene": gene, "log2FC": log2fc, "p_value": p_val})

deg = pd.DataFrame(results)
deg["padj"] = multipletests(deg["p_value"], method="fdr_bh")[1]
deg = deg.sort_values("padj")      # 按校正 P 值从小到大排

# ── 4. 画火山图 ──
deg["sig"] = (deg["padj"] < 0.05) & (abs(deg["log2FC"]) > 1)

fig, ax = plt.subplots(figsize=(8, 6))

# 不显著的
ax.scatter(deg[~deg["sig"]]["log2FC"], -np.log10(deg[~deg["sig"]]["p_value"]),
           c="grey", s=1, alpha=0.3)
# 上调的（log2FC > 0）
up = deg["sig"] & (deg["log2FC"] > 0)
ax.scatter(deg[up]["log2FC"], -np.log10(deg[up]["p_value"]),
           c="red", s=2, alpha=0.7, label=f"Up ({up.sum()})")
# 下调的（log2FC < 0）
down = deg["sig"] & (deg["log2FC"] < 0)
ax.scatter(deg[down]["log2FC"], -np.log10(deg[down]["p_value"]),
           c="blue", s=2, alpha=0.7, label=f"Down ({down.sum()})")

# 阈值线
ax.axhline(-np.log10(0.05), ls="--", color="grey", alpha=0.5)
ax.axvline(-1, ls="--", color="grey", alpha=0.5)
ax.axvline(1, ls="--", color="grey", alpha=0.5)
ax.set_xlabel("log2(Fold Change)")
ax.set_ylabel("-log10(p-value)")
ax.set_title("Tumor vs Normal: Differential Expression")
ax.legend()
plt.tight_layout()
plt.savefig("volcano_plot.png", dpi=150)

# ── 5. 保存结果 ──
deg[deg["sig"]].to_csv("significant_degs.tsv", sep="\t", index=False)
print(f"Found {deg['sig'].sum()} significant DEGs "
      f"({up.sum()} up, {down.sum()} down)")
```

### 13.2 流水线二：随机森林分类癌症亚型

```python
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
from sklearn.feature_selection import SelectKBest, f_classif

# ── 1. 准备数据 ──
expr = pd.read_csv("brca_subtype_expr.tsv", sep="\t", index_col=0)
labels = pd.read_csv("brca_subtype_labels.tsv", sep="\t")

X = expr.T.values     # 转置：样本 × 基因
y = labels["subtype"].values   # Basal, LumA, LumB, Her2

# ── 2. 特征选择：筛到 500 个基因 ──
selector = SelectKBest(f_classif, k=500)
X_selected = selector.fit_transform(X, y)
selected_genes = expr.index[selector.get_support()]
print(f"Selected {len(selected_genes)} genes")

# ── 3. 划分训练/测试集 ──
X_train, X_test, y_train, y_test = train_test_split(
    X_selected, y, test_size=0.2, random_state=42, stratify=y
)

# ── 4. 训练 ──
rf = RandomForestClassifier(n_estimators=500, max_depth=10,
                            min_samples_leaf=5, random_state=42)
# min_samples_leaf=5：叶子节点最少 5 个样本（防止过拟合）
rf.fit(X_train, y_train)

# ── 5. 评估 ──
print(f"Train accuracy: {rf.score(X_train, y_train):.3f}")
print(f"Test accuracy:  {rf.score(X_test, y_test):.3f}")
# 如果 train >> test：过拟合（模型太复杂，背题了）
# 如果 train ≈ test 且都低：欠拟合（模型太简单，没学会）

print("\n" + classification_report(y_test, rf.predict(X_test)))

# ── 6. 找出最重要的基因 ──
feature_importance = pd.DataFrame({
    "gene": selected_genes,
    "importance": rf.feature_importances_
}).sort_values("importance", ascending=False)

print("\nTop 10 discriminative genes:")
print(feature_importance.head(10))
# 这些基因很可能和乳腺癌亚型生物学相关，值得进一步富集分析
```

### 13.3 流水线三：单细胞从头分析

```python
import scanpy as sc
import matplotlib.pyplot as plt

sc.settings.verbosity = 1       # 减少日志输出（0=静默, 3=详细）
sc.settings.set_figure_params(dpi=100, facecolor="white")

# ── 读入 ──
adata = sc.read_10x_h5("filtered_feature_bc_matrix.h5")
adata.var_names_make_unique()

# ── QC ──
adata.var["mt"] = adata.var_names.str.startswith("MT-")
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"], percent_top=None, inplace=True)
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
adata = adata[adata.obs["pct_counts_mt"] < 20, :]

# ── 标准化 + 高变异基因 + 降维 + 聚类 ──
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
sc.tl.pca(adata, n_comps=50)
sc.pp.neighbors(adata, n_pcs=30)
sc.tl.umap(adata)
sc.tl.leiden(adata, resolution=0.6)

# ── 保存 QC 图 ──
sc.pl.umap(adata, color=["leiden", "n_genes_by_counts", "pct_counts_mt"],
           ncols=2, show=False)
plt.savefig("sc_qc_umap.png", dpi=150, bbox_inches="tight")

# ── 找 marker + 保存结果 ──
sc.tl.rank_genes_groups(adata, "leiden", method="wilcoxon")
top_markers = pd.DataFrame(adata.uns["rank_genes_groups"]["names"]).iloc[:10]
print("Top markers per cluster:\n", top_markers)

adata.write("processed_scdata.h5ad", compression="gzip")
# h5ad = AnnData 的 HDF5 磁盘格式，压缩后存所有数据
```

---

## 14. 速查卡片合集

以下是可以直接打印贴在显示器旁边的速查卡片。

### 14.1 Python 语法速查卡

```
╔══════════════════════════════════════════════════════╗
║               Python 生信常用语法                     ║
╠══════════════════════════════════════════════════════╣
║ for gene in gene_list:        遍历基因列表          ║
║ for k, v in d.items():        遍历字典的键和值      ║
║ for i, x in enumerate(lst):   带索引遍历            ║
║ for a, b in zip(l1, l2):      并行遍历两个列表      ║
║ [f(x) for x in lst]           列表推导式            ║
║ [x for x in lst if cond]      带条件的列表推导      ║
║ {k: v for k,v in d.items()}   字典推导式            ║
║ with open(f) as fh:           安全文件读写（自动关） ║
║ line.strip()                  去除行末换行符        ║
║ line.split("\t")              Tab 分隔解析          ║
║ ",".join(lst)                 将列表拼成字符串      ║
║ seq.count("CG")               子串计数              ║
║ seq.find("ATG")               子串查找(-1=不存在)   ║
║ seq.replace("T","U")          DNA→RNA 转录          ║
║ re.findall(r"GO:\d+", text)   正则提取              ║
║ try: ... except X as e:       异常处理              ║
║ x if cond else y              三元表达式            ║
╚══════════════════════════════════════════════════════╝
```

### 14.2 数据结构选择速查

```
╔══════════╤══════════════════════════════════════════╗
║ 结构     │ 用途              │ 典型生信场景         ║
╠══════════╪══════════════════════════════════════════╣
║ list     │ 有序可变集合      │ 基因列表、样本列表   ║
║ tuple    │ 不可变数据打包    │ 基因组坐标、函数返多值║
║ dict     │ 键值映射          │ 基因名→表达量、ID映射║
║ set      │ 去重+集合运算     │ 差异基因交集/差集    ║
║ def dict │ 自动初始化默认值  │ 计时（int）/分组(list)║
╚══════════╧══════════════════════════════════════════╝
```

### 14.3 Pandas 速查卡

```
╔══════════════════════════════════════════════════════╗
║                Pandas 高频操作                        ║
╠══════════════════════════════════════════════════════╣
║ pd.read_csv(f, sep="\t", index_col=0)  读 TSV       ║
║ df.head() / .shape / .columns / .index  基本查看     ║
║ df["col"] / df[["c1","c2"]]            选择列（Series/DF）║
║ df.loc["row"] / df.iloc[0]             按标签/位置选行 ║
║ df[df["col"] > 5]                      布尔过滤      ║
║ df[(df.a > 1) & (df.b > 2)]            多条件过滤    ║
║ df.mean(axis=1) / .sum(axis=0)         行(1)/列(0)运算║
║ df.apply(fn, axis=1)                   逐行自定义函数 ║
║ df.sort_values("col", ascending=False) 排序          ║
║ df.groupby("group")["col"].mean()      分组聚合      ║
║ df["new"] = df["a"] - df["b"]          创建新列      ║
║ df.merge(df2, on="key")                横向表连接    ║
║ pd.concat([d1, d2], axis=0)            纵向表拼接    ║
║ df.to_csv("out.tsv", sep="\t")         写入 TSV      ║
╚══════════════════════════════════════════════════════╝
```

### 14.4 sklearn 四步法速查

```
╔══════════════════════════════════════════════════════╗
║          sklearn 万能四步法                           ║
╠══════════════════════════════════════════════════════╣
║ 1. X = data.T.values; y = labels                    ║
║ 2. X_tr, X_te, y_tr, y_te = train_test_split(...)   ║
║ 3. model = XXXClassifier(...); model.fit(X_tr, y_tr)║
║ 4. y_pred = model.predict(X_te)                     ║
║    print(accuracy_score(y_te, y_pred))              ║
╚══════════════════════════════════════════════════════╝
```

---

## 附录：生信分析 × Python 知识对照表

| 你想做的分析 | 核心需要的 Python 知识 |
|-------------|----------------------|
| 读 FASTA/FASTQ，算 GC 含量、k-mer | `with open`, 字符串方法, 列表, 字典 |
| 基因表达矩阵处理 (TCGA/GEO) | Pandas (`read_csv`, `merge`, `groupby`, `apply`, 布尔过滤) |
| 差异表达分析 (批量 t 检验) | `scipy.stats`, `multipletests` (FDR 校正), 列表推导式 |
| PCA / t-SNE / UMAP 降维可视化 | NumPy, `sklearn.decomposition.PCA`, Matplotlib |
| 火山图 / 热图 / 箱线图 | Matplotlib + Seaborn, `scatter`, `heatmap`, `boxplot` |
| 癌症亚型分类 (RF/SVM/XGBoost) | `sklearn` 四步法 (`fit`→`predict`→`evaluate`) |
| 生存分析 | `lifelines` 包, Pandas 数据整理 |
| 基因富集分析 (GO/KEGG) | `set` 集合运算, `scipy.stats.hypergeom`, 字典映射 |
| 单细胞 RNA-seq (标准流程) | Scanpy (`pp`预处理, `tl`工具, `pl`绑图), AnnData 结构 |
| 单细胞批次校正 (scVI) | Scanpy + scVI (需要 PyTorch 作为后端) |
| 批量下载 NCBI/GEO 数据 | `Bio.Entrez`, `Bio.SeqIO` |
| 写可复用的分析脚本 | 函数定义 (`def`), `argparse` 命令行参数, `if __name__ == "__main__"` |

---

> **恢复语感的最快方法**：取 PBMC3k 或 TCGA-BRCA 数据集，拷贝第 13 章的流水线代码到 Jupyter，逐行运行，逐行理解。卡住的地方回到对应章节查阅——那里的解释就是为了解答你此刻的疑惑。

---

*文档生成于 2026-05-23 | 配合《生信机器学习深度学习入门指南》食用效果更佳*
