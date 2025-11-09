## **DP 状态转移方程**
设 ![image](https://cdn.nlark.com/yuque/__latex/d9ae4bc0ba9f7c56540f3aaecf525c0e.svg) 表示考虑前 ![image](https://cdn.nlark.com/yuque/__latex/2443fbcfeb7e85e1d62b6f5e4f27207e.svg) 个物品，当前背包剩余容量为 ![image](https://cdn.nlark.com/yuque/__latex/b891664b42113aee13f0bac25eb998e5.svg) 时，所能获得的最大价值；![image](https://cdn.nlark.com/yuque/__latex/d99fd2df7b5f652a4b7fc593fb9df750.svg) 和 ![image](https://cdn.nlark.com/yuque/__latex/0480d9f663a9cd686bae9ee284ce1bbb.svg) 分别为第 ![image](https://cdn.nlark.com/yuque/__latex/2443fbcfeb7e85e1d62b6f5e4f27207e.svg) 个物品的重量和价值。

则状态转移方程为：  

![image](https://cdn.nlark.com/yuque/__latex/569adbde0a75cde94b96271207249d50.svg)

## **对应术语**
| **DP术语** | **对应RL术语** |
| :--- | :--- |
| 状态 | ![image](https://cdn.nlark.com/yuque/__latex/1fecd70bfadc37b8edfe3c8188493600.svg)，即“考虑第 i 个物品，剩余容量 c” |
| 动作 | ![image](https://cdn.nlark.com/yuque/__latex/72150254a44cecea5a7065078aebb853.svg)：是否选第 i 个物品 |
| 奖励 | ![image](https://cdn.nlark.com/yuque/__latex/749518b9027ab2db1e8a9f7e0a6ec639.svg) |
| 状态转移 | ![image](https://cdn.nlark.com/yuque/__latex/d112a29861272dc9b9be2bb902928009.svg) |
| （状态）价值函数 | ![image](https://cdn.nlark.com/yuque/__latex/8d60bef27eeb514b170a341b535bde6b.svg) |


## **从 DP 状态转移方程到 Bellman 方程**
1. DP 状态转移方程

![image](https://cdn.nlark.com/yuque/__latex/540f8c6f3f3e9e5dde546224d32c7e9a.svg)

2. 代换为 RL 术语

![image](https://cdn.nlark.com/yuque/__latex/2b31541f652ed2e646044c36aa2dd83c.svg)

进一步整理得：

![image](https://cdn.nlark.com/yuque/__latex/f2cf51f2f0da9ec62e212c6f4cc5ec81.svg)

这与 Bellman 方程的形式相同（![image](https://cdn.nlark.com/yuque/__latex/a6c5b3a9218f95e5503499654cf9b08f.svg)）：

![image](https://cdn.nlark.com/yuque/__latex/a743cdce51d01c55b0f2fc5b4ba0a635.svg)