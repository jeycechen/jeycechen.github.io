---
title : dp
---
动态规划

## 区间dp

常用于字符串，戳气球...

一般是三维 `dp[i][j][k]` 表示区间 [i,j] 内状态k 的方案数/... 

但是有可能超时 

所以结合 线段树， 树状数组 ，二分等等



## 数位dp

与数字有关

>  例题 2801 统计范围内的步进数字数目
>
>  给定两个正整数low和high 计算 闭区间[low, high]之间的步进数字数目
>
>  步进数字: 相邻数位之间差的绝对值 是 1 ，
>
>  对 1e9 + 7 取模然后返回
>
>  1 <= low, high <= 10^100

 

模版

f(i, pre, is_limit, is_num); 表示构造第i位及其之后数位的合法方案数量，

i , pre 根据题目要求决定是否需要，后两个必须

pre 前一位数位是什么？

is_limit 是否收到了最大数字n的限制，比如 123， 第一位取了1 ，第二位就收到了限制 ，0～2 

is_num 前面是否有数字，如果前面没有数字，这一位也可以跳过，没有数字，可以是>= 1的数字，也可以跳过， 如果前面有数字，这一位就取决于is_limit


递归入口

f(0, 0, true, false);

从s[0]开始枚举 ，

pre初始化成什么都可以，填第一个数的时候忽略了pre

一开始要受到n的约束

一开始没有填数字

```c++
const int MOD = 1e9 + 7;
int calc(string &s){
  int m = s.size(), memo[m][10];
  memset(memo ,-1, sizeof(memo));
  function<int (int, int, bool, bool)> f = [&](int i, int pre, bool is_limit, bool is_num) -> int {
    if(i == m) return is_num;// 长度满足，is_num为true表示前面的数据是合法非空的
    if(!is_limit && is_num && memo[i][pre] != -1) return memo[i][pre];
    int res = 0;
    if(!is_num){ //前面没有有效数字，这一位也跳过有多少种？
    	res = f(i+1, pre, false, false);
    }
    //这一位不跳过了
    int up = is_limit ? s[i] - '0' : 9;
    for(int d = 1 - is_num; d<=up;d++){
      if(!is_num || abs(d - pre) == 1){
        res += f(i+1, d, d == up, true) % mod;
      }
    }
    if(!is_limit && is_num) memo[i][pre] = res;
    return res;
  };
  return f(0,0,true,false);
}
```

calc(high) - calc(slow-1) 就是答案，但是slow - 1不好减

=》 calc(high) - calc(low) + valid(low);

Valid(low) 判断 low是否是步进数字，返回 1， 或者 0 

```c++
int result(string low, string high){
  return (calc(high) - calc(low) + MOD + valid(low)) % MOD; // + MOD防止算出负数
}
```



### 补充

- (a+b)%M = (a % m + b % m)%m

- (a*b)%M = ((a%M) * (b%M)) % M

- 这里的f(i, pre, is_limit, is_num) 明明状态是4个，按道理记忆化递归的memo的维度也应该和函数的维度一样，为何memo只有 i和 pre两个维度？

  因为 实际上这里做了简化，is_limit， is_num 有四种状态

  |              | is_limit:false | is_limit:true |
  | ------------ | -------------- | ------------- |
  | is_num:false | A              | C             |
  | is_num:True  | B              | D             |

  A 与 C都是前面没有数字，受到限制与否不起作用，前面没有数字，则这种情况只会遍历一次，比如第一位不填，第二位也不填，后面再怎么递归也不会重新进入这种情况

  B 代表前面有数字有不受到限制的情况，这实际上就是memo记录的东西

  D 代表前面有数字的情况下，这一位也受到了限制，那么这种情况实际上也只会经过一次，没必要记忆，比如2344557， 第一位填2，第二位填3， 后面无论怎么递归都不会再次递归到第一位填2 ，第二位为3 的情况

  也就是保存ACD这三种情况没必要，后面不会复用，只有保存B才有用，所以此刻的memo记录的就是memo[i] [pre] <mark style="background-color yellow"> 不受到约束的情况下第i位开始递归 前一位数字为pre的结果有多少种</mark>

  <font color='blue'>(1,3,false,false) 表示i = 0填3 ，从i=1往后随便填的方案数量</font>

  

## 记忆化搜索

dp是顺推导，自底向上

记忆化搜索是 自顶向下 递归 +  数组记忆