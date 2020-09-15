### rsa

**读取公钥**

```
-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAMAzLFxkrkcYL2wch21CM2kQVFpY9+7+
/AvKr1rzQczdAgMBAAE=
-----END PUBLIC KEY-----
```

[解析公钥](http://tool.chacuo.net/cryptrsakeyparse)(提取 e，n)

![](https://gitee.com/emtanling/ctf_-re/raw/master/images/rsa_1.png)

**获取指数及模数**

```
e = 65537    
n=86934482296048119190666062003494800588905656017203025617216654058378322103517(模数转换为十进制)
```

[解析公钥](http://www.factordb.com/)(提取p，q)

![](https://gitee.com/emtanling/ctf_-re/raw/master/images/rsa_2.png)

```
p = 285960468890451637935629440372639283459
q = 304008741604601924494328155975272418463
```

**代码**

```python
import gmpy2
import rsa

e = 65537
n = 86934482296048119190666062003494800588905656017203025617216654058378322103517
p = 285960468890451637935629440372639283459
q = 304008741604601924494328155975272418463

phin = (q-1)*(p-1)
d = gmpy2.invert(e, phin)

key = rsa.PrivateKey(n, e, int(d), p, q)

with open("flag.enc", "rb+") as f:
    f = f.read()
    print(rsa.decrypt(f, key))
```

[rsa详细内容](https://err0rzz.github.io/2017/11/14/CTF%E4%B8%ADRSA%E5%A5%97%E8%B7%AF/)



### CrackRTF

载入ida找到main函数

```c
int __cdecl main_0()
{
  DWORD v0; // eax
  DWORD v1; // eax
  CHAR String; // [esp+4Ch] [ebp-310h]
  int v4; // [esp+150h] [ebp-20Ch]
  CHAR String1; // [esp+154h] [ebp-208h]
  BYTE pbData; // [esp+258h] [ebp-104h]

  memset(&pbData, 0, 0x104u);
  memset(&String1, 0, 0x104u);
  v4 = 0;
  printf("pls input the first passwd(1): ");
  scanf("%s", &pbData);
  if ( strlen((const char *)&pbData) != 6 )
  {
    printf("Must be 6 characters!\n");
    ExitProcess(0);
  }
  v4 = atoi((const char *)&pbData);
  if ( v4 < 100000 )
    ExitProcess(0);
  strcat((char *)&pbData, "@DBApp");
  v0 = strlen((const char *)&pbData);
  sub_40100A(&pbData, v0, &String1);
  if ( !_strcmpi(&String1, "6E32D0943418C2C33385BC35A1470250DD8923A9") )
  {
    printf("continue...\n\n");
    printf("pls input the first passwd(2): ");
    memset(&String, 0, 0x104u);
    scanf("%s", &String);
    if ( strlen(&String) != 6 )
    {
      printf("Must be 6 characters!\n");
      ExitProcess(0);
    }
    strcat(&String, (const char *)&pbData);
    memset(&String1, 0, 0x104u);
    v1 = strlen(&String);
    sub_401019((BYTE *)&String, v1, &String1);
    if ( !_strcmpi("27019e688a4e62a649fd99cadaafdb4e", &String1) )
    {
      if ( !(unsigned __int8)sub_40100F(&String) )
      {
        printf("Error!!\n");
        ExitProcess(0);
      }
      printf("bye ~~\n");
    }
  }
  return 0;
}
```

输入6个字符

23行将"@DBApp"插到输入的字符后面

第25行进入函数sub_40100A

```c
int __cdecl sub_401230(BYTE *pbData, DWORD dwDataLen, LPSTR lpString1)
{
  int result; // eax
  DWORD i; // [esp+4Ch] [ebp-28h]
  CHAR String2; // [esp+50h] [ebp-24h]
  BYTE v6[20]; // [esp+54h] [ebp-20h]
  DWORD pdwDataLen; // [esp+68h] [ebp-Ch]
  HCRYPTHASH phHash; // [esp+6Ch] [ebp-8h]
  HCRYPTPROV phProv; // [esp+70h] [ebp-4h]

  if ( !CryptAcquireContextA(&phProv, 0, 0, 1u, 0xF0000000) )
    return 0;
  if ( CryptCreateHash(phProv, 0x8004u, 0, 0, &phHash) )
  {
    if ( CryptHashData(phHash, pbData, dwDataLen, 0) )
    {
      CryptGetHashParam(phHash, 2u, v6, &pdwDataLen, 0);
      *lpString1 = 0;
      for ( i = 0; i < pdwDataLen; ++i )
      {
        wsprintfA(&String2, "%02X", v6[i]);
        lstrcatA(lpString1, &String2);
      }
      CryptDestroyHash(phHash);
      CryptReleaseContext(phProv, 0);
      result = 1;
    }
    else
    {
      CryptDestroyHash(phHash);
      CryptReleaseContext(phProv, 0);
      result = 0;
    }
  }
  else
  {
    CryptReleaseContext(phProv, 0);
    result = 0;
  }
  return result;
}
```

这是哈希加密 sha1加密算法

>  可以md5
>
> ![image](https://gitee.com/emtanling/ctf_-re/raw/master/images/crackrtf1.png)

 也可以爆破

```python
import hashlib
string='@DBApp'
for i in range(100000,999999):
    flag=str(i)+string
    x=hashlib.sha1(flag.encode("utf8"))
    y=x.hexdigest()
    if "6e32d0943418c2c33385bc35a1470250dd8923a9" == y:
        print("password.1= "+flag)
        break
```

37行将再次输入的6个字符添加到"123321@DBApp"之后

> 可以再次md5
>
> ![image](https://gitee.com/emtanling/ctf_-re/raw/master/images/crackrtf2.png)

得到“~!3a@0”

也可以进入函数sub_40100F中sub_4014D0函数

```c
char __cdecl sub_4014D0(LPCSTR lpString)
{
  LPCVOID lpBuffer; // [esp+50h] [ebp-1Ch]
  DWORD NumberOfBytesWritten; // [esp+58h] [ebp-14h]
  DWORD nNumberOfBytesToWrite; // [esp+5Ch] [ebp-10h]
  HGLOBAL hResData; // [esp+60h] [ebp-Ch]
  HRSRC hResInfo; // [esp+64h] [ebp-8h]
  HANDLE hFile; // [esp+68h] [ebp-4h]

  hFile = 0;
  hResData = 0;
  nNumberOfBytesToWrite = 0;
  NumberOfBytesWritten = 0;
  hResInfo = FindResourceA(0, (LPCSTR)0x65, "AAA"); // 定位所指定的资源
    												// 第一个参数包含所需资源的模块句柄为0即程序本身
    												// 第二个参数资源名称 0x65
    												// 第三个参数资源类型 "AAA"
    												// 从"AAA"中查到0x65这个资源并将内存中地址返回给hResInfo
  if ( !hResInfo )
    return 0;										// 如果没有找到资源则返回0
  nNumberOfBytesToWrite = SizeofResource(0, hResInfo);// 获取资源大小
  hResData = LoadResource(0, hResInfo);				// 加载到内存中
  if ( !hResData )									
    return 0;										// 如果没有加载成功返回0
  lpBuffer = LockResource(hResData);				// IpBuffer为指向hResData
  sub_401005(lpString, (int)lpBuffer, nNumberOfBytesToWrite);
  hFile = CreateFileA("dbapp.rtf", 0x10000000u, 0, 0, 2u, 0x80u, 0);//创建一个文件
  if ( hFile == (HANDLE)-1 )
    return 0;										
  if ( !WriteFile(hFile, lpBuffer, nNumberOfBytesToWrite, &NumberOfBytesWritten, 0) )
    return 0;										// 如果写文件错误返回0
  CloseHandle(hFile);
  return 1;											// 正确返回1
}
```

分析sub_401005函数

```c
unsigned int __cdecl sub_401420(LPCSTR lpString, int a2, int a3)
{
  unsigned int result; // eax
  unsigned int i; // [esp+4Ch] [ebp-Ch]
  unsigned int v5; // [esp+54h] [ebp-4h]

  v5 = lstrlenA(lpString);
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i >= a3 )
      break;
    *(_BYTE *)(i + a2) ^= lpString[i % v5];
  }
  return result;
}
```

查看资源

![image](https://gitee.com/emtanling/ctf_-re/raw/master/images/crackrtf3.png)

```
05 7D 41 15 26 01
```

查看第30行可以得到资源文件与密码的异或结果是rtf标志位，而rtf标志位是

```
7B 5C 72 74 66 31
```

所以

```python
rtf = [0x7B,0x5C,0x72,0x74,0x66,0x31]
A = [0x05, 0x7D, 0x41, 0x15, 0x26, 0x01]
password=''
for i in range(len(rtf)):
    x = rtf[i] ^ A[i]
    password+=chr(x)
print("password.2= "+password)

```

得到密码输入即可



### Java逆向解密

反编译得到文件

```java
import java.util.ArrayList;
import java.util.Scanner;

public class Reverse {

   public static void main(String[] args) {
      Scanner s = new Scanner(System.in);
      System.out.println("Please input the flag ");
      String str = s.next();
      System.out.println("Your input is ");
      System.out.println(str);
      char[] stringArr = str.toCharArray();
      Encrypt(stringArr);
   }

   public static void Encrypt(char[] arr) {
      ArrayList Resultlist = new ArrayList();

      for(int KEY = 0; KEY < arr.length; ++KEY) {
         int KEYList = arr[KEY] + 64 ^ 32;
         Resultlist.add(Integer.valueOf(KEYList));
      }

      int[] var5 = new int[]{180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65};
      ArrayList var6 = new ArrayList();

      for(int j = 0; j < var5.length; ++j) {
         var6.add(Integer.valueOf(var5[j]));
      }

      System.out.println("Result:");
      if(Resultlist.equals(var6)) {
         System.out.println("Congratulations");
      } else {
         System.err.println("Error");
      }

   }
}

```

可以写出

```python
flagy=[180, 136, 137, 147, 191, 137, 147, 191, 148, 136, 133, 191, 134, 140, 129, 135, 191, 65]
flag=""
for i in range(0,len(flagy)):
    flag+=chr(flagy[i]-64^32)
print(flag)
```



### JustRE

载入IDA

![image](https://gitee.com/emtanling/ctf_-re/raw/master/images/justre_1.png)

由第14行的sprintf得flag

```
flag{1999902069a45792d233ac}
```

