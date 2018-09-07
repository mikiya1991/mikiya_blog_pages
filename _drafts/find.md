# find

find其实不只是一个文件搜索工具，其实它是一个文件遍历工具， 并对遍历到的文件执行指定操作。
本文主要参照gnu版的find工具，参考自ubuntu上的`man find`。

## 1. 使用方式：
`find [options] [start-point...] [expression]`
参考上面的形式，find 从`start-point`目录开始遍历目录中的文件，对每个文件执行指定的`expression`表达式。

- `option` 影响整个遍历的一些选项，一般不常用。
- `start-point`如果不写，默认遍历当前的目录
- `expression`是find命令自己规定的一些表达式，是find命令的`最重要的部分`。

## 2. options
- `-P` 不跟进符号链接（默认行为）
- `-L` 跟进符号链接

	find命令的`expression`作用于文件的`stats`信息，这个信息符号链接没有，符号链接指向的文件才有；所以find命令的expression作用于指向的文件而不是符号链接本身。

- `-D debugoptions` 用于打印debug信息； `-Olevel` find命令遍历处理时的优化方式。都不常用


## 3. expression

expression通常用来指定：`如何找到匹配的文件，对文件做什么操作`。

expression可以分成不同的类型，并由不同类型组成：

- Tests:
	
	测试文件是否满足指定的要求，返回true/false。如`-empty`测试文件是否为空

- Actions: 
	
	对文件做处理，比如`-print`/`-delete`。 返回true/false，看action的结果。

- Global options: 
	
	全局选项，一般整个规定遍历的范围；对所有遍历文件的tests/actions操作都产生操作。总是返回true。

- Positional options:

	只对跟在其后的expression起作用的选项。总是返回true。

- Operators:

	连接其他expression；`-a` 逻辑与， `-o` 逻辑或；没有指定，默认`-a`


如果表达式没有指定`-prune`或者`-print` actions。则默认对所有expression为true的文件执行`-print`打印文件名。

##### 3.1 TESTS文件测试 - 重要

1. `-[i]name pattern`

	文件的`basename`（去掉路径前缀)`shell pattern`匹配pattern；pattern需要加双引号，防止shell解析

2. `-[i]regex pattern`

	整个文件名（含路径）正则匹配pattern。正则是emas RE；可以通过`-regextype` 选项修改正则类型

3. `-[i]path`
	
	整个文件名，`shell pattern`匹配pattern。不特殊处理`. /`符号，认为是普通字符

4. `-type`

	测试文件类型 [bcdf...],block（块设备文件）/char/f(regular)/directory/..

5. `-size [+-]?n[cwbkMG]`

	测试文件大小, c(bytes)/k(kb)/M(Mb)/G(Gb)

6. `perm /mode`， `perm -mode`



> [+-]?n:  +n 表示比n大的所有； -n表示比n小的所有； n表示等于n  
> [i]: 有i表示大小写不敏感，没有i表示大小写敏感

##### 3.2 TESTS - 可能使用
- `-empty`，`-executable`，`-readable`，`-writable`： 测试文件状态
- `-uid n` `-user name` `-gid n` `-group name`: 测试文件的uid/username/gid/groupname

- `-amin -cmin -mmin [+-]?n`: n分钟之前，被access/change(file stats)/modify(file data)
- `-anewer -cnewer -mnewer file`: access/change/modify比file更新
- `-atime -ctime -mtime [+-]?n`: n天之前（实际是n\*24h前）,被acess/change/modify过

#### 3.3 ACTIONS -重要

`-delete`

	删除文件，如果删除失败，会打印错误，并且影响最终find的exitcode。 find的expression从前往后执行，delete放前面会导致遍历的所有文件被删除。

`-exec command ;`
	- 每匹配到文件，执行command, expression结果是command的结果
	- command以`;`结束，不然find -exec后的都是command，直到`;`
	- command中用`{}`表示当前遍历的文件名，所有的`{}`都会被替换为文件名
	- 普通的`;` `{}`符号，需要转义`\;` `\{` `\}`，不然可能被shell解释展开

`-exec command {} +`
	- `-exec`的变体，但是只能将文件名放到command的最后一个参数
	- 总是返回true

`-execdir command ;`
`-execdir command {} +`

	对每个匹配到的文件的次级目录(直接上级目录)执行command。

`-ok command ;` , `-okdir command ;`

	询问用户，然后执行command

`-print`， `-print0`
	
	打印匹配的文件；返回true；print每个文件以结尾为`\n`，print0为null（如果通过管道交给其他程序处理，print0较为合适）。

`-printf format`
	
	返回true。c风格的格式化字符串，可以打印转义的时间/文件信息。`%p`文件名 `%y`type `%a`last access time。

`-prune`

	如果文件是目录，不再向下遍历。如果指定`-d`深度优先遍历，则不起作用，返回false。

`-quit`
	
	立即退出


#### 3.4 GLOBAL OPTIONS

`-d, --depth`: 深度优先模式，先处理目录的内容，再处理目录本身。
`-xdev, -mount`: 不遍历不同的文件系统
`-mindepth levels, -maxdepth levels`: 最大遍历深度，最小遍历深度；0表示 start-point本身。


#### 3.5 POSITIONAL OPTIONS

`-regextype type`: 指定其后的正则匹配的类型，影响`-regex`。


#### 3.6 OPERATIONS

`()`： 括号，改变计算优先级；`需要保护`，引号 or `\(\)`反斜杠，防止shell解释展开
`!, -not`： 逻辑非， `!`需要用引号反斜杠保护，防止shell展开
`expr1 expr2, expr1 -a expr2, expr1 -and expr2`： 逻辑与
`expr1 -o expr2, expr1 -or expr2`: 逻辑或
`expr1 , expr2 , expr3...`: list, 顺序执行，结果为最右的expr的结果


## 4. 举例

-name 选项pattern匹配要加上双引号，不加会被shell展开。
```bash
➜  /nfs find -name "*.bmp" | head -n 5 
./10/Alarm0101.bmp
./10/Alarm0085.bmp
./10/Alarm0074.bmp
./10/Alarm0083.bmp
./10/Alarm0047.bmp
```



-exec 命令中`;`需要转义，因为shell中会解释它；`\; ';' "";`都可以
```bash
➜  /nfs find  -maxdepth 1 -executable -type f -exec file {} ; 
find: missing argument to `-exec'

➜  /nfs find  -maxdepth 1 -executable -type f -exec file {} \;
./stop_report_oprofile.sh: POSIX shell script, ASCII text executable
./core: ELF 32-bit LSB core file MIPS, MIPS-I version 1 (SYSV), SVR4-style

➜  /nfs find  -maxdepth 1 -executable -type f -exec file {} ';'
./stop_report_oprofile.sh: POSIX shell script, ASCII text executable
./core: ELF 32-bit LSB core file MIPS, MIPS-I version 1 (SYSV), SVR4-style
```


-exec 指令中不能有shell的功能符号，如`|`管道 `>` 重定向 从下面输出看出`|`被当做参数传给了readelf。`|`需要shell解释它，但是这里看出其中没有shell参与。exec的命令不能有shell的功能。
```bash
➜  /nfs find  -maxdepth 1 -executable -type f -exec readelf -h {} '|' grep Machine \;

File: ./stop_report_oprofile.sh
readelf: Error: Not an ELF file - it has the wrong magic bytes at the start
readelf: Error: '|': No such file
readelf: Error: 'grep': No such file
readelf: Error: 'Machine': No such file
```