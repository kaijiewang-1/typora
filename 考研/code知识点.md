# **L1-002 打印沙漏**

本题要求你写个程序把给定的符号打印成沙漏的形状。例如给定17个“*”，要求按下列格式打印

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306151620473.png" alt="image-20260306151620473" style="zoom:45%;" />

所谓“沙漏形状”，是指每行输出奇数个符号；各行符号中心对齐；相邻两行符号数差2；符号数先从大到小顺序递减到1，再从小到大顺序递增；首尾符号数相等。

给定任意N个符号，不一定能正好组成一个沙漏。要求打印出的沙漏能用掉尽可能多的符号。

### 输入格式:

输入在一行给出1个正整数N（≤1000）和一个符号，中间以空格分隔。

### 输出格式:

首先打印出由给定符号组成的最大的沙漏形状，最后在一行中输出剩下没用掉的符号数。

### 输入样例:

19 *

### 输出样例:

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306151758402.png" alt="image-20260306151758402" style="zoom:50%;" />

数学公式：

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306150216151.png" alt="image-20260306150216151" style="zoom:50%;" />

打印公式：

```c++
 // 打印上半部分（含中间一行）
    for (int i = k; i >= 1; i -= 2) {
        int spaces = (k - i) / 2;
        for (int j = 0; j < spaces; j++) cout << ' ';
        for (int j = 0; j < i; j++) cout << ch;
        cout << endl;
    }

    // 打印下半部分
    for (int i = 3; i <= k; i += 2) {
        int spaces = (k - i) / 2;
        for (int j = 0; j < spaces; j++) cout << ' ';
        for (int j = 0; j < i; j++) cout << ch;
        cout << endl;
    }
```

# 错误点：

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306152748722.png" alt="image-20260306152748722" style="zoom:50%;" />



# **L1-003 个位数统计**

给定一个 *k* 位整数 *N*=*d**k*−110*k*−1+⋯+*d*1101+*d*0 (0≤*d**i*≤9, *i*=0,⋯,*k*−1, *d**k*−1>0)，请编写程序统计每种不同的个位数字出现的次数。例如：给定 *N*=100311，则有 2 个 0，3 个 1，和 1 个 3。

### 输入格式：

每个输入包含 1 个测试用例，即一个不超过 1000 位的正整数 *N*。

### 输出格式：

对 *N* 中每一种不同的个位数字，以 `D:M` 的格式在一行中输出该位数字 `D` 及其在 *N* 中出现的次数 `M`。要求按 `D` 的升序输出。

### 输入样例：

100311

### 输出样例：

0:2

1:3

3:1

# 错误点：

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306155056456.png" alt="image-20260306155056456" style="zoom: 67%;" />

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260306154918544.png" alt="image-20260306154918544" style="zoom:50%;" />

# **L1-006 连续因子**

一个正整数 *N* 的因子中可能存在若干连续的数字。例如 630 可以分解为 3×5×6×7，其中 5、6、7 就是 3 个连续的数字。给定任一正整数 *N*，要求编写程序求出最长连续因子的个数，并输出最小的连续因子序列。

### 输入格式：

输入在一行中给出一个正整数 *N*（1<*N*<231）。

### 输出格式：

首先在第 1 行输出最长连续因子的个数；然后在第 2 行中按 `因子1*因子2*……*因子k` 的格式输出最小的连续因子序列，其中因子按递增顺序输出，1 不算在内。

### 输入样例：

630

### 输出样例：

3

5*6*7

## 思路

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260307161843542.png" alt="image-20260307161843542" style="zoom:50%;" />

## 错误点

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260307161724881.png" alt="image-20260307161724881" style="zoom:50%;" />
