# ag-The silver search

在目录中`循环搜索`pattern(`默认即循环搜索`)。 类似grep，但是更快。 详情参阅`man ag`。

ag也是一个pattern搜索的神器，因为比grep快很多，所以我一直在用。

- 测试下来ag是默认是BREs搜索。
- ag默认搜索所有的文本文件，但是忽略掉隐藏文件，忽略.gitignore .ignore中指定的文件.


## 1. 常用选项

#### 重要
1. `-a， --all-types` 搜索所有类型的文件，包括binary，不包括隐藏文件
2. `-t, --all-text` 搜索所有的文本文件`不管.ignore的内容了`，不包括hidden
3. `-u, --unrestricted` 搜所有文件，包括binary/hidden； `-u(include hidden) > -a(include binary) > -t(include text)`
4. `-g PATTERN` 搜索匹配pattern的文件名; `可以用来搜索文件`
5. `-l` 只打印匹配了的文件名，不打印行
6. `-o, --only-matching` 同grep，只打印匹配字串，不是一行

#### 可能使用
1. `-G PATTERN` 只搜索文件名匹配pattern的文件
2. `--depth NUM` 搜索深度，-1代表无限制，默认是25
3. `-f, --follow` 追踪软连接，默认关闭
4. `-F, --fixed-strings` 同选项Q，为了兼容grep； `-Q, --litral`, 固定字符搜索
5. `-n, --norecurse` 不循环到查找下级目录，只是本级目录； `-r, --recurse` 循环搜索，默认
6. `-i` 大小写不敏感；  `-s` 大小写敏感； `-S` 自动大小写敏感（有大写就敏感，全小写则不敏感，默认）

#### `--list-file-types`
ag 支持仅搜索目录下某种语言的文件，使用`--list-file-types`选项可以查看ag可以专门搜索的文件类型。`选项通常是 --编译器/解释器/tool 的名字`

- `--cc` 只搜 .c .h .xs文件
- `--cpp` 只搜索 .cpp .cc .C .cxx .m .hpp .hh ...
- `--shell` 只搜索.sh .bash .csh .tcsh .ksh .zsh .gish文件
- `--make` .Makefiles .mk .mak
- `--python` .py
- `--html` .html .htm .shtml .xhtml


## 2. 常用举例

#### ag搜索的文件类型选项

下面三个文件中都有printf

```bash
➜  ls
grep_test.c   grep_test.o   grep_test.txt
```

`ag` 默认会搜索所有的`不在.ignore中的文本文件`，所以有两个匹配。
```bash
➜  ag printf
grep_test.c
6:	printf("hello world\n");

grep_test.txt
8:printf(var_a)
```

`ag -a` 会搜索`所有类型文件（除了hidden的，包括binary）`,所以二进制的被搜到了
```bash
➜  ag -a printf
grep_test.c
6:	printf("hello world\n");

grep_test.txt
8:printf(var_a)

Binary file grep_test.o matches.
```

`ag --cc` 只会搜索`c语言相关的文件`，所以txt的就没有搜索
```bash
➜ ag --cc printf
grep_test.c
6:	printf("hello world\n");
```

#### ag查找文件（相当于find）

`ag -g pattern` 查找目录中匹配pattern的文件

文件类型的选项还是起作用。
其他的只会打印找到的文件名，不会匹配内容。

```bash
➜ ag -g grep
grep_test.c
grep_test.o
grep_test.txt
➜ ag --cc -g grep
grep_test.c
```

## ag & grep 效率比对

使用ag & grep 在linux kernel里查找alloc_pages调用。

```bash
➜  linux-3.10.106 time ag --cc alloc_pages > /dev/null
ag --cc alloc_pages > /dev/null  1.25s user 2.67s system 116% cpu 3.367 total
➜  linux-3.10.106 time grep -R alloc_pages . >/dev/null
grep --color=auto --exclude-dir={.bzr,CVS,.git,.hg,.svn} -R alloc_pages . >   13.27s user 4.70s system 61% cpu 29.458 total
➜  linux-3.10.106 time fgrep -R alloc_pages . >/dev/null
fgrep -R alloc_pages . > /dev/null  9.06s user 0.56s system 99% cpu 9.638 total

```

#### 结果
| ag | 3.367s |
| grep | 29.458s |
| fgrep | 9.638s |

结果不言自明。

所以ag特点是：速度快， 可以分类搜索，可以当find用。 grep的功能几乎都有，但是可能兼容性比较差，嵌入式设备用的少，其他平台可能没有。


还有目前只能匹配BREs，而egrep可以EREs。