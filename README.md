一.日志

本文介绍的是开发框架中日志文件操作的方法。

开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 1.**日志类：CLogfile （封装）**

```c++
// 日志文件操作类
class CLogFile
{
public:
  FILE   *m_tracefp;           // 日志文件指针。
  char    m_filename[301];     // 日志文件名，建议采用绝对路径。
  char    m_openmode[11];      // 日志文件的打开方式，一般采用"a+"。
  bool    m_bEnBuffer;         // 写入日志时，是否启用操作系统的缓冲机制，缺省不启用。
  long    m_MaxLogSize;        // 最大日志文件的大小，单位M，缺省100M。
  bool    m_bBackup;           // 是否自动切换，日志文件大小超过m_MaxLogSize将自动切换，缺省启用。

  // MaxLogSize：最大日志文件的大小，单位M，缺省100M，最小为10M。
  CLogFile(const long MaxLogSize=100); 

  // 打开日志文件。
  // filename：日志文件名，建议采用绝对路径，如果文件名中的目录不存在，就先创建目录。
  // openmode：日志文件的打开方式，与fopen库函数打开文件的方式相同，缺省值是"a+"。
  // bBackup：是否自动切换，true-切换，false-不切换，在多进程的服务程序中，如果多个进行共用一个日志文件，bBackup必须为false。
  // bEnBuffer：是否启用文件缓冲机制，true-启用，false-不启用，如果启用缓冲区，那么写进日志文件中的内容不会立即写入文件，缺省是不启用。
  bool Open(const char *filename,const char *openmode=0,bool bBackup=true,bool bEnBuffer=false);

  // 如果日志文件大于100M，就把当前的日志文件备份成历史日志文件，切换成功后清空当前日志文件的内容。
  // 备份后的文件会在日志文件名后加上日期时间。
  // 注意，在多进程的程序中，日志文件不可切换，多线的程序中，日志文件可以切换。
  bool BackupLogFile();

  // 把内容写入日志文件，fmt是可变参数，使用方法与printf库函数相同。
  // Write方法会写入当前的时间，WriteEx方法不写时间。
  bool Write(const char *fmt,...);
  bool WriteEx(const char *fmt,...);

  // 关闭日志文件
  void Close();

  ~CLogFile();  // 析构函数会调用Close方法。
};

```

## 2.日志文件切换

```c++
/*
 *  程序名：demo43.cpp，此程序演示开发框架的CLogFile类的日志文件的切换。
*/
#include "../_public.h"

int main()
{
  CLogFile logfile;

  // 打开日志文件，如果"/tmp/log"不存在，就创建它，但是要确保当前用户具备创建目录的权限。
  if (logfile.Open("/tmp/log/demo43.log")==false)
  { printf("logfile.Open(/tmp/log/demo43.log) failed.\n"); return -1; }

  logfile.Write("demo43程序开始运行。\n");

  // 让程序循环10000000，生成足够大的日志。
  for (int ii=0;ii<10000000;ii++)
  {
    logfile.Write("本程序演示日志文件的切换，这是第%010d条记录。\n",ii);
  }

  logfile.Write("demo43程序运行结束。\n");
}

```

​		设置切片大小，在日志大于10M时切片--新开一个文件存储。

提供：**打开日志，写日志，切换日志文件，关闭日志，开关写日期**  等方法

## 3.站点参数

容器存储参数

```c++
LoadSTCode(const char *inifile)
{
    while(true)
    {
        //从站点读取一行，读取完跳出循环
        //把读取到的一行拆分
        //把站点参数的每个数据项保存到结构体中
        //把结构体放入容器中
    }
}
```

# 二.文件操作

## 1、源代码说明

本文介绍的是开发框架的文件操作的函数和类。

开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 2、文件操作函数

### 1)、删除文件

删除目录中的文件，类似Linux系统的rm命令。

```c++
函数声明：
bool REMOVE(const char *filename,const int times=1);

参数说明：
filename：待删除的文件名，建议采用绝对路径的文件名，例如/tmp/root/data.xml。
times：执行删除文件的次数，缺省是1，建议不要超过3，从实际应用的经验看来，如果删除文件第1次不成功，再尝试2次是可以的，更多次就意义不大了。还有，如果执行删除失败，usleep(100000)后再重试。
    
返回值：true-删除成功；false-删除失败，失败的主要原因是权限不足。
在应用开发中，可以用REMOVE函数代替remove库函数。
```



### 2)、文件重命名

把文件重命名，类似Linux系统的mv命令。

```c++
函数声明：

bool RENAME(const char *srcfilename,const char *dstfilename,const int times=1);

参数说明：

srcfilename：原文件名，建议采用绝对路径的文件名。

destfilename：目标文件名，建议采用绝对路径的文件名。

times：执行重命名文件的次数，缺省是1，建议不要超过3，从实际应用的经验看来，如果重命名文件第1次不成功，再尝试2次是可以的，更多次就意义不大了。还有，如果执行重命名失败，usleep(100000)后再重试。

 返回值：true-重命名成功；false-重命名失败，失败的主要原因是权限不足或磁盘空间不够，如果原文件和目标文件不在同一个磁盘分区，重命名也可能失败。

注意，在重命名文件之前，会自动创建destfilename参数中的目录名。

在应用开发中，可以用RENAME函数代替rename库函数。
```



### 3)、复制文件

复制文件，类似Linux系统的cp命令。

```c++
函数声明：

bool COPY(const char *srcfilename,const char *dstfilename);

参数说明：

srcfilename：原文件名，建议采用绝对路径的文件名。

destfilename：目标文件名，建议采用绝对路径的文件名。

返回值：true-复制成功；false-复制失败，失败的主要原因是权限不足或磁盘空间不够。

注意：

1）在复制名文件之前，会自动创建destfilename参数中的目录名。

2）复制文件的过程中，采用临时文件命名的方法，复制完成后再改名为destfilename，避免中间状态的文件被读取。

3）复制后的文件的时间与原文件相同，这一点与Linux系统cp命令不同。
```



### 4)、获取文件的大小

函数声明：

```c++
int FileSize(const char *filename);

参数说明：

filename：待获取的文件名，建议采用绝对路径的文件名。

返回值：如果文件不存在或没有访问权限，返回-1，成功返回文件的大小，单位是字节。
```



### 5)、获取文件的时间

函数声明：

```c++
bool FileMTime(const char *filename,char *mtime,const char *fmt=0);

参数说明：

filename：待获取的文件名，建议采用绝对路径的文件名。

mtime：用于存放文件的时间，即stat结构体的st_mtime。

fmt：设置时间的输出格式，与LocalTime函数相同，但缺省是"yyyymmddhh24miss"。

返回值：如果文件不存在或没有访问权限，返回false，成功返回true。
```



### 6)、重置文件的时间

函数声明：

```c++
int UTime(const char *filename,const char *mtime);

参数说明：

filename：待重置的文件名，建议采用绝对路径的文件名。

stime：字符串表示的时间，格式不限，但一定要包括yyyymmddhh24miss，一个都不能少。

返回值：true-成功；false-失败，失败的原因保存在errno中。
```



### 7)、示例程序

**示例（demo34.cpp）**

```c++
/*
 \* 程序名：demo34.cpp，此程序演示开发框架的文件操作函数的用法
*/

\#include "../_public.h"

int main()
{

 // 删除文件。
 if (REMOVE("/tmp/root/_public.h")==false)
 {
  printf("REMOVE(/tmp/root/_public.h) %d:%s\n",errno,strerror(errno));
 }

 // 重命名文件。
 if (RENAME("/tmp/root/_public.cpp","/tmp/root/aaa/bbb/ccc/_public.cpp")==false)
 {
  printf("RENAME(/tmp/root/_public.cpp) %d:%s\n",errno,strerror(errno));
 }
    
 // 复制文件。
 if (COPY("/project/public/_public.h","/tmp/root/aaa/bbb/ccc/_public.h")==false)

 {
  printf("COPY(/project/public/_public.h) %d:%s\n",errno,strerror(errno));
 }

 // 获取文件的大小。
 printf("size=%d\n",FileSize("/project/public/_public.h"));

 // 重置文件的时间。
 UTime("/project/public/_public.h","2020-01-05 13:37:29");

 // 获取文件的时间。
 char mtime[21]; memset(mtime,0,sizeof(mtime));
 FileMTime("/project/public/_public.h",mtime,"yyyy-mm-dd hh24:mi:ss");
 printf("mtime=%s\n",mtime);  // 输出mtime=2020-01-05 13:37:29

}
```



### 8)、打开文件

函数声明：

```c
FILE *FOPEN(const char *filename,const char *mode);

参数说明：

FOPEN函数调用fopen库函数打开文件，如果文件名中包含的目录不存在，就创建目录。

FOPEN函数的参数和返回值与fopen函数完全相同。

在应用开发中，用FOPEN函数代替fopen库函数。
```



### 9)、读取文件

从文本文件中读取一行。

```
函数声明：

bool FGETS(const FILE *fp,char *buffer,const int readsize,const char *endbz=0);

参数说明：

fp：已打开的文件指针。

buffer：用于存放读取的内容。

readsize：本次打算读取的字节数，如果已经读取到了结束标志，函数返回。

endbz：行内容结束的标志，缺省为空，表示行内容以"\n"为结束标志。

返回值：true-成功；false-失败，一般情况下，失败可以认为是文件已结束。
```



### 10)、示例程序

**示例（demo36.cpp）**

```c++
/*
 \* 程序名：demo36.cpp，此程序演示开发框架中FOPEN函数的用法。
*/

\#include "../_public.h"

int main()
{
 FILE *fp=0;

 // 用FOPEN函数代替fopen库函数，如果目录/tmp/aaa/bbb/ccc不存在，会创建它。
 if ( (fp=FOPEN("/tmp/aaa/bbb/ccc/tmp.xml","w"))==0)  
 {
  printf("FOPEN(/tmp/aaa/bbb/ccc/tmp.xml) %d:%s\n",errno,strerror(errno)); return -1;
 }

 // 向文件中写入两行超女数据。
 fprintf(fp,"<data>\n"\
   "<name>妲已</name><age>28</age><sc>火辣</sc><yz>漂亮</yz><memo>商朝要亡，关我什么事。</memo><endl/>\n"\
   "<name>西施</name><age>25</age><sc>火辣</sc><yz>漂亮</yz><memo>1、中国排名第一的美女；\n"\
   "2、男朋友是范蠡；\n"\
   "3、老公是夫差，被勾践弄死了。</memo><endl/>\n"\
   "</data>\n");

 fclose(fp); // 关闭文件。

}
```

demo36.cpp程序生成了数据文件/tmp/aaa/bbb/ccc/tmp.xml，数据内容如下：

![image-20220716185353918](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716185353918.png)

以上数据文件只有两条有效的记录，但是第二条数据是跨多行的，在memo标签里，文字和换行符都是内容的一部分。

**示例（demo37.cpp）**

```c++
/*
 \* 程序名：demo37.cpp，此程序演示开发框架中FGETS函数的用法。
*/
\#include "../_public.h"
 
int main()
{
 FILE *fp=0;

 if ( (fp=FOPEN("/tmp/aaa/bbb/ccc/tmp.xml","r"))==0)
 {
  printf("FOPEN(/tmp/aaa/bbb/ccc/tmp.xml) %d:%s\n",errno,strerror(errno)); return -1;
 }

 char strBuffer[301];
 
 while (true)

 {
  memset(strBuffer,0,sizeof(strBuffer));
  if (FGETS(fp,strBuffer,300)==false) break;   // 行内容以"\n"结束。
  //if (FGETS(fp,strBuffer,300,"<endl/>")==false) break; // 行内容以"<endl/>"结束。
  printf("strBuffer=%s",strBuffer);
 }

 fclose(fp);
}
```

**运行效果**

 ![image-20220716185413656](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716185413656.png)

第二条记录的内容不完整，这并不是程序员想要的结果，如果demo37.cpp启用以下代码：

  if (FGETS(fp,strBuffer,300,"<endl/>")==false) break; // 行内容以"<endl/>"结束。

**运行效果**

 ![image-20220716185419615](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716185419615.png)

这才是程序员想要的结果。



## 3、CFile类

CFile类根据实际开发中的应用场景，对常用的文件操作功能做了封装。

我们来先介绍类的声明，然后再列出常用应用场景的示例。

### 1)、类的声明

// 文件操作类声明

```c++
class CFile

{

private:

 FILE *m_fp;    // 文件指针

 bool m_bEnBuffer; // 是否启用缓冲，true-启用；false-不启用，缺省是启用。

 char m_filename[301]; // 文件名，建议采用绝对路径的文件名。

 char m_filenametmp[301]; // 临时文件名，在m_filename后加".tmp"。


public:

 CFile();  // 类的构造函数。
 ~CFile();  // 类的析构函数。

 bool IsOpened(); // 判断文件是否已打开，返回值：true-已打开；false-未打开。

 // 打开文件。
 // filename：待打开的文件名，建议采用绝对路径的文件名。
 // openmode：打开文件的模式，与fopen库函数的打开模式相同。
 // bEnBuffer：是否启用缓冲，true-启用；false-不启用，缺省是启用。
 // 注意：如果待打开的文件的目录不存在，就会创建目录。
 bool Open(const char *filename,const char *openmode,bool bEnBuffer=true);

 // 关闭文件指针，并删除文件。
 bool CloseAndRemove();
 

 // 专为重命名而打开文件，参数与Open方法相同。
 // 注意：OpenForRename打开的是filename后加".tmp"的临时文件，所以openmode只能是"a"、"a+"、"w"、"w+"。
 bool OpenForRename(const char *filename,const char *openmode,bool bEnBuffer=true);

 // 关闭文件指针，并把OpenForRename方法打开的临时文件名重命名为filename。
 bool CloseAndRename();


 // 调用fprintf向文件写入数据，参数与fprintf库函数相同，但不需要传入文件指针。
 void Fprintf(const char *fmt,...);


 // 从文件中读取以换行符"\n"结束的一行。
 // buffer：用于存放读取的内容。
 // readsize：本次打算读取的字节数，如果已经读取到了结束标志"\n"，函数返回。
 // bdelcrt：是否删除行结束标志，true-删除；false-不删除，缺省值是false。
 // 返回值：true-成功；false-失败，一般情况下，失败可以认为是文件已结束。
 bool Fgets(char *buffer,const int readsize,bool bdelcrt=false);

 
 // 从文件文件中读取一行。
 // buffer：用于存放读取的内容。
 // readsize：本次打算读取的字节数，如果已经读取到了结束标志，函数返回。
 // endbz：行内容结束的标志，缺省为空，表示行内容以"\n"为结束标志。
 // 返回值：true-成功；false-失败，一般情况下，失败可以认为是文件已结束。
 bool FFGETS(char *buffer,const int readsize,const char *endbz=0);

 
 // 从文件中读取数据块。
 // ptr：用于存放读取的内容。
 // size：本次打算读取的字节数。
 // 返回值：本次从文件中成功读取的字节数，如果文件未结束，返回值等于size，如果文件已结束，返回值为实际读取的字节数。
 size_t Fread(void *ptr,size_t size);

    
 // 向文件中写入数据块。
 // ptr：待写入数据的地址。
 // size：待写入数据的字节数。
 // 返回值：本次成功写入的字节数，如果磁盘空间足够，返回值等于size。
 size_t Fwrite(const void *ptr,size_t size);

 // 关闭文件指针，如果存在临时文件，就删除它。

 void Close();

};
```



### 2)、应用场景示例

在CFile类中，每个成员变量和函数都很容易理解，但不一定能准确的用于应用开发的场景中，我现在列出两个常用的场景。

**场景一：某程序每隔若干时间就会产生一批数据，并把这些数据写入文件中。**

程序的流程如下：

1）创建数据文件；

2）往文件中写入数据；

3）关闭数据文件。

如果程序这么写，得0分，因为这个流程存在一个严重的问题，那就是在第2）步往文件中写入数据需要时间，从创建文件到写入完成之前，这个文件的内容是不完整的，如果这个不完整的文件被其它程序读取了，怎么办？给文件加锁？用标志位区分？太麻烦。

假设待写入的数据文件名是/tmp/data/surfdata_20200101123000.txt，修改后的程序流程如下：

1）创建临时文件/tmp/data/surfdata_20200101123000.txt.tmp，注意，临时文件是以.tmp命名的；

2）往临时文件中写入数据；

3）关闭临时文件；

4）把临时文件/tmp/data/surfdata_20200101123000.txt.tmp重命名为正式文件/tmp/data/surfdata_20200101123000.txt。

**示例（demo39.cpp）**

```c++
/*
 \* 程序名：demo39.cpp，此程序演示开发框架中采用CFile类生成数据文件的用法。
*/
\#include "../_public.h"
 
int main()
{
 CFile File;
 
 char strLocalTime[21];  // 用于存放系统当前的时间，格式yyyymmddhh24miss。
 memset(strLocalTime,0,sizeof(strLocalTime));
 LocalTime(strLocalTime,"yyyymmddhh24miss"); // 获取系统当前时间。

 // 生成绝对路径的文件名，目录/tmp/data，文件名：前缀(surfdata_)+时间+后缀(.xml)。
 char strFileName[301];
SNPRINTF(strFileName,sizeof(strFileName),300,"/tmp/data/surfdata_%s.xml",strLocalTime);

 // 采用OpenForRename创建文件，实际创建的文件名例如/tmp/data/surfdata_20200101123000.xml.tmp。
 if (File.OpenForRename(strFileName,"w")==false)
 {
  printf("File.OpenForRename(%s) failed.\n",strFileName); return -1;
 }

 // 这里可以插入向文件写入数据的代码。
 // 写入文本数据用Fprintf方法，写入二进制数据用Fwrite方法。
 // 向文件中写入两行超女数据。

 File.Fprintf("<data>\n"\

   "<name>妲已</name><age>28</age><sc>火辣</sc><yz>漂亮</yz><memo>商要亡，关我什么事。</memo><endl/>\n"\

   "<name>西施</name><age>25</age><sc>火辣</sc><yz>漂亮</yz><memo>1、中国排名第一的美女；\n"\

   "2、男朋友是范蠡；\n"\

   "3、老公是夫差，被勾践弄死了。</memo><endl/>\n"\

   "</data>\n");


 sleep(30);  // 停止30秒，用ls /tmp/data/*.tmp可以看到生成的临时文件。

 // 关闭文件指针，并把临时文件名改为正式的文件名。
 // 注意，不能用File.Close()，因为Close方法是关闭文件指针，并删除临时文件。
 // CFile类的析构函数调用的是Close方法。
 File.CloseAndRename();

}
```

每运行一次demo39程序，就会在/tmp/data目录下生成一个数据文件，如下：

 ![image-20220716185626174](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716185626174.png)

**场景二：从目录中读取场景一生成的数据文件，解析处理后保存到数据库中，然后删除目录中的文件。**

我们利用demo39程序生成的数据来测试。

**示例（demo40.cpp）**

```c++
/*
 \* 程序名：demo40.cpp，此程序演示开发框架中采用CDir类和CFile类处理数据文件的用法。
*/
\#include "../_public.h"
 
int main()
{
 CDir Dir;
 
 // 扫描/tmp/data目录下文件名匹配"surfdata_*.xml"的文件。
 if (Dir.OpenDir("/tmp/data","surfdata_*.xml")==false)
 {
  printf("Dir.OpenDir(/tmp/data) failed.\n"); return -1;
 }
 
 CFile File;
 
 while (Dir.ReadDir()==true)
 {
  printf("处理文件%s...",Dir.m_FullFileName);
 
  if (File.Open(Dir.m_FullFileName,"r")==false)
  {
   printf("failed.File.Open(%s) failed.\n",Dir.m_FullFileName); return -1;
  }
 
  char strBuffer[301];
 
  while (true)
  {
   memset(strBuffer,0,sizeof(strBuffer));
   if (File.FFGETS(strBuffer,300,"<endl/>")==false) break; // 行内容以"<endl/>"结束。

   // printf("strBuffer=%s",strBuffer);
   // 这里可以插入解析xml字符串并把数据写入数据库的代码。
  }

  // 处理完文件中的数据后，关闭文件指针，并删除文件。
  File.CloseAndRemove();
  printf("ok\n");

 }

}
```

**运行效果**

![image-20220716185640175](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716185640175.png)

 

# 三、加载参数文件

## 1、源代码说明

本文介绍的是开发框架中加载参数文件的方法。

开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 2、参数文件的意义

在项目开发中，一个完整的系统由多个C/C++服务程序组成，这些服务程序有共同的参数，例如数据库的连接参数、日志文件存放的目录、数据文件存放的目录等。

传统的方法是把参数放在文本文件中，例如hssms.ini，格式如下：

logpath=/log/hssms       # 日志文件存放的目录。

connstr=hssms/smspwd@hssmszx # 数据库连接参数。

datapath=/data/hssms      # 数据文件存放的根目录。

serverip=192.168.1.1       # 中心服务器的ip地址。

port=5058             # 中心服务器的通信端口。

online=true            # 是否采用长连接。

现在有更好的方法是把参数放在xml文件中，例如hssms.xml，格式如下：

```xml
<?xml version="1.0" encoding="gbk" ?>

<root>

  <!-- 程序运行的日志文件名。 -->

  <logpath>/log/hssms</logpath>

  <!-- 数据库连接参数。 -->

  <connstr>hssms/smspwd@hssmszx</connstr>

  <!-- 数据文件存放的根目录。 -->

  <datapath>/data/hssms</datapath>

  <!-- 中心服务器的ip地址。 -->

  <serverip>192.168.1.1</serverip>


  <!-- 中心服务器的通信端口。 -->

  <port>5058</port>

  <!-- 是否采用长连接，true-是；false-否。 -->

  <online>true</online>

</root>
```

一般来说，一个项目是由多种语言开发完成，xml文件格式比传统的ini文件格式更方便。

## 3、CIniFile类

CIniFile类用于服务程序从参数文件中加载参数。

### 1)、类的声明

// 参数文件操作类。

// CIniFile类操作的是xml格式的参数文件，并不是传统的ini文件。

```c++
class CIniFile
{
public:
 // 存放参数文件全部的内容，由LoadFile载入到本变量中。
 string m_xmlbuffer;

 CIniFile();

 // 把参数文件的内容载入到m_xmlbuffer变量中。

 bool LoadFile(const char *filename);


 // 获取参数文件字段的内容。
 // fieldname：字段的标签名。
 // value：传入变量的地址，用于存放字段内容，支持bool、int、insigned int、long、unsigned long、double和char[]。
 // 注意，当value参数的数据类型为char []时，必须保证value数组的内存足够，否则可能发生内存溢出的问题，
 // 也可以用ilen参数限定获取字段内容的长度，ilen的缺省值为0，表示不限定获取字段内容的长度。
 // 返回值：true-获取成功；false-获取失败。
 bool GetValue(const char *fieldname,bool *value);
 bool GetValue(const char *fieldname,int *value);
 bool GetValue(const char *fieldname,unsigned int *value);
 bool GetValue(const char *fieldname,long *value);
 bool GetValue(const char *fieldname,unsigned long *value);
 bool GetValue(const char *fieldname,double *value);
 bool GetValue(const char *fieldname,char *value,const int ilen=0);

};
```



### 2)、示例程序

**示例（demo45.cpp）**

```c++
/*
 \* 程序名：demo45.cpp，此程序演示采用开发框架的CIniFile类加载参数文件。
*/
\#include "../_public.h"

// 用于存放本程序运行参数的数据结构。
struct st_args
{
 char logpath[301];
 char connstr[101];
 char datapath[301];
 char serverip[51];
 int port;
 bool online;
}stargs;

int main(int argc,char *argv[])
{
 // 如果执行程序时输入的参数不正确，给出帮助信息。
 if (argc != 2) 
 { 
  printf("\nusing:/project/public/demo/demo45 inifile\n"); 
  printf("samples:/project/public/demo/demo45 /project/public/ini/hssms.xml\n\n");
  return -1;
 }

 // 加载参数文件。
 CIniFile IniFile;
 if (IniFile.LoadFile(argv[1])==false)
 {
  printf("IniFile.LoadFile(%s) failed.\n",argv[1]); return -1;
 } 

 // 获取参数，存放在stargs结构中。
 memset(&stargs,0,sizeof(struct st_args));
 IniFile.GetValue("logpath",stargs.logpath,300);
 IniFile.GetValue("connstr",stargs.connstr,100);
 IniFile.GetValue("datapath",stargs.datapath,300);
 IniFile.GetValue("serverip",stargs.serverip,50);
 IniFile.GetValue("port",&stargs.port);
 IniFile.GetValue("online",&stargs.online);

 printf("logpath=%s\n",stargs.logpath);
 printf("connstr=%s\n",stargs.connstr);
 printf("datapath=%s\n",stargs.datapath);
 printf("serverip=%s\n",stargs.serverip);
 printf("port=%d\n",stargs.port);
 printf("online=%d\n",stargs.online);

 // 以下可以写更多的主程序的代码。

}
```

**运行效果**

![image-20220716190447287](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716190447287.png)

#  四、字符串处理框架

## 1、源代码说明

本文介绍的是开发框架的字符串操作函数和类。



开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 2、字符串复制

### 1）、STRCPY函数

安全的strcpy函数。

```c++
函数声明：

char *STRCPY(char* dest,const size_t destlen,const char* src);

参数说明：

dest：目标字符串，不需要初始化，在STRCPY函数中有初始化代码。

destlen：目标字符串dest占用内存的大小。

src：原字符串。

返回值：目标字符串dest的地址。

**示例（demo1.cpp）**

/*

 \* 程序名：demo1.cpp，此程序演示框架中STRCPY函数的使用。

*/

\#include "_public.h"

 

int main()

{

 char str[11];  // 字符串str的大小是11字节。

 

 STRCPY(str,sizeof(str),"google"); // 待复制的内容没有超过str可以存放字符串的大小。

 printf("str=%s=\n",str);  // 出输结果是str=google=

 

 STRCPY(str,sizeof(str),"www.google.com"); // 待复制的内容超过了str可以存放字符串的大小。

 printf("str=%s=\n",str);  // 出输结果是str=www.google= 

}
```



### 2）、STRNCPY函数

安全的strncpy函数。

```c++
函数声明：

char *STRNCPY(char* dest,const size_t destlen,const char* src,size_t n);

参数说明：

dest：目标字符串，不需要初始化，在STRNCPY函数中有初始化代码。

destlen：目标字符串dest占用内存的大小。

src：原字符串。

n：待复制的字节数。

返回值：目标字符串dest的地址。

**示例（demo2.cpp）**

/*

 \* 程序名：demo2.cpp，此程序演示开发框架中STRNCPY函数的使用。

*/

\#include "_public.h"

 

int main()

{

 char str[11];  // 字符串str的大小是11字节。

 

 STRNCPY(str,sizeof(str),"google",5); // 待复制的内容没有超过str可以存放字符串的大小。

 printf("str=%s=\n",str);  // 出输结果是str=googl=

 

 STRNCPY(str,sizeof(str),"www.google.com",8); // 待复制的内容没有超过str可以存放字符串的大小。

 printf("str=%s=\n",str);  // 出输结果是str=www.goog=

 

 STRNCPY(str,sizeof(str),"www.google.com",17); // 待复制的内容超过了str可以存放字符串的大小。

 printf("str=%s=\n",str);  // 出输结果是str=www.google=

}
```



## 3、字符串拼接

### 1）、STRCAT函数

安全的strcat函数。

```c++
函数声明：

char *STRCAT(char* dest,const size_t destlen,const char* src);

参数说明：

dest：目标字符串，注意，如果dest从未使过，那么需要至少一次初始化。

destlen：目标字符串dest占用内存的大小。

src：待追加字符串。

返回值：目标字符串dest的地址。

**示例（demo4.cpp）**

/*

 \* 程序名：demo4.cpp，此程序演示开发框架中STRCAT函数的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str[11];  // 字符串str的大小是11字节。

 STRCPY(str,sizeof(str),"www");

 

 STRCAT(str,sizeof(str),".fr"); // str原有的内容加上待追加的内容没有超过str可以存放的大小。

 printf("str=%s=\n",str);    // 出输结果是str=www.fr=

 

 STRCAT(str,sizeof(str),"eecplus.net"); // str原有的内容加上待追加的内容超过了str可以存放的大小。

 printf("str=%s=\n",str);        // 出输结果是str=www.freecp=

}
```



### 2）、STRNCAT函数

安全的strncat函数。

```c++
函数声明：

char *STRNCAT(char* dest,const size_t destlen,const char* src,size_t n)

参数说明：

dest：目标字符串，注意，如果dest从未使过，那么需要至少一次初始化。

destlen：目标字符串dest占用内存的大小。

src：待追加字符串。

n：待追加的字节数。

返回值：目标字符串dest的地址。

**示例（demo5.cpp）**

/*

 \* 程序名：demo5.cpp，此程序演示开发框架中STRNCAT函数的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str[11];  // 字符串str的大小是11字节。

 

 STRCPY(str,sizeof(str),"www");

 STRNCAT(str,sizeof(str),".free",10); // str原有的内容加上待追加的内容没有超过str可以存放的大小。

 printf("str=%s=\n",str);       // 出输结果是str=www.free=

 

 STRCPY(str,sizeof(str),"www");

 STRNCAT(str,sizeof(str),".freecplus.com",6); // str原有的内容加上待追加的内容没有超过str可以存放的大小。

 printf("str=%s=\n",str);           // 出输结果是str=www.freec=

 

 STRCPY(str,sizeof(str),"www");

 STRNCAT(str,sizeof(str),".freecplus.com",10); // str原有的内容加上待追加的内容超过了str可以存放的大小。

 printf("str=%s=\n",str);           // 出输结果是str=www.freecp=

}
```



## 4、格式化输出到字符串

### 1、SPRINTF函数

安全的sprintf函数，将可变参数(...)按照fmt描述的格式输出到dest字符串中。

```c++
函数声明：

int SPRINTF(char *dest,const size_t destlen,const char *fmt,...);

参数说明：

dest：输出字符串，不需要初始化，在SPRINTF函数中会对它进行初始化。

destlen：输出字符串dest占用内存的大小，如果格式化后的字符串内容的长度大于destlen-1，后面的内容将丢弃。

fmt：格式控制描述。

...：填充到格式控制描述fmt中的参数。

返回值：格式化后的内容的长度，程序员一般不关心返回值。

**示例（demo7.cpp）**

/*

 \* 程序名：demo7.cpp，此程序演示开发框架中SPRINTF函数的使用。

*/

\#include "_public.h"

 

int main()

{

 char str[21];  // 字符串str的大小是21字节。

 

 SPRINTF(str,sizeof(str),"name:%s,no:%d","messi",10);

 printf("str=%s=\n",str);  // 出输结果是str=name:messi,no:10=

 

 SPRINTF(str,sizeof(str),"name:%s,no:%d,job:%s","messi",10,"striker");

 printf("str=%s=\n",str);  // 出输结果是str=name:messi,no:10,job=

}
```



### 2）、SNPRINTF函数

安全的snprintf函数，将可变参数(...)按照fmt描述的格式输出到dest字符串中。

```c++
函数声明：

int SNPRINTF(char *dest,const size_t destlen,size_t n,const char *fmt,...);

参数说明：

dest：输出字符串，不需要初始化，在SNPRINTF函数中会对它进行初始化。

destlen：输出字符串dest占用内存的大小，如果格式化后的字符串内容的长度大于destlen-1，后面的内容将丢弃。

n：把格式化后的字符串截取n-1存放到dest中，如果n>destlen，则取destlen-1。

fmt：格式控制描述。

...：填充到格式控制描述fmt中的参数。

返回值：格式化后的内容的长度，程序员一般不关心返回值。

**示例（demo8.cpp）**

/*

 \* 程序名：demo8.cpp，此程序演示开发框架中SNPRINTF函数的使用。

*/

\#include "_public.h"

 

int main()

{

 char str[26];  // 字符串str的大小是11字节。

 

 SNPRINTF(str,sizeof(str),5,"messi");

 printf("str=%s=\n",str);  // 出输结果是str=mess=

 

 SNPRINTF(str,sizeof(str),9,"name:%s,no:%d,job:%s","messi",10,"striker");

 printf("str=%s=\n",str);  // 出输结果是str=name:mes=

 

 SNPRINTF(str,sizeof(str),30,"name:%s,no:%d,job:%s","messi",10,"striker");

 printf("str=%s=\n",str);  // 出输结果是str=name:messi,no:10,job:stri=

}
```



## 5、删除字符串左、右和两边字符

删除字符串左边、右边和两边指定的字符。

```c++
函数声明：

void DeleteLChar(char *str,const char chr); // 删除字符串左边指定的字符。

void DeleteRChar(char *str,const char chr); // 删除字符串右边指定的字符。

void DeleteLRChar(char *str,const char chr); // 删除字符串左右两边指定的字符。

参数说明：

str：待处理的字符串。

chr：需要删除的字符。

**示例（demo10.cpp）**

/*

 \* 程序名：demo10.cpp，此程序演示开发框架中删除字符串左、右、两边指定字符的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str[11];  // 字符串str的大小是11字节。

 

 STRCPY(str,sizeof(str)," 西施 ");

 DeleteLChar(str,' '); // 删除str左边的空格

 printf("str=%s=\n",str);  // 出输结果是str=西施 =

 

 STRCPY(str,sizeof(str)," 西施 ");

 DeleteRChar(str,' '); // 删除str右边的空格

 printf("str=%s=\n",str);  // 出输结果是str= 西施=

 

 STRCPY(str,sizeof(str)," 西施 ");

 DeleteLRChar(str,' '); // 删除str两边的空格

 printf("str=%s=\n",str);  // 出输结果是str=西施=

}
```

注意，如果要删除字符串中间的字符，可以用开发框架中的UpdateStr函数，后面的章节中会介绍它。

## 6、字符串大小写转换

把字符串中的小写字母转换成大写，忽略不是字母的字符。

```c++
函数声明：

void ToUpper(char *str);  // 把字符串中的小写字母转换成大写,参数是C语言风格的char []。

void ToUpper(string &str); // 把字符串中的小写字母转换成大写,参数是C++语言风格的string。

void ToLower(char *str);  // 把字符串中的大写字母转换成小写,参数是C语言风格的char []。

void ToLower(string &str); // 把字符串中的大写字母转换成小写,参数是C++语言风格的string。

参数说明：

str：待转换的字符串，函数重载了char[]和string两种数据类型。

**示例（demo12.cpp）**

/*

 \* 程序名：demo12.cpp，此程序演示开发框架中字符串大小写转换函数的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str1[31];  // C语言风格的字符串。

 

 STRCPY(str1,sizeof(str1),"12abz45ABz8西施。");

 ToUpper(str1);   // 把str1中的小写字母转换为大写。

 printf("str1=%s=\n",str1);  // 出输结果是str1=12ABZ45ABZ8西施。=

 

 STRCPY(str1,sizeof(str1),"12abz45ABz8西施。");

 ToLower(str1);   // 把str1中的大写字母转换为小写。

 printf("str1=%s=\n",str1);  // 出输结果是str1=12abz45abz8西施。=

 

 string str2;  // C++语言风格的字符串。

 

 str2="12abz45ABz8西施。";

 ToUpper(str2);   // 把str2中的小写字母转换为大写。

 printf("str2=%s=\n",str2.c_str());  // 出输结果是str2=12ABZ45ABZ8西施。=

 

 str2="12abz45ABz8西施。";

 ToLower(str2);   // 把str2中的大写字母转换为小写。

 printf("str2=%s=\n",str2.c_str());  // 出输结果是str1=12abz45abz8西施。=

}
```



## 7、字符串替换

把字符串中的内容替换成其它的内容，在字符串str中，如果存在字符串str1，就替换为字符串str2。

```c++
函数声明：

void UpdateStr(char *str,const char *str1,const char *str2,const bool bLoop=true);

参数说明：

str：待处理的字符串。

str1：旧的内容。 

str2：新的内容。 

bloop：是否循环执行替换。

注意：

1）如果str2比str1要长，替换后str会变长，所以必须保证str有足够的长度，否则内存会溢出。

2）如果str2中包函了str1的内容，且bloop为true，存在逻辑错误，将不执行任何替换。

**示例（demo21.cpp）**

/*

 \* 程序名：demo21.cpp，此程序演示开发框架中字符串替换UpdateStr函数的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str[301];

 

 STRCPY(str,sizeof(str),"name:messi,no:10,job:striker.");

 UpdateStr(str,":","=");   // 把冒号替换成等号。

 printf("str=%s=\n",str);  // 出输结果是str1=name=messi,no=10,job=striker.=

 

 STRCPY(str,sizeof(str),"name:messi,no:10,job:striker.");

 UpdateStr(str,"name:","");  // 把"name:"替换成""，相当于删除内容"name:"。

 printf("str=%s=\n",str);   // 出输结果是str1=messi,no:10,job:striker.=

 

 STRCPY(str,sizeof(str),"messi----10----striker");

 UpdateStr(str,"--","-",false);  // 把两个"--"替换成一个"-"，bLoop参数为false。

 printf("str=%s=\n",str);     // 出输结果是str1=messi--10--striker=

 

 STRCPY(str,sizeof(str),"messi----10----striker");

 UpdateStr(str,"--","-",true);  // 把两个"--"替换成一个"-"，bLoop参数为true。

 printf("str=%s=\n",str);     // 出输结果是str1=messi-10-striker=

 

 STRCPY(str,sizeof(str),"messi-10-striker");

 UpdateStr(str,"-","--",false);  // 把一个"-"替换成两个"--"，bLoop参数为false。

 printf("str=%s=\n",str);     // 出输结果是str1=messi--10--striker=

 

 STRCPY(str,sizeof(str),"messi-10-striker");

 

 // 以下代码把"-"替换成"--"，bloop参数为true，存在逻辑错误，UpdateStr将不执行替换。

 UpdateStr(str,"-","--",true);  // 把一个"-"替换成两个"--"，bloop参数为true。

 printf("str=%s=\n"
```

,str);     // 出输结果是str1=messi-10-striker=

}

## 8、从字符串中提取数字

从一个字符串中提取出数字、符号和小数点，存放到另一个字符串中。

```c++
函数声明：

void PickNumber(const char *src,char *dest,const bool bsigned,const bool bdot);

参数说明：

src：源字符串。

dest：目标字符串。

bsigned：是否包括符号（+和-），true-包括；false-不包括。

bdot：是否包括小数点的圆点符号，true-包括；false-不包括。

**示例（demo16.cpp）**

/*

 \* 程序名：demo16.cpp，此程序演示开发框架中PickNumber函数的使用。

*/

\#include "../_public.h"

 

int main()

{

 char str[26];  // 字符串str的大小是11字节。

 

 STRCPY(str,sizeof(str),"iab+12.3xy");

 PickNumber(str,str,false,false);

 printf("str=%s=\n",str);  // 出输结果是str=123=

 

 STRCPY(str,sizeof(str),"iab+12.3xy");

 PickNumber(str,str,true,false);

 printf("str=%s=\n",str);  // 出输结果是str=+123=

 

 STRCPY(str,sizeof(str),"iab+12.3xy");

 PickNumber(str,str,true,true);

 printf("str=%s=\n",str);  // 出输结果是str=+12.3=

}
```



## 9、正则表达式

正则表达式，判断一个字符串是否匹配另一个字符串。

```c++
函数声明：
bool MatchStr(const string str,const string rules);

参数说明：
str：需要判断的字符串，精确表示的字符串，如文件名"_public.h.cpp"。
rules：匹配规则表达式，用星号"*"表示任意字符串，多个字符串之间用半角的逗号分隔，如"*.h,*.cpp"。

注意，str参数不支持"?"，rules只支持"*"，函数在判断str是否匹配rules的时候，会忽略字母的大小写。
**示例（demo18.cpp）**
/*
 \* 程序名：demo18.cpp，此程序演示开发框架正则表达示MatchStr函数的使用。
*/
\#include "../_public.h"

int main()
{
 char filename[301];
 STRCPY(filename,sizeof(filename),"_public.h");

 // 以下代码将输出yes。
 if (MatchStr(filename,"*.h,*.cpp")==true) printf("yes\n");
 else printf("no\n");
 
 // 以下代码将输出yes。
 if (MatchStr(filename,"*.H")==true) printf("yes\n");
 else printf("no\n");

 // 以下代码将输出no。
 if (MatchStr(filename,"*.cpp")==true) printf("yes\n");
 else printf("no\n");
}
```



## 10、字符串的拆分

### 1.CCmdstr

```c++
//CCmdStr类用于拆分用分隔符分隔字段内容的字符串。

//字符串的格式为：字段内容1+分隔符+字段内容2+分隔符+字段内容3+分隔符+....+字段内容n。

//例如："messi,10,striker,30,1.72,68.5,Barcelona"，这是足球运动员梅西的资料，包括姓名、球衣号码、场上位置、年龄、身高、体重和效力的俱乐部，字段之间用半角的逗号分隔开。
```

CCmdStr类的声明：

```c++
class CCmdStr
{
public:
 vector<string> m_vCmdStr; // 存放拆分后的字段内容。
 CCmdStr();
 
 // 把字符串拆分到m_vCmdStr容器中。
 // buffer：待拆分的字符串。
 // sepstr：buffer字符串中字段内容的分隔符，注意，分隔符是字符串，如","、" "、"|"、"~!~"。
 // bdelspace：是否删除拆分后的字段内容前后的空格，true-删除；false-不删除，缺省删除。
 void SplitToCmd(const string buffer,const char *sepstr,const bool bdelspace=true);

 // 获取拆分后字段的个数，即m_vCmdStr容器的大小。
 int CmdCount();
 
 // 从m_vCmdStr容器获取字段内容。
 // inum：字段的顺序号，类似数组，从0开始。
 // value：传入变量的地址，用于存放字段内容。
 // 返回值：true-获取成功；false-获取失败。
 bool GetValue(const int inum,char  *value,const int ilen=0); // 字符串，ilen缺省值为0。
 bool GetValue(const int inum,int  *value); // int整数。
 bool GetValue(const int inum,unsigned int *value); // unsigned int整数。
 bool GetValue(const int inum,long  *value); // long整数。
 bool GetValue(const int inum,unsigned long *value); // unsigned long整数。
 bool GetValue(const int inum,double *value); // 双精度double。
 bool GetValue(const int inum,bool  *value); // bool型。

 ~CCmdStr();

};
```

**示例（demo20.cpp）**

```c++
/*

 \* 程序名：demo20.cpp，此程序演示开发框架拆分字符串的类CCmdStr的使用。

*/

\#include "../_public.h"

 

// 用于存放足球运动员资料的结构体。

struct st_player

{

 char name[51];  // 姓名

 char no[6];    // 球衣号码

 bool striker;   // 场上位置是否是前锋，true-是；false-不是。

 int age;     // 年龄

 double weight;  // 体重，kg。

 long sal;     // 年薪，欧元。

 char club[51];  // 效力的俱乐部

}stplayer;

 

int main()

{

 memset(&stplayer,0,sizeof(struct st_player));

 

 char buffer[301]; 

 STRCPY(buffer,sizeof(buffer),"messi,10,true,30,68.5,2100000,Barcelona");

 

 CCmdStr CmdStr;

 CmdStr.SplitToCmd(buffer,",");    // 拆分buffer

 CmdStr.GetValue(0, stplayer.name,50); // 获取姓名

 CmdStr.GetValue(1, stplayer.no,5);  // 获取球衣号码

 CmdStr.GetValue(2,&stplayer.striker); // 场上位置

 CmdStr.GetValue(3,&stplayer.age);   // 获取年龄

 CmdStr.GetValue(4,&stplayer.weight); // 获取体重

 CmdStr.GetValue(5,&stplayer.sal);   // 获取年薪，欧元。

 CmdStr.GetValue(6, stplayer.club,50); // 获取效力的俱乐部

 

 printf("name=%s,no=%s,striker=%d,age=%d,weight=%.1f,sal=%ld,club=%s\n",\

​     stplayer.name,stplayer.no,stplayer.striker,stplayer.age,\

​     stplayer.weight,stplayer.sal,stplayer.club);

 // 输出结果:name=messi,no=10,striker=1,age=30,weight=68.5,sal=21000000,club=Barcelona

}
```

# 五、时间操作框架

## 1、源代码说明

本文介绍的是开发框架的时间操作函数。

开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 2、计算机时间的表示方法

 UNIX操作系统根据计算机产生的年代和应用采用1970年1月1日作为UNIX的纪元时间，1970年1月1日0点作为计算机表示时间的是中间点，将从1970年1月1日开始经过的秒数用一个整数存放，这种高效简洁的时间表示方法被称为“Unix时间纪元”，向左和向右偏移都可以得到更早或者更后的时间。

在实际开发中，对日期和时间的操作场景非常多，例如程序启动和退出的时间，程序执行任务的时间，数据生成的时间，数据处理的各环节的时间等等。

在Linux系统中，自定义了time_t类型，如下：

typedef long time_t;  // 时间值time_t为长整型long的别名。

## 3、获取操作系统的时间

取操作系统的时间，并把整数表示的时间转换为字符串表示的格式。

函数声明：

void LocalTime(char *out_stime,const char *in_fmt=0,const int in_interval=0);

参数说明：

**stime：**用于存放获取到的时间字符串。

**timetvl：**时间的偏移量，单位：秒，0是缺省值，表示当前时间，30表示当前时间30秒之后的时间点，-30表示当前时间30秒之前的时间点。

**fmt：**输出时间的格式，fmt每部分的含义："yyyy"-年份；"mm"-月份；"dd"-日期；"hh24"-小时；"mi"-分钟；"ss"-秒，缺省是"yyyy-mm-dd hh24:mi:ss"，目前支持以下格式：

 "yyyy-mm-dd hh24:mi:ss"

 "yyyymmddhh24miss"

 "yyyy-mm-dd"

 "yyyymmdd"

 "hh24:mi:ss"

 "hh24miss"

 "hh24:mi" 

 "hh24mi"

 "hh24"

 "mi"

注意：

1）小时的表示方法是hh24，不是hh，这么做的目的是为了保持与数据库的时间表示方法一致；

2）以上列出了常用的时间格式，如果不能满足您应用开发的需求，请修改源代码timetostr函数增加更多的格式支持；

3）调用函数的时候，如果fmt与上述格式都匹配，stime的内容将为空。

**示例（demo24.cpp）**

```c++
/*
 \* 程序名：demo24.cpp，此程序演示开发框架中LocalTime时间函数的使用。
*/
\#include "../_public.h"

int main()
{
 char strtime[20];
 memset(strtime,0,sizeof(strtime));
    
 LocalTime(strtime,"yyyy-mm-dd hh24:mi:ss",-30); // 获取30秒前的时间。
 printf("strtime1=%s\n",strtime);

 LocalTime(strtime,"yyyy-mm-dd hh24:mi:ss");   // 获取当前时间。
 printf("strtime2=%s\n",strtime);
 LocalTime(strtime,"yyyy-mm-dd hh24:mi:ss",30);  // 获取30秒后的时间。
 printf("strtime3=%s\n",strtime);

 printf("=%d\n",time(0));
}
```



## 4、时间转换函数

### 1）、把整数表示的时间转换为字符串表示的时间

函数声明：

void timetostr(const time_t ltime,char *stime,const char *fmt=0);

参数说明：

**ltime**：整数表示的时间。

**stime**：字符串表示的时间。

**fmt：**输出字符串时间stime的格式，与LocalTime函数的fmt参数相同，如果fmt的格式不正确，stime将为空。

### 2）、把字符串表示的时间转换为整数表示的时间

函数声明：

time_t strtotime(const char *stime);

参数说明：

**stime：**字符串表示的时间，格式不限，但一定要包括yyyymmddhh24miss，一个都不能少。

返回值：整数表示的时间，如果stime的格式不正确，返回-1。

**示例（demo26.cpp）**

```c++
/*
 \* 程序名：demo26.cpp，此程序演示开发框架中整数表示的时间和字符串表示的时间之间的转换。
*/
\#include "../_public.h"


int main()
{
 time_t ltime;
 char strtime[20];

 memset(strtime,0,sizeof(strtime));
 strcpy(strtime,"2020-01-01 12:35:22");

 ltime=strtotime(strtime);  // 转换为整数的时间
 printf("ltime=%ld\n",ltime); // 输出ltime=1577853322

 memset(strtime,0,sizeof(strtime));
 timetostr(ltime,strtime,"yyyy-mm-dd hh24:mi:ss"); // 转换为字符串的时间
 printf("strtime=%s\n",strtime);   // 输出strtime=2020-01-01 12:35:22

}
```



## 5、时间的运算

把字符串表示的时间加上一个偏移的秒数后得到一个新的字符串表示的时间。

函数声明：

bool AddTime(const char *in_stime,char *out_stime,const int timetvl,const char *fmt=0);

参数说明：

**in_stime：**输入的字符串格式的时间。

**out_stime：**输出的字符串格式的时间。

**timetvl：**需要偏移的秒数，正数往后偏移，负数往前偏移。

**fmt：**输出字符串时间out_stime的格式，与LocalTime函数的fmt参数相同。

注意：in_stime和out_stime参数可以是同一个变量的地址，如果调用失败，out_stime的内容会清空。

返回值：true-成功，false-失败，如果返回失败，可以认为是in_stime的格式不正确。

**示例（demo28.cpp）**

```c++
/*
 \* 程序名：demo28.cpp，此程序演示开发框架中采用AddTime进行时间的运算。
*/
\#include "../_public.h"

int main()
{
 time_t ltime;
 char strtime[20];

 memset(strtime,0,sizeof(strtime));
 strcpy(strtime,"2020-01-01 12:35:22");

 AddTime(strtime,strtime,0-1*24*60*60); // 减一天。
 printf("strtime=%s\n",strtime);   // 输出strtime=2019-12-31 12:35:22


 AddTime(strtime,strtime,2*24*60*60); // 加两天。
 printf("strtime=%s\n",strtime);   // 输出strtime=2020-01-02 12:35:22
}
```



## 6、计时器

CTimer类是一个精确到微秒的计时器。

类声明：

// 这是一个精确到微秒的计时器。

```c++
class CTimer
{
private:
 struct timeval m_start;  // 开始计时的时间。
 struct timeval m_end;   // 计时完成的时间。

 // 开始计时。
 void Start();
public:

 CTimer(); // 构造函数中会调用Start方法。


 // 计算已逝去的时间，单位：秒，小数点后面是微秒。
 double Elapsed();
};
```

CTimer创建对象后立即开始计时，每次调用Elapsed方法获取已逝去的时间（单位：秒，小数点后面是微秒），并重新开始计时。

**示例（demo29.cpp）**

```c++
/*
 \* 程序名：demo29.cpp，此程序演示开发框架中的CTimer类（计时器）的用法。
*/
\#include "../_public.h"

int main()
{
 CTimer Timer;


 printf("elapsed=%lf\n",Timer.Elapsed());
 sleep(1);
 printf("elapsed=%lf\n",Timer.Elapsed());
 sleep(1);
 printf("elapsed=%lf\n",Timer.Elapsed());
 usleep(1000);
 printf("elapsed=%lf\n",Timer.Elapsed());
 usleep(100);
 printf("elapsed=%lf\n",Timer.Elapsed());
 sleep(10);
 printf("elapsed=%lf\n",Timer.Elapsed());
}
```

**运行效果**

![image-20220716192542438](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220716192542438.png)

从demo29运行的效果上看，好像计时有误差，同样是睡1秒，实际耗时却是1.000126或1.000171，这是因为程序本身执行需要时间，虽然时间很短，那也是需要时间。

# 六、xml解析

## 1、源代码说明

本文介绍的是采用开发框架的解析xml格式字符串函数。

开发框架函数和类的声明文件是/project/public/_public.h。

开发框架函数和类的定义文件是/project/public/_public.h.cpp。

示例程序位于/project/public/demo目录中。

编译规则文件是/project/public/demo/makefile。

## 2、xml格式字符串介绍

xml格式字符串是应用开发中被广泛采用的一种数据格式，简单易懂，容错性和可扩展性非常好，是数据处理、数据通讯和数据交换等应用场景的首选数据格式。

完整的xml格式比较复杂，但是，在实际开发中，对我们C/C++程序员来说，绝大部分场景下用到的xml数据格式比较简单，例如表示文件列表信息的xml数据集或文件内容如下：

<data>

<filename>_public.h</filename><mtime>2020-01-01 12:20:35</mtime><size>1834</size><endl/>

<filename>_public.cpp</filename><mtime>2020-01-01 10:10:15</mtime><size>5094</size><endl/>

</data>

数据集说明：

<data>：数据集的开始。

</data>：数据集的结束。

<endl/>：每行数据的结束。

filename标签：文件名。

mtime标签：文件最后一次被修改的时间。

size标签：文件的大小。

## 3、xml格式字符串的解析

在开发框架中，提供了解析以下xml格式字符串的一系函数。

函数声明：

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,bool  *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,int  *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,unsigned int *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,long  *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,unsigned long *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,double *value);

bool GetXMLBuffer(const char *xmlbuffer,const char *fieldname,char *value,const int ilen=0);

参数说明：

xmlbuffer：待解析的xml格式字符串的内容。

fieldname：字段的标签名。

value：传入变量的地址，用于存放字段内容，支持bool、int、insigned int、long、unsigned long、double和char[]。

注意，当value参数的数据类型为char []时，必须保证value数组的内存足够，否则可能发生内存溢出的问题，也可以用ilen参数限定获取字段内容的长度，ilen的缺省值为0，表示不限定获取字段内容的长度。

返回值：true-获取成功；false-获取失败。

**示例（demo22.cpp）**

```c++
/*
 \* 程序名：demo22.cpp，此程序演示调用开发框架的GetXMLBuffer函数解析xml字符串。
*/
\#include "../_public.h"
 
// 用于存放足球运动员资料的结构体。
struct st_player
{
 char name[51];  // 姓名
 char no[6];    // 球衣号码
 bool striker;   // 场上位置是否是前锋，true-是；false-不是。
 int age;     // 年龄
 double weight;  // 体重，kg。
 long sal;     // 年薪，欧元。
 char club[51];  // 效力的俱乐部
}stplayer;

int main()
{
 memset(&stplayer,0,sizeof(struct st_player));

 char buffer[301]; 
  STRCPY(buffer,sizeof(buffer),"<name>messi</name><no>10</no><striker>true</striker><age>30</age><weight>68.5</weight><sal>21000000</sal><club>Barcelona</club>");

 GetXMLBuffer(buffer,"name",stplayer.name,50);
 GetXMLBuffer(buffer,"no",stplayer.no,5);
 GetXMLBuffer(buffer,"striker",&stplayer.striker);
 GetXMLBuffer(buffer,"age",&stplayer.age);
 GetXMLBuffer(buffer,"weight",&stplayer.weight);
 GetXMLBuffer(buffer,"sal",&stplayer.sal);
 GetXMLBuffer(buffer,"club",stplayer.club,50);

 
 printf("name=%s,no=%s,striker=%d,age=%d,weight=%.1f,sal=%ld,club=%s\n",\
​     stplayer.name,stplayer.no,stplayer.striker,stplayer.age,\
​     stplayer.weight,stplayer.sal,stplayer.club);
 // 输出结果:name=messi,no=10,striker=1,age=30,weight=68.5,sal=21000000,club=Barcelona

}
```



## 4、应用经验

对C/C++程序员来说，采用简单的xml字符串表达数据可以提高开发效率，我不建议采用复杂的xml格式，会让程序代码很烦锁。

如果在实际开发中需要解析更复杂的xml，可以寻找网上的开源库，例如libxml++。



# 七、linux经验

## 1、多进程

![image-20220717142934675](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717142934675.png)



**文件缓冲区没有刷新可能导致子进程获取父进程文件缓冲区的数据，从而重复父进程的数据。**

![image-20220717182547696](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717182547696.png)

改进：加入fflush(fp)

fork之后，父进程和子进程的执行先后不确定。两个进程相互独立，不会影响对方。

**僵尸进程**

![image-20220717184401755](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717184401755.png)

解决僵尸进程：处理SIG_HLD信号；

![image-20220717184725883](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717184725883.png)

## 2、调度

excel();

excecv();

这两个函数用于将指定的程序覆盖当前运行的程序。实现调度。要注意添加exit(0).防止调度失败造成死循环。

![image-20220717194941009](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717194941009.png)

## 3、共享内存

Linux中提供了一组函数用于操作共享内存，程序中需要包含以下头文件：

```c++
#include <sys/ipc.h>
#include <sys/shm.h>
```



### 1、shmget函数

shmget函数用来获取或创建共享内存，它的声明为：

```c++
int shmget(key_t key, size_t size, int shmflg);
```

参数key是共享内存的键值，是一个整数，typedef unsigned int key_t，是共享内存在系统中的编号，不同共享内存的编号不能相同，这一点由程序员保证。key用十六进制表示比较好。

参数size是待创建的共享内存的大小，以字节为单位。

参数shmflg是共享内存的访问权限，与文件的权限一样，0666|IPC_CREAT表示全部用户对它可读写，如果共享内存不存在，就创建一个共享内存。

### 2、shmat函数

把共享内存连接到当前进程的地址空间。它的声明如下：

```c++
void *shmat(int shm_id, const void *shm_addr, int shmflg);
```

参数shm_id是由shmget函数返回的共享内存标识。

参数shm_addr指定共享内存连接到当前进程中的地址位置，通常为空，表示让系统来选择共享内存的地址。

参数shm_flg是一组标志位，通常为0。

调用成功时返回一个指向共享内存第一个字节的指针，如果调用失败返回-1.

### 3、shmdt函数

该函数用于将共享内存从当前进程中分离，相当于shmat函数的反操作。它的声明如下：

```c++
int shmdt(const void *shmaddr);
```

参数shmaddr是shmat函数返回的地址。

调用成功时返回0，失败时返回-1.

### 4、shmctl函数

删除共享内存，它的声明如下：

```c++
int shmctl(int shm_id, int command, struct shmid_ds *buf);
```

参数shm_id是shmget函数返回的共享内存标识符。

参数command填IPC_RMID。

参数buf填0。

解释一下，shmctl是控制共享内存的函数，其功能不只是删除共享内容，但其它的功能没什么用，所以不介绍了。

**注意，用root创建的共享内存，不管创建的权限是什么，普通用户无法删除。**

### 5、实例

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>

struct st_pid
{
  int  pid;       // 进程编号。
  char name[51];  // 进程名称。
};

int main(int argc,char *argv[])
{
  // 共享内存的标志。
  int shmid;
  
  // 获取或者创建共享内存，键值为0x5005。
  if ( (shmid=shmget(0x5005, sizeof(struct st_pid), 0640|IPC_CREAT))==-1)
  { printf("shmget(0x5005) failed\n"); return -1; }

  // 用于指向共享内存的结构体变量。
  struct st_pid *stpid=0;

  // 把共享内存连接到当前进程的地址空间。
  if ( (stpid=(struct st_pid *)shmat(shmid,0,0))==(void *)-1)
  { printf("shmat failed\n"); return -1; }
  //写入之前读一次
  printf("pid=%d,name=%s\n",stpid->pid,stpid->name);

  stpid->pid=getpid();
  strcpy(stpid->name,argv[1]);
  //写入之后读一次
  printf("pid=%d,name=%s\n",stpid->pid,stpid->name);

  // 把共享内存从当前进程中分离。
  shmdt(stpid);

  // 删除共享内存。
  // if (shmctl(shmid,IPC_RMID,0)==-1)
  // { printf("shmctl failed\n"); return -1; }
  return 0;
}

```

![image-20220717200513677](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717200513677.png)

用ipcs -m可以查看系统的共享内存，内容有键值（key），共享内存编号（shmid），创建者（owner），权限（perms），大小（bytes）。

​        ![image-20220717200832638](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717200832638.png)                       

用ipcrm -m 共享内存编号，可以手工删除共享内存，如下：

![image-20220717200839510](C:\Users\hcw\Desktop\项目整理笔记\image\image-20220717200839510.png)

 
