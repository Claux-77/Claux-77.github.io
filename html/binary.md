### 1.各数据类型与二进制之间的相互转换
+ #### 各类型转字节参考API
```C#
byte[] bytes = Bitcoverter.Getbytes(256);
```
+ #### 字节转各类型API
```C#
int i = Bitcoverter.ToInt32(bytes,0);
```

+ #### 标准编码格式
    + 常见的编码格式有ASCII，ANSI，GBK，GB2312，UTF-8，UNITCODE等
    + 数据的本质是二进制数据，编码格式就是用对应的二进制的数据对应不同的文字
    + 存储与读取数据时须使用统一的编码格式，否则可能出现乱码
    + C#中的专门编码格式类（Encoding），来帮助我们进行字符串与字节数组进行转换
    + 代码如下
```C#
byte[] bytes2 = Encoding.UTF8.GetBytes("可爱");
string s2 = Encoding.UTF8.GetString(bytes2);
 ```
 
### 2.文件相关操作，C#中主要是File类
+ #### 判断文件是否存在
```
File.Exists(Application.dataPath + "/teach.cl");
```
+ #### 创建文件，返回值是文件流
```
 FileStream fs = File.Create(Application.dataPath + "/teach.cl");
```

+ #### 写入文件。注意写入文件和创建文件并不是一回事
```
File.WriteAllBytes(Application.dataPath + "/teach.cl",bytes); //写入字节数组
File.WriteAllLines(Application.dataPath + "/teach.cl",strs); //写入字符串数组，会自动换行
File.WriteAllText(Application.dataPath+ "/teach.cl","CC"); //写入指定字符串
```

+ #### 读取文件
```
bytes = File.ReadAllBytes(Application.dataPath + "/teach.cl"); //读取指定字节数组
//其他api类似写入
```

+ #### 删除文件。注意不能删除正在打开的文件
```
File.delete(...);
```

+ #### 复制文件，强制覆盖目标文件
```
File.Copy(source,denstination,true);
```
+ #### 文件替换，可备份文件
```
File.Replace();
```

+ ### 以流的形式，打开文件写入或者读取
```
//访问文件路径
//打开模式
//访问模式
 FileStream fs1 = File.Open(Application.dataPath + "/teach.cl", FileMode.OpenOrCreate, FileAccess.ReadWrite);
```

### 3.文件流相关
+ #### 文件流是什么
    + 文件里面存储的数据就像是一条数据流
    + 可以通过FileStream一部分一部分的读写数据流
    + 利用FileStream可以以流式逐个读写

+ #### 打开或创建指定文件
    + new FileStream()
    + File.Create()
    + File.Open()
    
+ #### 重要属性和方法
```
fs.Length;
fs.CanRead;
fs.CanWrite;

fs.Flush(); //将字节写入文件，写入后，执行一次
fs.Close(); //关闭流
fs.Dispose();//缓存资源回收
```

+ #### 写入字节。写入字符串字节数组时，先写入其长度，再写入其内容。
```
fs1.Write(bytes,0,4);  //写入的字节数组，数组的开始索引，写入多少字节
```

+ #### 读取字节。读取字符串字节数组时，先读取长度
```
//存储读取的字节数组容器
//容器开始的位置
//读取多少个字节
//返回值：当前流索引前进了几个位置。记录此值方便后续继续读取
fs2.Read(bytes2,0,4)
```

### 4.文件夹
+ #### C#提供了相关的文件夹操作公共类Directory。相关的API与File类类似


### 5.C#的序列化和反序列化
+ #### 使用C#自带的序列化二进制的方法
+ 1. 声明类时要添加[System.Serializable]特性
```C#
[System.Serializable]
public class Person {...}
``````
+ 2. 将对象进行二进制序列化
    + 使用内存流
    ```
    Person person = new Person(23,"AA");

    using (MemoryStream ms = new MemoryStream())
    {
        BinaryFormatter bf = new BinaryFormatter(); //二进制格式化类
        bf.Serialize(ms, person);       //序列化
        byte[] bytes1 = ms.GetBuffer(); //获取字节数组
        File.WriteAllBytes(Application.dataPath+"/teach1.cl",bytes); //将字节数组写入文件
        ms.Close(); //关闭内存流
    }
    ```
    + 使用文件流，相对内存流方式要简洁一点，但是如果要将字节数组进行网络传输时，得使用内存流
    ```
    using (FileStream fs1 = new FileStream(Application.dataPath + "/teach2.cl",FileMode.OpenOrCreate,FileAccess.Write))
    {
        BinaryFormatter bf = new BinaryFormatter();
        bf.Serialize(fs1,person);
        fs.Flush();
        fs.Close();
    }
    ```

+ #### 使用C#自带的反序列化二进制的方法
+ 1. 反序列化文件中的二进制数据
```
using (FileStream fs2 = File.Open(Application.dataPath + "/teach2.cl",FileMode.Open,FileAccess.Read))
{
    BinaryFormatter bf = new BinaryFormatter();
    Person person1 = bf.Deserialize(fs2) as Person;
                
    fs2.Close();
}
```
+ 2. 反序列化网络传输中的二进制数据
```
//假定这就是网络传输过来的二进制数据
byte[] bytes1 = File.ReadAllBytes(Application.dataPath + "/teach2.cl");
using (MemoryStream ms = new MemoryStream(bytes1))
{
    BinaryFormatter bf = new BinaryFormatter();
    Person person2 = bf.Deserialize(ms) as Person;
                
    ms.Close();
}
```  

### 6. C#类对象的二进制数据加密
+ #### 常用加密算法：[MD5](https://www.cnblogs.com/foxclever/p/7668369.html)，[SHA1](https://www.cnblogs.com/scu-cjx/p/6878853.html)，[HMAC](https://www.cnblogs.com/shoshana-kong/p/11497676.html)，AES/DES/3DES等
+ #### 实现一个简单的异或加密
    + 与密钥进行异或来进行简单的加密和解密
    + coding
    ```
    //使用异或进行简单的加密和解密
    Person person3 = new Person(23,"Tom");
    byte key = 200;
    using (MemoryStream ms = new MemoryStream())
    {
        BinaryFormatter bf = new BinaryFormatter();
        bf.Serialize(ms,person3);

        byte[] bytes3 = ms.GetBuffer();
        //进行异或
        for (int i = 0; i < bytes3.Length; i++)
        {
            bytes3[i] ^= key;
        }
        File.WriteAllBytes(Application.dataPath+"/teach_cry.cl",bytes3);
    }
    ```