# 硬币收集问题算法设计

## 1. 符号

设： 有一个 $n$ 行 $m$ 列的棋盘 $M(n,m)$ ，并用 $M(i,j)$ 表示 $M$ 的第 $i$ 行第 $j$ 列位置上放有几枚硬币（在本题中取值为0或1）

设： 有一个机器人 $R$ 位于 $M$ 的最左上角，初始坐标为 $R(1,1)$

令： $F(i,j)$ 为机器人走到 $(i,j)$ 位置时能收集到的硬币的最大值

令： $R(i+1,j)$ 为机器人向下移动一个单位

令： $R(i,j+1)$ 为机器人向右移动一个单位

令： $OPT(n,m)$ 为机器人在n行m列棋盘中移动所能收集到的最大硬币数量，即全局最优解

## 2. 精确解

### 解法一：Brute force

1. 因为机器人在移动到除了最右边或最下边的格子外的位置时，都需要进行一次选择——向右或向下，所以可以生成一棵二叉树来表示所有可能的解
2. 因为机器人从 $R(1,1)$ 到 $R(n,m)$ 需要移动 $n+m-2$ 步，又因为机器人只能向右或者向下移动，所以深度优先遍历二叉树后有 $2^{n+m-2}$ 条路径，因此可以采用Brute force遍历所有路径，等效于对 $['DOWN'] \times (n-1) + ['RIGHT'] \times (m-1)$ 做全排列，然后遍历所有路径找到最优解

#### 伪代码一

<pre class="pseudocode" lineNumber="true">
\begin{algorithm}
\caption{Exact Solution: Brute force}
\begin{algorithmic}
\PROCEDURE{BruteForce}{$M[n,m], Path_{temp}, i, j$}
    \STATE $Path_{temp} = ['DOWN'] \times (n-1) + ['RIGHT'] \times (m-1)$
    \PROCEDURE{Permutations}{$arr, position, end$}
        \IF{$position = end$}
            \IF{\NOT $arr \in Path_{aval}$}
                \STATE $Path_{aval}.append(arr.copy())$
            \ENDIF
        \ELSE
            \FOR{$index=position$ \TO $end$}
                \STATE exchange
                \STATE exchange $arr[index]$ with $arr[position]$
                \STATE \CALL{Permutations}{$arr, position+1, end$}
                \STATE exchange $arr[index]$ with $arr[position]$
            \ENDFOR
        \ENDIF
    \ENDPROCEDURE
    \STATE \CALL{Permutations}{$Path_{temp}, 0, len(Path_{temp})$}
    \FOR{$Path_{each} \in Path_{aval}$}
        \STATE $i,j=0,0$
        \STATE $F=M[0,0]$
        \FOR{$step \in Path_{each}$}
            \IF{$step="DOWN"$}
                \STATE $i += 1$
            \ELIF{$step="RIGHT"$}
                \STATE $j += 1$
            \ENDIF
            \STATE $F += M[i,j]$
        \ENDFOR
        \IF{$F > Ans$}
            \STATE $Ans=F$
            \STATE $Path=Path_{each}$
        \ENDIF
    \ENDFOR
    \RETURN $Ans,Path$
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
</pre>

### 解法二：迭代遍历

1. 因为机器人只能从左向右，从上向下移动，且机器人会收集经过的路径上的所有硬币，所以 $F(i,j)$ 的值为 $F(i-1,j)$ 和 $F(i,j-1)$ 中较大的值加上 $c_{ij}$
2. 因为机器人在 $i=1$ 时无法向上移动，前一推理结果，等效为上一行的硬币数量为0，即 $P(0,j)=0$，同理可得 $P(i,0)=0$

$$ F(i,j)=max\{ F(i-1,j),F(i,j-1)\}+M(i,j) {\quad} for {\quad} 1 \leq i \leq n, {\quad} 1 \leq j \leq m \tag{1-1}$$
$$ F(i,0)=0 {\quad} for {\quad} 1 \leq i \leq n {\quad} and {\quad} F(0,j)=0 {\quad} for {\quad} 1 \leq j \leq m \tag{1-2}$$

#### 伪代码二

<pre class="pseudocode" lineNumber="true">
\begin{algorithm}
\caption{Exact Solution: Iterative Traversal}
\begin{algorithmic}
\PROCEDURE{IterativeTraversal}{$M[n,m]$}
    \STATE $F[1,1] = M[1,1]$
    \FOR{$j=2$ \TO $m$}
        \STATE $F[1,j] = F[1,j-1] + M[1,j]$
    \ENDFOR
    \FOR{$i=2$ \TO $n$}
        \STATE $F[i,1] = F[i-1,1] + M[i,1]$
        \FOR{$j=2$ \TO $m$}
            \STATE $F[i,j]=max(F[i-1,j],F[i,j-1])+M[i,j]$
        \ENDFOR
    \ENDFOR
    \STATE $x, y=n, m$
    \WHILE{$x \gt 1$ \AND $y \gt 1$}
        \IF{$F[x-1,y] \gt F[x,y-1]$}
            \STATE $y-=1$
            \STATE $path += "UP -> "$
        \ELIF{$F[x-1,y] \lt F[x,y-1]$}
            \STATE $x-=1$
            \STATE $path += "LEFT -> "$
        \ELSE
            \STATE $x-=1, y-=1$
            \STATE $path+="LEFT/UP -> UP/LEFT -> "$
        \ENDIF
    \ENDWHILE
    \RETURN $F[n,m], {\quad} path$
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
</pre>

## 3. 近似解

### 解法三：贪心算法

1. 因为机器人每移动一次，若机器人没有移动到最右边或最左边 $(i,j) {\quad} 1 \leq i < n-1, 1 \leq j < m-1$ ，就需要判断下一步要移动到右边相邻的位置或者下边相邻的位置，此时可以遇到两种情况： $A.$ 移动到两个位置后硬币数量相同（对应两个邻居都右硬币或都没有硬币的情况），以及 $B.$ 移动到两个位置后硬币数量不同（对应其中一个位置有硬币，另一个位置没有硬币的情况）；所以可以设置指导规则，判断最优路线
2. 指导规则为：情况A下，始终选择右边的相邻位置；情况B下，始终选择有硬币的相邻位置；

$$ F(i,j)=F(i,j-1)+M(i,j) {\quad} if {\quad} M(i-1, j) \leq M(i, j-1) \tag{2-1}$$
$$ F(i,j)=F(i-1,j)+M(i,j) {\quad} if {\quad} M(i-1, j) > M(i, j-1) \tag{2-2}$$

#### 伪代码三

<pre class="pseudocode" lineNumber="true">
\begin{algorithm}
\caption{Approximate Solution 1: Greedy Algorithm}
\begin{algorithmic}
\STATE $path[n+m-1]$
\PROCEDURE{GreedyAlgorithm}{$M[n,m], i, j$}
    \IF{$i < n$ \AND $j < m$}
        \IF{$M(i+1,j) = M(i,j+1)$}
            \STATE $path.append("RIGHT")$
            \STATE $ans=$ \CALL{GreedyAlgorithm}{i, j+1}
        \ELIF{$M(i+1,j) > M(i,j+1)$}
            \STATE $path.append("DOWN")$
            \STATE $ans=$ \CALL{GreedyAlgorithm}{i+1, j}
        \ELIF{$M(i+1,j) < M(i,j+1)$}
            \STATE $path.append("RIGHT")$
            \STATE $ans=$ \CALL{GreedyAlgorithm}{i, j+1}
        \ENDIF
    \ELIF{$i=n$}
        \STATE $ans=\sum_{j}^m{M_{i,j}}$
        \STATE $path.append(["RIGHT"]\times({m-j}))$
    \ELIF{$j=m$}
        \STATE $ans=\sum_{i}^n{M_{i,j}}$
        \STATE $path.append(["DOWN"]\times({m-j}))$
    \ELSE
        \RETURN $path, ans$
    \ENDIF
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
</pre>

### 解法四：模拟退火算法

1. 因为贪心算法在硬币密度过低或过高或分布不均匀的情况下很难找到全局最优解，所以设计退火算法引入随机扰动并多次迭代来避免陷入局部最优解
2. 优化解法三的指导规则为：情况A下，完全随机选择两个位置中的一个；情况B下，设置可调节的概率参数 $p \in (0,1)$ ，使得机器人以 $p$ 的概率移动到有硬币的位置；
3. 利用初始化温度 $Temperature_{init}$ 和温度变化步长 $Temperature_{change}$ 以及退火停止温度 $Temperature_{min}$ 控制退火循环次数
4. 利用 $Energy=(Temperature_{init}-Temperature_{now})/Temperature_{init}$ 和 $EnergyFix$ 为初始高温时决策的随机性提供较大的动能（可能性），动能随着温度降低而降低
5. 记录退火过程中每一次搜索到的最优解和路径，取最大值作为全局最优解的近似解

$$ F(i,j)=F(i-1,j)+M(i,j) {\quad} if {\quad} M(i-1, j) > M(i, j-1) {\quad} and {\quad} q-Energy > random(0,1) \tag{3-1}$$
$$ F(i,j)=F(i-1,j)+M(i,j) {\quad} if {\quad} M(i-1, j) = M(i, j-1) {\quad} and {\quad} random(0,1)>0.5 \tag{3-2}$$

#### 伪代码四

<pre class="pseudocode" lineNumber="true">
\begin{algorithm}
\caption{Approximate Solution 1: Simulated Annealing Algorithm}
\begin{algorithmic}
\PROCEDURE{SimulatedAnnealing}{$M[n,m], i, j$}
    \STATE $Temperature_{now} = Temperature_{init}$
    \WHILE{$Temperature_{now} > Temperature_{min}$}
        \STATE $Energy = (Temperature_{init}-Temperature_{now})/Temperature_{init}$
        \IF{$i < n$ \AND $j < m$}
            \IF{$M(i+1,j) = M(i,j+1)$}
                \IF{$random(0,1) > 0.5$}
                    \STATE $Path_{temp}.append("RIGHT")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i, j+1}
                \ELSE
                    \STATE $Path_{temp}.append("DOWN")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i+1, j}
                \ENDIF
            \ELIF{$M(i+1,j) > M(i,j+1)$}
                \IF{$p + {Energy \times EnergyFix} > random(0,1)$}
                    \STATE $Path_{temp}.append("DOWN")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i+1, j}
                \ELSE
                    \STATE $Path_{temp}.append("RIGHT")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i, j+1}
                \ENDIF
            \ELIF{$M(i+1,j) < M(i,j+1)$}
                \IF{$p + {Energy \times EnergyFix} > random(0,1)$}
                    \STATE $Path_{temp}.append("RIGHT")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i, j+1}
                \ELSE
                    \STATE $Path_{temp}.append("DOWN")$
                    \STATE $Ans_{temp}=$ \CALL{SimulatedAnnealing}{i+1, j}
                \ENDIF
            \ENDIF
        \ELIF{$i=n$}
            \STATE $Ans_{temp}=\sum_{j}^m{M_{i,j}}$
            \STATE $Path_{temp}.append(["RIGHT"]\times({m-j}))$
        \ELIF{$j=m$}
            \STATE $Ans_{temp}=\sum_{i}^n{M_{i,j}}$
            \STATE $Path_{temp}.append(["DOWN"]\times({m-j}))$
        \ELSE
            \IF{$Ans_{temp} > Ans_{max}$}
                \STATE $Ans_{max} = Ans_{temp}$
                \STATE $Path_{prop} = Path_{temp}$
            \ENDIF
        \ENDIF
        \STATE $Temperature_{now} = Temperature_{now} - Temperature_{chang}$
        \ENDWHILE
    \RETURN $Ans_{max}, Path_{prop}$
\ENDPROCEDURE
\end{algorithmic}
\end{algorithm}
</pre>
