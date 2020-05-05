[TOC]

# os

os模块是与操作系统交互的一个接口

## 3.4版本之前

### 路径操作

os.path模块:

import os

os.path.abspath():获取绝对路径

os.path.getsize():获取文件大小

os.path.getatime():获取访问时间

os.path.getmtime():获取修改时间

os.path.split():分割文件和目录路径,返回元组

os.path.normpath():规范path字符串形式

os.path.split():分割文件名与目录(事实上，如果你完全使用目录，它也会将最后一个目录作为文件名而分离，同时它不会判断文件或目录是否存在)

os.path.splitext():分离文件名和扩展名

os.path.join():连接目录与文件名或目录

os.path.basename():返回path最后的文件名,如果path以/或\结尾，则返回空值

os.path.dirname():返回文件路径

os.path.isfile()和os.path.isdir()分别检验给出的路径是一个目录还是文件

os.path.existe():检验给出的路径是否真的存在

os.path.isdir():判断是不是目录，不是目录就返回false

os.path.isfile():判断这个文件是否存在，不存在返回false

os.path.exists():判断是否存在文件或目录

os.path.isabs():判断是否为绝对路径

os.path.normcase('path'):在Linux和Mac平台上，该函数会原样返回path，在windows平台上会将路径中所有字符转换为小写，并将所有斜杠转换为反斜杠

 

```
os.path.normcase('c:/windows\\system32\\')  
# 运行结果:'c:\\windows\\system32\\'
```

os.path.normpath('path'):规范化路径，如'..'和'/'

 

```
os.path.normpath('c://windows\\System32\\../Temp/')  
# 运行结果:'c:\\windows\\Temp'

a='/Users/my-pc/test1/\\\a1/\\\\aa.py/../..'
print(os.path.normpath(a))
# 运行结果:/Users/my-pc/test1
```

路径处理

 

```
# 方式一：推荐使用
import os,sys
possible_topdir = os.path.normpath(os.path.join(
    os.path.abspath(__file__),
    os.pardir, #上一级
    os.pardir,
    os.pardir
))
sys.path.insert(0,possible_topdir)

#方式二：不推荐使用
os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```

### 目录操作

os.mkdir("file"): 创建目录

shutil.copyfile("oldfile","newfile"): 复制文件,oldfile和newfile都只能是文件

shutil.copy("oldfile","newfile"): 复制文件,oldfile只能是文件夹，newfile可以是文件，也可以是目标目录

shutil.copytree("olddir","newdir"): 复制文件夹.olddir和newdir都只能是目录，且newdir必须不存在

os.rename("oldname","newname"): 重命名文件（目录）.文件或目录都是使用这条命令

shutil.move("oldpos","newpos"): 移动文件（目录）

os.rmdir("dir"): 只能删除空目录

shutil.rmtree("dir"): 空目录、有内容的目录都可以删

## 其他方法

os.sep:显示当前操作系统平台特定的路径分隔符,win为'\\',linux为'/'

os.linesep:给出当前平台的行终止符。例如，Windows使用'\r\n'，Linux使用'\n'，而Mac使用'\r'

os.name:指示你正在使用的工作平台。比如对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'。

os.pathsep:输出用于分割文件路径的字符串,win下为';',linux下为':'

os.getcwd:得到当前工作目录，即当前python脚本工作的目录路径。

os.chdir('dirname'):改变当前工作目录到dirname,相当于shell下的cd

os.curdir:返回当前目录（'.'）

os.pardir:获取当前目录的父目录字符串名('..')

os.mkdir('dirname'):创建单级目录,相当于shell下mkdir dirname

os.makedirs('dirname1/dirname2'):递归创建多层目录

os.removedirs（'dirname'）:删除多级目录,若目录为空，则删除，并递归到上级目录，若也为空，也删除

os.listdir('dirname'):列出指定目录下的所有文件和子目录，包括隐藏文件，生成列表形式

os.remove('filename'):删除一个文件

os.rename('oldname','newname'):重命名文件或目录

os.getenv()和os.putenv:分别用来读取和设置环境变量

os.chmod('filename'):修改文件权限和时间戳

os.rmdir('dirname'):删除目录

os.system('bash command'):运行shell命令,直接显示

os.exit():终止当前进程

os.stat(path, *, dir_fd=None, follow_symlinks=True): 获得文件或目录属性信息,本质上调用linux系统的stat

path：路径的string或bytes或fd文件描述符

follow_symlinks True：返回文件本身信息，False且如果是软连接则显示软链接本身

os.getcwd() 方法用于返回当前工作目录

os.path.getatime(file) 输出文件访问时间

os.path.getctime(file) 输出文件的创建时间

os.path.getmtime(file) 输出文件最近修改时间

# sys

常用模块:

sys.argv: 在外部向程序内部传递参数

例如：

创建一个py文件:argv.py,内容如下:

 

```
import sys
print(sys.argv) # 显示文件所在绝对路径,为列表的第0个元素
print(sys.argv[1]) # 以空格为分隔符,将后面的字符串生成一个列表,显示列表第一个元素
print(sys.argv[2]) # 显示第二个元素
```

在命令行终端执行:

 

```
> python /test/tmp.py --host 127.0.0.1 --port 8000
"""
运行结果:
['/test/tmp.py', '--host', '192.168.1.1', '--port', '8080']
--host
127.0.0.1
"""
```

sys.exit(n): 退出程序,正常退出时exit(0)

说明: 执行到主程序末尾,解释器自动退出,但是如果需要中途退出程序,可以调用sys.exit函数,带有一个可选的整数参数返回给调用它的程序,表示你可以在主程序中捕获对sys.exit的调用.(0是正常退出，其他为异常)

sys.modules: 一个全局字典,该字典是python启动后就加载在内存中.每当程序员导入新的模块,sys.modules将自动记录该模块,当第二次再导入该模块时,python会直接到字典中查找,从而加快了程序运行的速度,它拥有字典所拥有的一切方法.

sys.version: 获取python解释程序的版本信息

sys.getdefaultencoding(): 获取系统当前字符编码,一般默认为ASCII

sys.setdefaultencoding(): 设置系统默认编码,如果在解释器中执行print(dir(sys)),找不到此方法,可以先执行reload(sys)

sys.getfilesystemencoding(): 获取文件系统使用的编码格式,win返回mbcs,mac返回utf-8

sys.maxint: 最大的Int值

sys.path: 返回模块的搜索路径,初始化时使用PYTHONPATH环境变量的值,可将自定义的模块放在此路径下,通过import可导入

sys.platform: 返回操作系统平台名称

sys.stdin: 标准输入

sys.stdout: 标准输出

sys.stderr: 标准错误输出

说明: stdin,stdout,stderr变量包含与标准I/O 流对应的流对象,如果需要更好地控制输出,而print不能满足你的要求, 它们就是你所需要的.你也可以替换它们,这时候你就可以重定向输出和输入到其它设备(device),或者以非标准的方式处理它们.

应用场景:打印#号进度条

 

```
import sys, time

for i in range(100):
    sys.stdout.write('%s\r', %('#'*i))
    sys.stdout.flush()
    time.sleep(0.1)
```