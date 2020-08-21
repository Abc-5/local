

## [creakme程序下载](https://emtanling.lanzous.com/ihhabdkhyze)

### [Windows下常用调试API断点](https://www.cnblogs.com/LyShark/p/13071792.html)

## creakme1

#### 1. 运行



![image](https://gitee.com/emtanling/image/raw/master/img/creakme1_run.png)

​																					    creakme1-1

![image](https://gitee.com/emtanling/image/raw/master/img/creakme1_run2.png)

​																						creakme1-2

#### 2. 载入 OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme1_od.png)

> 一个主程序窗口即图像creakme1-1，判断决定弹出哪个窗口。

* 在00401026处执行判断语句

* je 表示等于就跳转，jnz是不等于就跳转。

    

#### 3. 修改为jnz (也可以直接跳转，修改为jmp)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme1_od1.png)



#### 4. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme1_run3.png)



## creakme2

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_run1.png)

#### 2. 载入 OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2.od1.png)



```ml
00401232  FF25  A0104000  jmp dword ptr ds:[0x4010A0]  ;  MSVBVM60.ThunRTMain
00401238  68  141E4000    push 0x401E14				   ;  = EP
0040123D  E8  F0FFFFFF    call 00401232				   ;  JMP.&MSVBVM60.#100
```

* 以上三行是VB文件的全部启动代码

#### 3.  检索字符串

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_od2.png)

> 双击字符串进行跳转 (Wrong serial!)

**可以猜测这是通过某种比较进行判断**

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_od3.png)

#### 4. 寻找字符串地址

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_od4.png)

> * 00403329地址处的 __vbaVarTstEq() 函数为字符串比较函数 
>
> * 在00403329地址处设置断点
> * 查看edx和eax的地址可以得到如下图
> * Name: name   Serial: D2C5D1C9

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_od5.png)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_run2.png)

#### 5. Serial算法

##### 1. 查找函数开始部分

> 向上一点点查找

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2._s2.png)

* 00402ED0 是栈帧代码，执行函数就会形成栈帧，可以猜测此处为函数开始部分，即check按钮处理事件。将00402ED0设置断点。

##### 2. 预测代码

   **若是win32 api 程序**

* 读取Name字符串(GetWindowsText, GetDIgItemText等API)
* 启动循环对字符加密(XOR, ADD, SUB等等)

##### 3. 读取Name字符串的代码

* 调试完成00402FB6,寄存器EAX的值为 输入Name的字符串 name 。
* 可以推断[EBP-0x88]，即为字符串变量

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_s33.png)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_s4.png)



* 00402F8E传递此地址，为保存Name字符串的字符串对象
* 00402F98为获取Name字符串

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_s5.png)

**验证**

> 重新调试，观察栈窗口当运行了00402F98 call命令后[EBP-88]的值为Name字符串。

##### 5. 加密

* 加密循环语句

>__vbaVarForInit()函数

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_x1.png)

> __vbaVarForNext()函数

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_x2.png)

* 加密方法

输入字符串为 "name"

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y1.png)

查看可以知道保存一个字符

* 调试结束0040323D地址的指令

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y2.png)

* 查看内存地址

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y3.png)

>* EAX 保存 d 即为 密钥 64
>
>* ECX 保存内容的区域是用于保存结果的缓冲区
>* call 命令是进行加法的 即 64 + 6e = D2

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y4.png)

下面代码将生成的字符串连接起来，__vbaVarCat()函数作用是将字符串进行连接。

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y5.png)

* 调试到 0040327B

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y6.png)

* F9 再次执行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme2_y8.png)

> 1. EAX和EDX存放第二个字符
> 2. ECX存放上一个字符

##### 6. 加密方法

> 1、从输入的Name字符串前端逐一读取字符(共4次);
>
> 2、将字符转换为数字(Unicode>ASCII);
>
> 3、向变换后的数字加64;
>
> 4、将数字转换为字符(ASCII>Unicode);
>
> 5、连接变换后的字符即为Serial序列号。

##### 7. 生成Serial

```cpp
#include <string>
#include <iostream>

using namespace std;

int main()
{
    string str;
    cout << "请输入Name：" << endl;
    cin >> str;
    cout << "你的Serial为：" << endl;
    cout << uppercase;
    for (int i = 0; i < 4; ++i) {
        cout << hex << (int)str[i] + 0x64;
    }
    cout << nouppercase;

    system("PAUSE");
    return 0;
}
```

### creakme3

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme3_run1.png)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme3_run2.png)

> 提示缺少密钥文件

#### 2. 载入OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme3_od1.png)



>* **MessageBoxA()** 和 **CreateFileA()** 两个函数

[CreateFileA()](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea)

```c++
HANDLE CreateFileA(
  LPCSTR                lpFileName,
  DWORD                 dwDesiredAccess,
  DWORD                 dwShareMode,
  LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  DWORD                 dwCreationDisposition,
  DWORD                 dwFlagsAndAttributes,
  HANDLE                hTemplateFile
);
```

> * 此函数可以创建或打开文件。

![image](https://gitee.com/emtanling/image/raw/master/img/creakme3_od2.png)

> * OPEN_EXISTING，打开现有文件
> * GENERIC_READ，打开文件进行读取
> * FileName参数，打开了一个名为" **abex.l2c** "

GetFileSize() 函数将 abex.l2c 文件的长度返回给 eax

![image](https://gitee.com/emtanling/image/raw/master/img/creakme3_od3.png)

```ml
00401046	cmp eax,0x12
```

判断eax的值和0x12是否相等

#### 3. 结论

> 在通路径下创建文件 **abex.l2c** 并使其的长度为 0x12 即有18个字符。

```abex.l2c 内容
aaaaaaaaaaaaaaaaaa
```

### creakme4

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_run1.png)

> Registered 无法使用

#### 2. 载入OD

> 查找字符串并未有结果
>
> 查找函数

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_od2.png)

> * 发现有函数 __vbaStrCmp()
> * 从 0040230D 位置开始

从此函数的开始部分，00402280 设置断点开始调试

当调试到 0040230C 位置

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_od3.png)

* ECX为UNICODE “2100800”

* 查看EAX的内存，存储输入的字符 'a' 。可能比较EAX和ECX

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_od4.png)

* 输入ECX的值成功

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_x1.png)

#### 3. 生成序列号

![image](https://gitee.com/emtanling/image/raw/master/img/creakme4_y1.png)

> 1. 函数有GetHourOFDay() , Getyear() 获得年，小时。(00402156 和 00402160 分别获得)
> 2. 0040216F 为计算方法即 (小时 * 5 + 1000) * 年 

### creakme5

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme5_r1.png)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme5_r2.png)

#### 2. 载入OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme5_od1.png)

> 004010FC EAX必须为 0 

[lstrcmpi](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-lstrcmpia)

```c++
int lstrcmpiA(
  LPCSTR lpString1,
  LPCSTR lpString2
);
```

返回值应当为0，所以输入

```ml
L2C-57816784-ABEX
```

![image](https://gitee.com/emtanling/image/raw/master/img/creakme5_y1.png)

### creakme6

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme6_r1.png)

#### 2. 载入OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme6_od1.png)

```ml
00401555 cmp dword ptr ss:[ebp-0x4],0x7F97E56C 
```

![image](https://gitee.com/emtanling/image/raw/master/img/creakme6_od2.png)

ebp-4 的值为输入的值

输入 2140661100

![image](https://gitee.com/emtanling/image/raw/master/img/creakme6_y1.png)

### creakme7

#### 1. 运行

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_r1.png)

#### 2. 载入OD

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od1.png)

```ml
__vbaVarCmpEq()  // 比较
__vbaVarCmpNe()  // 比较
```

**在此函数下断点**

查看堆栈直接显示

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od2.png)

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_y1.png)

#### 3. Serial算法

在调用比较函数的栈帧处开始调试

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od3.png)

运行至此处 EDX 为 Serial 值

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od4.png)

进入 004029DC 

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od5.png)

继续调试

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od6.png)

调试到此处

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od7.png)

看到字符串 "TWVQ"

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od8.png)

继续调试

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od9.png)

调试至 4024EA 此为 "TWVQ" 生成方法

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od10.png)

将 字符 a 和 0x35 进行xor运算得到 T，循环四次

![image](https://gitee.com/emtanling/image/raw/master/img/creakme7_od11.png)

最后在转换成 16进制码

```ml
(0x61 xor 0x35)
```

#### 4.生成Serial

```c
#include<stdio.h>
int main()
{
	printf("输入Name: ");
	char name[30];
	scanf("%s", name);
	printf("Serial= ");
	for (int i = 0; name[i] != '\0'; i++) {
		printf("%x", int(name[i]) xor 0x35);
	}
	return 0;
}
```

