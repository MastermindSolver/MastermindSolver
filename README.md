## Hi there 👋
（本仓库为AI生成，暂未核对）
<!--
**MastermindSolver/MastermindSolver** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->


# Bulls-only Mastermind Solver (Entropy Policy)

一个纯前端（单文件 HTML）的 **Bulls-only / Black-peg-only Mastermind** 求解器。

- 支持 **n 位格子**、**m 进制（2–36）**
- 规则：每次猜测只返回 **bulls**（命中位数 / “位置和值都正确”的个数）
- 自动维护候选集，并在右侧给出 **Policy / Next guess ranking**（下一步猜什么最“信息密集”）

> 本项目重点在于：把“下一步猜什么”转成信息论问题（熵最大化），并把它做成浏览器里可运行的交互式工具。

---

## 规则定义（Bulls-only / Black-peg-only）

隐藏答案为长度 `n` 的序列，符号来自 `0..m-1`（以 2–36 进制字符表示）。

每次提交 guess `x` 后，系统返回：

\[
\mathrm{bulls}(x,y) = |\{ i \in [0,n-1] : x_i = y_i \}|
\]

也就是“正确位置且正确值”的个数。

---

## 核心思路：候选集 + 最大熵（Maximum Entropy）选猜

### 1) 维护候选集（Consistency Filtering）

每一轮输入 `(guess, bulls)` 后，把所有不满足该反馈的候选答案剔除：

\[
C \leftarrow \{ s \in C : \mathrm{bulls}(guess, s) = bulls \}
\]

直到候选集只剩 1 个时答案唯一确定。

### 2) Policy：按反馈分桶，最大化信息增益（熵）

对某个候选猜测 `g`，把当前候选集 `C` 按可能反馈 `r ∈ {0..n}` 分桶：

- `count_r = |{ s ∈ C : bulls(g, s) = r }|`
- `p_r = count_r / |C|`

该猜测的期望信息增益等价于香农熵：

\[
H(g) = \sum_{r=0}^{n} p_r \log_2 \frac{1}{p_r}
      = -\sum_{r=0}^{n} p_r \log_2 p_r
\]

本工具将所有被评估的猜测按以下指标排序（默认）：

1. **Entropy**：越大越好（信息量越高）
2. **WorstRemain**：最大桶大小（最坏情况下剩余候选数）越小越好
3. **ExpectedRemain**：期望剩余候选数越小越好

---

## 性能说明

候选集大小为 `m^n`。当规模较大时，精确评估所有猜测的熵会很慢。

因此工具提供三种评估模式：

- **exact**：对被评估的 guess 集合逐一精确计算分桶与熵（n=4,m=10 时可用）
- **sample**：对 guess 集合随机采样，再做精确分桶（更快）
- **auto**：根据粗略操作量自动选择 exact 或 sample

另外可选“猜测集合”来源：

- 仅候选集（更快）
- 全空间（可能更好但更慢）

---

## 使用方法

1. 直接用浏览器打开 `ms.q2333.com`
3. 设置 `格子数n` 与 `每格子内的量m (进制)`
4. 每轮输入：
   - Guess（长度 n，字符在 base 内）
   - Bulls（0..n 的整数）
5. 点击“提交这一轮”，右侧会更新 `Next guess ranking`

---

## 免责声明

- 本工具默认假设隐藏答案在当前候选集中 **均匀分布**，因此“最大熵策略”主要优化的是 **平均表现**，并不等同于对抗式最坏情况最优（minimax）。
- 对于非常大的 `m^n`，前端枚举会受内存/时间限制；此时建议使用采样模式或降低规模。

---

## 致谢与参考（Acknowledgements & References）

猜谜游戏：
- 页面地址：https://lczkeyide.github.io/guessnum/
  - 作者：lczkeyide@GitHub
  - 仓库地址；https://github.com/lczkeyide/lczkeyide.github.io


本项目的“最大熵选猜”思想与表述，参考了：

- Erik Göransson Gaspar 的 *Optimal Mastermind*（信息论/熵驱动的 Mastermind 策略说明与开源实现）
  - https://www.goranssongaspar.com/mastermind
  - https://github.com/ErikGoranssonGaspar/OptimalMastermind

关于 **Black-peg / Bulls-only Mastermind**（仅返回命中位数）的理论研究与边界结果，推荐参考：

- Gerold Jäger, Marcin Peczarski, *The number of pessimistic guesses in Generalized Black-peg Mastermind*, Information Processing Letters, 2011.
- Michael T. Goodrich, *On the Algorithmic Complexity of the Mastermind Game*, 2009.
- 以及相关综述资料/讲义（Mastermind variants, query complexity 等）。

本仓库为学习与工程实现目的整理，感谢包括但不限于上述作者与研究者的公开工作。
