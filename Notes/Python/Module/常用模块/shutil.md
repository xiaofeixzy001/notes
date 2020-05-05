[TOC]

# shutil

高级的文件,文件夹,压缩包处理模块

## copyfileobj

copyfileobj(fsrc, fdst, [length])

文件对象的复制，将文件内容拷贝到另一个文件中。

fsrc和fdst是open打开的文件对象，复制内容。fdst要求可写，length指定了表示buffer的大小

 

```
import shutil

src = 'test1.txt'
dst = 'copy_test1'

shutil.copyfileobj(open(src), open(dst, 'w'))  # src必须事先存在

with open(src, 'r+') as f1:
    f1.write('123\n456')
    f1.flushu()
    f1.seek(0)
    with open(dst, 'w+') as f2:
        shutil.copyfileobj(f1, f2)
```

## copyfile

copyfile(src, dst, *, follow_symlinks=True)

复制文件内容，不含元数据,dst文件无需存在,会自动创建,存在则覆盖。

src和dst为文件的路径(字符串)，本质上调用的就是copyfileobj，所以是不带元数据二进制内容的复制。

## copymode

copymode(src, dst, *, follow_symlinks=True): 仅复制权限，dst文件必须事先存在。

 

```
import shutil

src = 'test1.txt'
dst = 'test2.txt'

shutil.copymode(src, dst)
os.stat(src)
os.stat(dst)
'''
os.stat_result(st_mode=33206, st_ino=5629499534234861, st_dev=3027550873, st_nlink=1, st_uid=0, st_gid=0, st_size=14, st_atime=1552283439, st_mtime=1551952511, st_ctime=1552283439)
os.stat_result(st_mode=33206, st_ino=281474976732403, st_dev=3027550873, st_nlink=1, st_uid=0, st_gid=0, st_size=6, st_atime=1552292421, st_mtime=1552292421, st_ctime=1552292410)

'''
```

## copystat

copystat(src, dst, *, follow_symlinks=True)

复制元数据信息(linux中stat包含的元数据信息)。dst文件需事先存在

## copy

copy(src, dst, *, follow_symlinks=True)

复制文件内容、权限和部分元数据，不包括创建时间和修改时间，本质上调用的是copyfile和copymode。

## copy2

copy2(src, dst, *, follow_symlinks=True)

copy2比copy多了复制所有元数据功能，但需要平台支持，本质上调用的是copyfile和copystat。

## ignore_patterns

ignore_patterns('*patterns')

模式匹配

## copytree

copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False)

递归复制目录，默认使用copy2，也就是带更多的元数据复制。

src和dst必须是目录，src必须存在，dst必须不存在;

ignore=func：提供一个callable(src, names)，提供一个回调函数，它会被调用。src是源目录，names是os.listdir(src)的结果，就是列出src中的文件名，返回值是要被过滤的文件名的set类型数据。

symlinks=False|True: 为False表示将软链接文件拷贝成硬链接,就是说对软链接来说,创建新的文件,为True表示拷贝软链接

示例：

folder1目录下有a.txt, b.py, tmp123.txt三个文件

递归拷贝folder1中的所有目录和文件,排除'*.py', 'tmp*'文件,folder2事前不能存在,且对folder2的父目录有可写权限

 

```
import shutil
# 方法1
shutil.copytree('folder1', 'folder2', ignore=shutil.ignore_patterns('*.py', 'tmp*'))

# 方法2
def ig(src, names):
    s = set()
    for name in names:
        if name.startswith('tmp') and name.endswith('.py'):
            s.add(name)
    return s
shutil.copytree('folder1', 'folder2', ignore=ig)

# 方法3
def ignore(src, names):
    ig = filter(lambda x: x.startswith('tmp') and x.endswith('.py'), names)
    return set(ig)
shutil.copytree('folder1', 'folder2', ignore=ignore)
```

## rmtree

rmtree(path, ignore_errors=False, onerror=None)

递归删除,同linux的rm -rf一样危险，慎用！

它不是原子操作，有可能删除错误，就会中断，已经删除的就删除了，不会撤销。

ignore_errors为true,表示忽略错误，False或omitted时，onerror生效。

onerror为callable,接受函数function、path和execinfo

## move

shutil.move(src, dst, copu_function=copy2)

递归移动文件、目录到目标，并返回目标。本身使用的是os.rename方法，如果不支持rename，如果是目录，先copytree再删除源目录。默认使用copy2方法。

 

```
os.rename('test1.txt', 'test1_new.txt')
os.rename('test1.txt', '/tmp/py/test1_new.txt')
```

## make_archive

make_archive('base_name', 'format', root_dir=r'path/to/file', ...): 归档压缩

创建压缩包并返回文件路径，例如：zip、tar

参数说明:

base_name: 压缩包的文件名,也可以是压缩包的路径.只是文件名时,则保存至当前目录,否则保存至指定路径.

format: 指定压缩格式,如zip,tar,bz,gz,xz等.

root_dir： 指定目标目录,即要压缩的文件夹路径(默认当前目录)

owner： 指定压缩文件的属主,默认当前用户

group： 指定压缩文件的属组,默认当前组

logger： 用于记录日志,通常是logging.Logger对象

 

```
# 将/data目录打包压缩放到当前程序目录
import shutil
ret = shutil.make_archive("data_bak", 'tar', root_dir='/data')

# 将/data目录打包放置/tmp目录下
import shutil
ret = shutil.make_archive("/tmp/data_bak", 'gztar', root_dir='/data')
```

说明:shutil对压缩包的处理是调用了ZipFile和tarfile两个模块来进行的.

 

```
import zipfile

# 压缩
z = zipfile.ZipFile('laxi.zip', 'w')
z.write('a.log')
z.write('data.data')
z.close()

# 解压
z = zipfile.ZipFile('laxi.zip', 'r')
z.extractall(path='.')
z.close()

##############################

import tarfile

# 压缩
>>> t=tarfile.open('/tmp/egon.tar','w')
>>> t.add('/test1/a.py',arcname='a.bak')
>>> t.add('/test1/b.py',arcname='b.bak')
>>> t.close()

# 解压
>>> t=tarfile.open('/tmp/egon.tar','r')
>>> t.extractall('/egon')
>>> t.close()
```