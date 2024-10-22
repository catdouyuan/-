# Datalab

## 逻辑运算的条件判断

当不能用if-else语句,?三目运算符这种判断语句时,如何实现条件判断n?
用类似异或形式就可以进行条件判断：(~x&y)|(x&~z)这里的x是0或1,y和z是两个分支

下面是一个例子：

- 要求：对每个比特（位）分别执行 x ? y : z
- 允许的操作符：~、&、^、|
- 最大操作次数:4
  
  ```cpp
  int bitConditional(int x, int y, int z) {
   // 对每一位，如果 x_i 为 1，那么选择 y_i，否则选择 z_i
   return (x & y) | (~x & z);}
    ```

## 掩码

掩码(Mask)是一种特殊的值，通过位运算符与另一个值进行操作，以提取、设置或清除特定位。掩码通常用于访问或修改数据结构中的单个位或一组位。

构造掩码是一个常见的编程技巧，它可以用于各种应用，例如位掩码、位操作、数据压缩等。

下面给出几个功能的实现：

### 提取

- 要求：如果 x 和 y 符号相同，返回 1,否则返回 0
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:5
  
   ```cpp
  int sameSign(int x, int y) {
   // 分别计算x和y的符号位，然后异或判断是否相同
   int sign = 1 << 31;
   int x_sign = x & sign;
   int y_sign = y & sign;
   return !(x_sign ^ y_sign);}
   ```

当然也可以写成下面形式

```cpp
  int sameSign(int x, int y) {
   // 分别计算x和y的符号位，然后异或判断是否相同
  return !((x >> 31) ^ (y >> 31));
  }
```

### 清除

我们可以用掩码来清除一个数的某些位。例如，我们可以使用掩码 0xFFFFFF00 来清除一个数的低 8 位，然后使用掩码 0x000000FF 来清除一个数的高 8 位。

- 要求：交换第 n 字节和第 m 字节
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:16
  
```cpp
  int byteSwap(int x, int n, int m) {
   // 首先保存原始x，然后在对应的byte上按位与消去，再按位或上交换后的byte
   int origin = x, clip_mask, swap_mask;
   n <<= 3, m <<= 3;
   // 0xff << n 表示将第 n 个字节保留，其他字节清零
   // 取反后，表示将第 n 个字节清零，其他字节保留
   clip_mask = ~((0xff << n) | (0xff << m));
   x &= clip_mask;
   // 先通过右移 n*8 位，将第 n 个字节移动到第 0 个字节，与 0xff 进行与运算，得到第 n 个字节，再左移 m*8 位，即实现将第 n 个字节移动到第 m 个字节
   swap_mask = (0xff & (origin >> n)) << m | (0xff & (origin >> m)) << n;
   // 完形填空
   x |= swap_mask;
   return x;
}
```

### 设置

设置是一个比较难的点，就像高数里构造函数来进行求解一样，需要一定的训练和灵感。

- 要求：清除 x 的二进制形式中连续的 1
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:16
  
```cpp
    int cleanConsecutive1(int x) {
   // 左右移动一位成为mask，左侧mask要额外考虑算数右移对最高位的影响
   int right_mask = x << 1;
   int left_mask = x >> 1 & ~(1 << 31);
   x &= ~(right_mask | left_mask);
   return x;
   }
```

上面的方法属于比较容易想到的，下面给出一个更巧妙的方法：

```cpp
int cleanConsecutive1(int x) {
   int t = x & (x << 1);
   return x & ~(t | t >> 1);
   // 或
   // return x ^ (t | t >> 1);
} 
```

首先我们来看看t的作用:当t的某一个比特位为0时,x同一位以及右边一位都为1,那么(t|t>>1)中的t>>1的意思是x同一位以及左边一位都为1(不明白的可以随便写一个数来试试看),样一来(t|t>>1)就存储了所有连续1的比特位,先取反然后对x进行&就可以清除连续1的比特位了。

## 练习

### 位运算

#### byteSwap

- 要求：交换第 n 字节和第 m 字节
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:16

```cpp
int swapbyte(int x, int n, int m) {
 int mask = 0xff;
 int newn = n << 3;
 int newm = m << 3;
 int newx = x & ~((mask << newn) | (mask << newm));//保留除了第n和m个字节的其他字节，同时将第n和m个字节清零
 int a = ((x >> newn) & mask) << newm;//将第n个字节移动到第m个字节，下面的同理
 int b = ((x >> newm) & mask) << newn;
 newx |= a | b;
 return newx;

}
```

#### logicalShift

- 要求：把 x 逻辑右移 n 位
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:16
  
```cpp
int logicalshift(int x,int n) {
 int flag = 1 << 31;
 int sign = x & flag;
 int re = (x >> n) & ~(flag >> n<<1);
 return sign & re;
}
```

#### cleanConsecutive1

- 要求：计算 x 的二进制形式中最高位开始的前导 1 的数量
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:50
  
```cpp
int leftBitCount(int x) {
   //伪代码
   int ans = 0;
   if (x 最高 16 位均为 1){
      ans += 16;
      x = x << 16;
   }
   if (x 最高 8 位均为 1){
      ans += 8;
      x = x << 8;
   }
   if (x 最高 4 位均为 1){
      ans += 4;
      x = x << 4;
   }
   if (x 最高 2 位均为 1){
      ans += 2;
      x = x << 2;
   }
   if (x 最高 1 位均为 1){
      ans += 1;
      x = x << 1;
   }
   // 对于 32 位全为 1 的情况特判
   if (x == -1){
      ans += 1;
   }
}

```cpp
int leftbitcount(int x) {
 int ans = 0,pt_16,pt_8,pt_4,pt_2,pt_1,is_neg_1;

 int tmin = 1 << 31;
 pt_16 = !(~(x & (tmin >> 15))>>16)<<4;
 ans += pt_16;
 x <<= pt_16;
 pt_8 = (!(~(x & (tmin >> 7)) >> 24)) << 3;
 ans += pt_8;
 x <<= pt_8;
 pt_4 = (!(~(x & (tmin >> 3)) >> 28)) << 2;
 ans += pt_4;
 x <<= pt_4;
 pt_2 = (!(~(x & (tmin >> 1)) >> 30)) << 1;
 ans += pt_2;
 x <<= pt_2;
 pt_1 = (!(~(x & tmin) >> 31));
 ans += pt_1;
 x <<= pt_1;
 is_neg_1 = x >> 31 & 1;
 ans += is_neg_1;
 return ans;
}
```

### 补码运算

#### counter1To5

- 要求：如果 x<5,返回 1+x,否则返回 1
- 允许的操作符：~、&、!、|、+、<<、>>
- 最大操作次数:15
- 题目保证:1 <= x <= 5

```cpp
int counterlto5(int x) {
 bool a = (x & ~5) | (~x & 5);//x异或上5
return x + a + (!a  << 31>> 29);

}
```

#### satMUl3

- 要求：将 x 乘以 3,如果溢出,上 / 下取到 tmin 或 tmax
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:25
  
```cpp
int satMUl3(int x) {
 int newx = x + (x << 1);
 int tmin = 1 << 31;
 int tmax = tmin - 1;
 int sign_x = x >> 31;
 int sign_nx = newx >> 31;
 int flag = sign_x ^ sign_nx;
 return (~flag & newx) | (flag & ((sign_nx & tmax) | (~sign_nx & tmin)));
}
```

#### isGreat

- 要求：如果 x>y,返回 1,否则返回 0
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:24

```cpp
int isGreat(int x, int y) {
 int sign_x = x >> 31;
 int sign_y = y >> 31;
 int sign_not = (!sign_x & sign_y);//符号不同的情况
 int re = x + ~y;//x-y-1>0 就说明x>y
 return sign_not | !(re >> 31)&!(sign_x^sign_y);
}
```

#### subOk

- 要求：如果 x-y 不会溢出，返回 1,否则返回 0
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:20

```cpp
int subOK(int x, int y) {
    // 因为x=y+(x-y)，所以x-y溢出等价于 y 和 (x-y)符号均与x符号不同
    return !(((x ^ y) & (x ^ (x + ~y + 1))) >> 31) & 1;
}
```

#### trueFiveEighths

- 要求：计算 5x/8,向 0 舍入
- 允许的操作符：!、~、&、^、|、+、<<、>>
- 最大操作次数:25
  
```cpp
int trueFiveEights(int x) {
 int fivex = (x << 2) + x;
 
 int a = x >> 31 & 7;
 return (fivex  + (x >> 31 & 7)) >> 3;//当x是负数时，x/2^k=( x+(1<<k)-1) » k
} 
```
