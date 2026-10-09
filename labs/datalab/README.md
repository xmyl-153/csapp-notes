# datalab 实验记录

> 日期：2026-10-09　课程：CMU 15-213 Data Lab（2019 Self-Study Handout）
> 结果：**62/62**，`./dlc bits.c` 静默通过（0 违规），13/13 函数全对。

## 一、这个 lab 在做什么

用位运算实现 13 个函数：

- **10 个整数谜题**：只许用 `! ~ & ^ | + << >>`，常数不超过 8 位，只能写直线代码（无 `if`/`while`/`?:`）；
- **3 个浮点谜题**：允许分支与循环、允许任意整数常数，但**不许出现任何浮点类型、浮点常量、union/struct/数组**——
  浮点操作数以 `unsigned` 位模式传入传出，IEEE 754 的拆装全靠手写。

一句话：**把 CSAPP 第 2 章的补码、溢出、IEEE 754 从"看得懂"逼成"写得出"。**

## 二、成绩

```
Score = 62/62 [36/36 Corr + 26/26 Perf] (142 total operators)
```

```
Correctness Results	Perf Results
Points	Rating	Errors	Points	Ops	Puzzle
1	1	0	2	7	bitXor
1	1	0	2	1	tmin
1	1	0	2	8	isTmax
2	2	0	2	7	allOddBits
2	2	0	2	2	negate
3	3	0	2	9	isAsciiDigit
3	3	0	2	8	conditional
3	3	0	2	15	isLessOrEqual
4	4	0	2	6	logicalNeg
4	4	0	2	40	howManyBits
4	4	0	2	12	floatScale2
4	4	0	2	18	floatFloat2Int
4	4	0	2	9	floatPower2
```

- **正确性 36/36**：13 题 Rating 加权。`btest.c` 里 `tscore = terrors == 0 ? 1.0 : 0.0` —— 单题错一个用例该题分数整个归零，没有部分分。
- **性能 26/26**：每题 2 分，条件是运算符个数不超过该题 `Max ops`。用量最紧的是 `isLessOrEqual`（15/24）与 `howManyBits`（40/90）。
- 13 题合计 142 个运算符。

## 三、验证方式（三条命令，缺一不可）

```bash
cd ~/csapp/labs/datalab-handout
make btest && ./btest      # 结果对不对（改了 bits.c 必须重编！）
./dlc bits.c               # 写法合不合规则：静默 = 通过
./driver.pl                # 官方口径的最终成绩（dlc + btest + 运算符统计）
```

> 这个 2019 版 handout **不带 BDD checker**（目录里没有 `bddcheck/`），穷尽验证由 `driver.pl` 内部完成。

## 四、踩过的五个坑（全部是 `dlc` 抓出来的）

| # | 症状 | 根因 | 修法 |
| - | --- | --- | --- |
| 1 | `./btest` 显示 27/36，但这个分数是假的 | `bits.c` 有语法错误 → `make` 失败 → **跑的是上一次编译的旧二进制** | 先看 `make` 输出，有 `error` 就别看分数 |
| 2 | `dlc: conditional: Illegal operator (-)` | 用 `-isNonZero` 造全 1 掩码，减号不在合法运算符里 | `int mask = (!!x << 31) >> 31;` —— 用移位造掩码 |
| 3 | `dlc: logicalNeg: Illegal operator (!)` | 题目是"不许用 `!` 实现 `!`"，而原式第一个字符就是 `!` | `(((x \| (~x+1)) >> 31) & 1) ^ 1` |
| 4 | `floatFloat2Int` 十几条 `parse error` + `undeclared variable` | 声明写在 `if` 之后，`dlc` 只认 C89：块内所有声明必须在任何语句之前 | 把 `int E, mantissa, result;` 挪到函数体开头 |
| 5 | `floatScale2(-0.0)`、负数非规格化数算错 | `0x80000000 << 1 == 0`、`0x80000001 << 1 == 2` —— 左移把符号位移丢了 | ±0 直接返回；非规格化走 `((uf & 0x7FFFFFFF) << 1) \| (uf & 0x80000000)` |

**第 1 条最值得记**：编译失败时 `btest` 不报错，它安静地测旧二进制，给出一个看起来"还行"的分数。
这是唯一一个"测试工具本身没错、是你漏看了 `make` 输出"的坑。

## 五、13 题实现一览

### bitXor　`Rating 1` / 上限 `14` ops

> `x^y`，只许用 `~ &`

```c
return ~(~x & ~y) & ~(x & y);
```

### tmin　`Rating 1` / 上限 `4` ops

> 返回最小补码整数 `0x80000000`

```c
return 1 << 31;
```

### isTmax　`Rating 1` / 上限 `10` ops

> x 是不是 `0x7FFFFFFF`

```c
return !((x + 1) ^ ~x) & !!(x + 1);
```

### allOddBits　`Rating 2` / 上限 `12` ops

> 所有奇数位是否都为 1

```c
int mask = 0xAA;
mask = mask | (mask << 8);
mask = mask | (mask << 16);
return !((x & mask) ^ mask);
```

### negate　`Rating 2` / 上限 `5` ops

> `-x`，不许用减号

```c
return ~x + 1;
```

### isAsciiDigit　`Rating 3` / 上限 `15` ops

> `0x30 <= x <= 0x39`

```c
int t = x + ~0x30 + 1;
int geLower = !(t >> 31);
int leUpper = ((t + ~9) >> 31);
return geLower & leUpper;
```

### conditional　`Rating 3` / 上限 `16` ops

> 等价于 `x ? y : z`

```c
int mask = (!!x << 31) >> 31;
return (y & mask) | (z & ~mask);
```

### isLessOrEqual　`Rating 3` / 上限 `24` ops

> `x <= y`

```c
int signX = (x >> 31) & 1;
int signY = (y >> 31) & 1;
int sameSign = !(signX ^ signY);
int z = y + ~x + 1;
int sameSignResult = !(z >> 31);
return (sameSign & sameSignResult) | ((!sameSign) & signX);
```

### logicalNeg　`Rating 4` / 上限 `12` ops

> 实现 `!` 运算符（禁用 `!`）

```c
return (((x | (~x + 1)) >> 31) & 1) ^ 1;
```

### howManyBits　`Rating 4` / 上限 `90` ops

> 表示 x 最少需要多少位

```c
int y = x ^ (x >> 31);
int count = 0;
int hasHigh;
hasHigh = !!(y >> 16); count += hasHigh << 4; y >>= hasHigh << 4;
hasHigh = !!(y >> 8);  count += hasHigh << 3; y >>= hasHigh << 3;
hasHigh = !!(y >> 4);  count += hasHigh << 2; y >>= hasHigh << 2;
hasHigh = !!(y >> 2);  count += hasHigh << 1; y >>= hasHigh << 1;
hasHigh = !!(y >> 1);  count += hasHigh << 0; y >>= hasHigh << 0;
return count + 1 + (y & 1);
```

### floatScale2　`Rating 4` / 上限 `30` ops

> `2.0*f`

```c
unsigned e = (uf >> 23) & 0xFF;
if (e == 255) return uf;                 /* Inf / NaN：原样返回 */
if ((uf & 0x7FFFFFFF) == 0) return uf;   /* ±0.0 单独处理：
                                            0x80000000 << 1 会变成 +0.0，符号位丢了 */
if (e > 0) {
    return uf + (1 << 23);   /* 规格化：指数加 1 */
}
return ((uf & 0x7FFFFFFF) << 1) | (uf & 0x80000000);
```

### floatFloat2Int　`Rating 4` / 上限 `30` ops

> `(int) f`

```c
int s = (uf >> 31) & 1;            /* 符号位 */
int e = (uf >> 23) & 0xFF;         /* 阶码 */
int f = uf & 0x7FFFFF;             /* 尾数 */
int E, mantissa, result;
if (e == 255) return 0x80000000;   /* Inf / NaN → 题目指定的哨兵值 */
if (e == 0) return 0;              /* 非规格化数 / ±0 → 0 */
E = e - 127;                       /* 实际指数 */
if (E < 0) return 0;               /* |f| < 1 → 截断为 0 */
if (E >= 31) return 0x80000000;    /* 超出 int 范围 */
mantissa = (1 << 23) | f;          /* 隐含前导 1 */
if (E <= 23) {
    result = mantissa >> (23 - E); /* 右移丢小数位（截断） */
} else {
    result = mantissa << (E - 23); /* 左移补零 */
}
if (s) result = -result;
return result;
```

### floatPower2　`Rating 4` / 上限 `30` ops

> `2.0^x`

```c
if (x >= 128) return 0x7F800000;
if (x >= -126) return (x + 127) << 23;
if (x >= -149) return 1 << (x + 149);
return 0;
```

## 六、文件说明

| 文件 | 内容 |
| --- | --- |
| `bits.c` | 提交文件，13 个函数的实现（整个 lab 唯一需要改的文件） |
| `driver_output.txt` | `./driver.pl` 的完整输出，分数凭证 |
| `REFLECTION.md` | 位级洞察记录：为什么这么写、边界情况怎么想 |
