### RSA

`加解密公式:`
$$
ciphertext = plaintext ^ E  mod N
$$

$$
plaintext  = ciphertext ^ D  mod N
$$

由N求p，q [在线网站](https://www.factordb.com/)  或者工具 yafu

#### 题目来源:Hellsream

载入IDA，进入主函数逻辑清晰，这里为主要判断。

![](https://gitee.com/emtanling/image/raw/master/20210429190916.png)

分析sub_4019F0函数:

> ![](https://gitee.com/emtanling/image/raw/master/20210429191044.png)
>
> 运行之后查看内存地址，得到其功能
>
> ![](https://gitee.com/emtanling/image/raw/master/20210429191525.png)
>
> **这里的结构为，为9个4字节，所以之前是9，后面跟9个4字节，即**
>
> ```c
> struct Bigint
> {
>     int BigIntlength = sizeof(BigIntNumber) / 4;
>     DWORD BigIntNumber[BigIntlength]; 
> }
> ```
>
> 

这里判断sn的值是否合法

![](https://gitee.com/emtanling/image/raw/master/20210429191856.png)

将sn的值转换为16进制数

![](https://gitee.com/emtanling/image/raw/master/20210429192001.png)

```
6D6F632E6D736131352E777777 # 运行输出此值，为www.51asm.com转换为16进制逆序输出
```

进入函数sub_401510()

分析sub_4064D0()函数，调试运行到此处可以查看其参数

```c
int __usercall sub_4064D0@<eax>(int *a1@<ecx>, int a2@<ebx>, int *a3@<esi>)
```

int *a1@<ecx>

> ![](https://gitee.com/emtanling/image/raw/master/20210429192809.png)
>
> 此处为0，猜测可能存放，计算后的值，可能为返回值。

int a2@<ebx>

>![](https://gitee.com/emtanling/image/raw/master/20210429192938.png)
>
>此处为Name的16进制值

int *a3@<esi>

> ![](https://gitee.com/emtanling/image/raw/master/20210429193120.png)
>
> 此处为0x11

函数sub_405310()参数为已有值，即功能为初始化。

分析下一个函数

>![](https://gitee.com/emtanling/image/raw/master/20210429202520.png)
>
>int __usercall BigNumPow@<eax>(_DWORD *a1@<edx>, int a2@<ecx>, void *a3@<edi>, int a4, int a5)

DWORD *a1@<edx>

>![](https://gitee.com/emtanling/image/raw/master/20210429202646.png)
>
>此处的值为0x11

int a2@<ecx>

>![](https://gitee.com/emtanling/image/raw/master/20210429203015.png)

void *a3@<edi>

> ![](https://gitee.com/emtanling/image/raw/master/20210429203251.png)
>
> 此处为1可能为返回值

int a4

> ![](https://gitee.com/emtanling/image/raw/master/20210429231146.png)
>
> 多次调试可得此处为e_size

int a5

> ![](https://gitee.com/emtanling/image/raw/master/20210429220806.png)
>
> 地址0x19F2A8
>
> ![](https://gitee.com/emtanling/image/raw/master/20210429220829.png)
>
> 此处为A324F100182D501F6F6F78F397A3AA59641023D6A3DED8A4BF344F1E0FC71C188F4D

运行函数，观察内存地址0x19FAF8

> ![](https://gitee.com/emtanling/image/raw/master/20210429231515.png)
>
> ![](https://gitee.com/emtanling/image/raw/master/20210429231634.png)
>
> 结果一样可得此处为加密运算

之后循环100每次加e每次+1 循环100次 ，进行加密，加密后的结果与"Happy Birthday Buddy\0"的16进制比较。

![](https://gitee.com/emtanling/image/raw/master/20210429232356.png)

```
N = 0xA324F100182D501F6F6F78F397A3AA59641023D6A3DED8A4BF344F1E0FC71C188F4D
p = 0xe2389b3c140be6423ee5eb9db5dac2559f
q = 0xb89eb7e0c9a568202f38b169d9d7e27b93
```

求p、q

![](https://gitee.com/emtanling/image/raw/master/20210426100745.png)

加密脚本

```python
import gmpy2
m_username = 0x6D6F632E6D736131352E777777
e_username = 0x11
N = 0xA324F100182D501F6F6F78F397A3AA59641023D6A3DED8A4BF344F1E0FC71C188F4D
p = 0xe2389b3c140be6423ee5eb9db5dac2559f
q = 0xb89eb7e0c9a568202f38b169d9d7e27b93
D = m_username ** e_username % N
print(hex(D))
def print_string_hex(data):
    data = data[::-1]
    lin = ['%02X' % ord(i) for i in data]
    print("".join(lin))
s = 'Happy Birthday Buddy\0'
print_string_hex(s)
s = b'007964647542207961646874726942207970706148'
# s = b'\0ydduB yadhtriB yppaH'
e_sn = D
phi = (p-1)*(q-1)
E_sn = []
for i in range(100):
    e_tmp = e_sn + i
    try:
        D_sn = gmpy2.invert(e_tmp,phi)
        E_sn.append(D_sn)
    except ZeroDivisionError:
        continue
for i in range(len(E_sn)):
    print(str(hex(E_sn[i]))[2:]) # 输出所有可用的E
print("sn")
s= 0x007964647542207961646874726942207970706148
def pow_mod(p, q, n):
    '''
    幂模运算，快速计算(p^q) mod (n)
    这里采用了蒙哥马利算法
    '''
    res = 1
    while q :
        if q & 1:
            res = (res * p) % n
        q >>= 1
        p = (p * p) % n
    return res
for i in range(len(E_sn)):
    print(hex(pow_mod(int(s),int(E_sn[i]),int(N))))
# 这里为多解,因为 sn = sn mod n ，所以 sn = sn + n * (0,1,2,...) 只要不溢出，都可以
```

![](https://gitee.com/emtanling/image/raw/master/20210429233446.png)

![](https://gitee.com/emtanling/image/raw/master/20210429233542.png)

这里之后的输出可以随意控制，即

![](https://gitee.com/emtanling/image/raw/master/20210429234012.png)

