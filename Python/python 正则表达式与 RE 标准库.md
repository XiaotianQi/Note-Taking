> 如果有个问题需要用正则表达式来解决，那就成了两个问题。

正则表达式本质上只做一件事，那就是编写一个表达式“字符串”，然后用这个字符串去匹配目标文本。核心的核心，都在编写这个“字符串”表达式上面。

正则表达式是从左到右依次匹配，如果满足了某个分支的话它就不会再管其他分支了。一个子表达式开始匹配的位置，是从前一子表达匹配成功的结束位置开始。

python中re模块，常用的功能函数包括：`compile()`、`search()`、`match()`、`split()`、`findall()`、`sub()`。

***

## 正则表达式基本概念

### 字符串组成

对于字符串 `'123'` 而言，包括三个字符四个位置。如下图所示：

![字符串组成](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/re-1.png)

### 占有字符和零宽度

占有字符与零宽度，是针对匹配的内容是否保存到最终的匹配结果中而言的。

* 占有字符：如果一个子正则表达式匹配到的是字符，且会被保存到最终的结果中，那个这个子表达式就是占有字符，比如 /ha/ （匹配 ha ）就是占有字符的；
* 零宽度：只匹配一个位置，或者是匹配的内容并不保存到匹配结果中。如果一个子正则匹配的是位置，而不是字符，或者匹配到的内容不保存在结果中，那么这个子表达式是零宽度的，比如 `read(?=ing)` 匹配 reading ，但是只将read放入结果中，其中的`(?=ing)`就是零宽度的，代表一个位置。


零宽度的匹配，它匹配到的内容不会保存到匹配结果中去，最终匹配结果只是一个位置而已。

占有字符是互斥的，零宽度是非互斥的。也就是一个字符，同一时间只能由一个子表达式匹配，而一个位置，却可以同时由多个零宽度的子表达式匹配。

### 控制权和传动

正则表达式由左到右依次进行匹配，通常情况下是由一个表达式取得控制权，从字符串的的某个位置进行匹配，一个子表达式开始尝试匹配的位置，是从前一子表达匹配成功的结束位置开始。

如果表达式一是零宽度，那表达式一匹配完成后，表达式二匹配的位置还是原来表达式已匹配的位置。也就是说它匹配开始和结束的位置是同一个。

***

## 正则表达式符号和语法

以下字符都可以被称作元字符，只不过根据功能的不同进行了进一步的分类。

### 普通字符

字母、数字、汉字、下划线、以及没有特殊定义的符号，都是"普通字符"。正则表达式中的普通字符，在匹配的时候，只匹配与自身相同的一个字符。

### 元字符

|元字符|说明|
|:-|:-|
|`.`|匹配除换行符`\n`以外的任意字符|
|`[]`|匹配字符集中的任意字符|
|`()`|对表达式进行分组，将圆括号内的内容当做一个整体，并获得匹配的值|
|`x|y`|逻辑或操作符，匹配 `x` 或者 `y`|
|`\d`|匹配任意数字，`[0-9]`|
|`\s`|匹配任意空白字符，`[\t\n\r\f\v]`|
|`\w`|匹配包括下划线在内任意字母、数字、**中文**任意字符|

**tips**：

元字符在未指定重复次数时，均只匹配一个字符。

`\w` 在支持 ASCII 码的语言中，等价于 `[a-zA-Z0-9_]`。

`\w` 在支持 Unicode 的语言中，例如 python。默认情况下，除了可以匹配 `[a-zA-Z0-9_]` 外，还可以匹配一些 Unicode 字符集，如汉字，全角数字等等。

### 定位符

|定位符|说明|示例|
|:-|:-|:-|
|`^`|匹配字符串开头||
|`$`|匹配字符串结尾||
|`\b`|匹配字符串的开始或结尾||
|`\B`|匹配字符串除去开始或结尾的部分||

### 反义元字符

|元字符|说明|示例|
|:-|:-|:-|
|`\B`|匹配字符串除去开始或结尾的部分||
|`\D`|`[^0-9]`||
|`\S`|匹配任意非空白字符，`[^\t\n\r\f\v]`||
|`\W`|匹配 `\w` 之外的任意字符||
|`[^]`|匹配非字符集中的任意字符|`[^abc]`|

**tips**：

上表均与元字符中小写形式表示相反的匹配范围。

`[^]` 排除型字符组，表示匹配任意一个未列举的字符。同时，也支持字符分组，例如 `[^0-9]` 表示除数字外的任意一个字符。例如 `[^abc]` 表示除字符 `a`、`b`、`c` 外的任意一个字符。

### 限定符

|限定符|前面正则表达式匹配次数|示例|
|:-|:-|:-|
|*|0 次或多次||
|+|至少 1 次||
|?|0 次或 1 次||
|{n}|n 次||
|{n,}|至少 n 次||
|{n,m}|至少 n 次，但不多于m次||

***

### 匹配优先级

相同优先级的从左到右进行运算，不同优先级的运算先高后低。各种操作符的优先级从高到低如下：

|          操作符           | 描述 |
| :-----------------------: | :--: |
|             `\`             | 转义符 |
|    `()`, `(?:)`, `(?=)`, `[]`    | 圆括号和方括号 |
| `*`, `+`, `?`,`{n}`, `{n,}`, `{n,m}` | 限定符 |
| `^`, `$`, `\any metacharacter` | 位置和顺序 |
|             \|             | “或”操作 |

***

### 贪婪匹配与非贪婪匹配

re 模块默认采用贪婪匹配，即输出匹配最长的子串。限定符默认是贪婪匹配，后缀 `?` 可以转换为非贪婪匹配。例如：`*?`与 `+?` 与 `??`，均只取第一个匹配结果。`{n,m}?` 只匹配 `n 次`。

```python
import re

test_str = "Words, 你, words."
regEx = re.compile(r'\w+')		
print(regEx.findall(test_str))	# ['Words', '你', 'words']
regEx = re.compile(r'\w+?')
print(regEx.findall(test_str))	# ['W', 'o', 'r', 'd', 's', '你', 'w', 'o', 'r', 'd', 's']
regEx = re.compile(r'\w{1,2}')
print(regEx.findall(test_str))	# ['Wo', 'rd', 's', '你', 'wo', 'rd', 's']
regEx = re.compile(r'\w{1,2}?')
print(regEx.findall(test_str))	# ['W', 'o', 'r', 'd', 's', '你', 'w', 'o', 'r', 'd', 's']
```

对于 `\w\d*\w` 与 `\w\d*?\w` 两个正则表达式，匹配过程基本一致，只不过就是 `\d*` 和 `\d*?` 匹配与回溯过程因优先级有所区别而已。对于 `\d*?` 来说，回溯和匹配是交替进行的。而 `\d*` 在最后一次成功匹配前，只记录回溯，匹配失败后，进行回溯。

贪婪匹配会优先尝试匹配，达到最长成功匹配时(即遇到第一个匹配失败时)，才会交出控制权，并记录回溯状态。

非贪婪匹配会优先尝试忽略匹配，只要找到一个匹配成功项，并记录回溯状态。当然也会取得最长成功匹配，只是优先级不高。

提取 `boooooooob`:

```python
import re

test = 'a1boooooooobb123'

regEx = re.compile(r'.*(b.*b)')
print(regEx.findall(test))			# ['bb']
regEx = re.compile(r'.*(b.*?b)')
print(regEx.findall(test))			# ['bb']
regEx = re.compile(r'.*?(b.*?b)')
print(regEx.findall(test))			# ['boooooooob']
```

可以改写为：

```python
import re

test = 'a1boooooooobb123'

regEx = re.compile(r'(b.*b)')
print(regEx.findall(test))			# ['boooooooobb']
regEx = re.compile(r'(b.*?b)')
print(regEx.findall(test))			# ['boooooooob']
```

提取 '南京大学'：

```python
import re

test = 'study in 南京大学'

regEx = re.compile(r'.*([\u4E00-\u9FA5]+大学)')
print(regEx.findall(test))						# ['京大学']
regEx = re.compile(r'.*?([\u4E00-\u9FA5]+大学)')
print(regEx.findall(test))						# ['南京大学']
```

可以直接写为 `([\u4E00-\u9FA5]+大学)`

***

## 部分字符解释

### 分支条件 `|`

当一个字符串具有多种规则时，采用分支结构来匹配，用 `|` 把不同的规则分开，表示多个子表达式之间或的关系。

`|` 是以 `()` 限定范围的，如果在 `|` 的左右两侧没有 `()` 来限定范围，那么它的作用范围即为 `|` 左右两侧整体。

正则表达式是从左到右依次匹配，如果满足了某个分支的话它就不会再管其他分支了。匹配成功，则不再尝试另一个分支，只有这一分支匹配失败时，才会去尝试另一个分支。

用 `(a|ab)` 匹配 `ab` 时，只能匹配 `a`，但是如果用 `(ab|a)`，则可以匹配 `ab`。

### 单词边界 `\b`

`\b` 匹配单词边界，不匹配任何字符。`\b` 在字符组外表示单词边界，但是在字符组内 `[\b]` 表示退格符。

`\b` 匹配的只是一个位置，这个位置的一侧是构成单词的字符，另一侧为非单词字符、字符串的开始或结束位置。`\b` 是零宽度的。

`\b` 一般应用在需要匹配某一单词字符组成的子串，但这一字符不能包含在同样由单词字符组成的更长的子串中。比如要替换掉一段英文中的单词 `to`，而 `today` 显然不在替换的范围内，所以正则可以用 `\bto\b` 来限定。

### `[]` 与 `()` 的区别

`[]` 可以使用连字符 `-` 连接两个字符，来表示一个字符范围。需要注意的是，`-` 前后的两个字符是有顺序的，即使用相同的编码时，后面的字符码位应大于或等于前面字符的码位。例如 `[a-z]` 表示任意一个小写字母。而在程序中使用 `[z-a]` 则会报范围的顺序颠倒这样的异常。

大部分在正则中有特殊意义、在匹配其本身时需转义的字符，在 `[]` 内是不需要转义的。必须转义的只有 `\`、`[` 和 `]`，而 `^` 出现在 `[]` 开始位置， `-` 前后构成范围区间时，需要转义，出现在其它位置不需要转义。例如 `[\^.$^{\[(|)*+?-\\]`。  

`[]` 表示匹配的字符在 `[]` 中，并且只能出现一次，并且特殊字符写在 `[]` 会被当成普通字符来匹配。例如 `[(a)]`，会匹配 `(`、`a`、`)`这三个字符。

`()` 内的内容表示的是一个子表达式，`()` 本身不匹配任何东西，也不限制匹配任何东西，只是把括号内的内容作为同一个表达式来处理，例如 `(ab){1,3}`，就表示 `ab` 一起连续出现最少1次，最多3次。如果没有括号的话，`ab{1,3}` 表示 `a` 后面紧跟的 `b` 出现最少1次，最多3次。

`()` 将其里面的内容作为一组，这就是与 `[]` 不同的地方，例如：`(com|cn|net)` 只能是 `com` 或 `cn` 或 `net`。`(ab){1,3}` 就表示 `ab` 一起连续出现最少1次，最多3次，区别于 `ab{1,3}` 和 `[ab]{1,3}`。

* 匹配连续字符串

|表达式|内容|
|:-|:-|
|`abc+`|ab与c匹配一次或多次|
|`[abc]+`|abc 中任意一个字符匹配一次或多次|
|`(abc)+`|abc 为整体匹配一次或多次|

```python
test = "xabcabcxxabccabbc"
print(re.findall(r'abc+', test))	# ['abc', 'abc', 'abcc']
print(re.findall(r'[abc]+', test))	# ['abcabc', 'abccabbc']
print(re.findall(r'(abc)+', test))	# ['abc', 'abc']
```

`[^abc]` 与 `(?!abc)` 同理，只不过当表示 不含有 `abc` 连续字符串时写成 `[^(abc)]` 是达不到目的的。

```python
# 取出格式为<...>，但<>中不是abc的内容
test = "<abc><axx><bxx><cxx><xxx>"
print(re.findall(r'<[^abc>]*>', test))		# ['<xxx>']
print(re.findall(r'<(?!abc)[^>]*>', test))	# ['<axx>', '<bxx>', '<cxx>', '<xxx>']
```

关于`()`的功能，在另一个文章详细讲解，大致如下：

`()` 指分组。大概分为以下四种：

* 捕获组
* 非捕获组
* 反向引用
* 断言

***

## re 标准库

|函数|说明|返回值|
|:-|:-|:-|
|`re.compile(pattern[, flags=0])`|编译表达式，返回一个正则表达式对象，方便重复调用|re对象|
|`re.match(pattern, string[, flags=0])`|尝试从字符串的起始位置匹配。匹配成功返回一个 `Match` 对象，如果起始位置匹配不成功的话，返回 `None`|在字符串开头匹配到的对象或者None|
|`re.search(pattern, string[, flags=0])`|扫描整个字符串并返回第一个成功的匹配，是一个 `Match` 对象|第一个匹配到的对象或者None|
|`re.findall(pattern, string[, flags=0])`|找到 `RE` 匹配的所有字符串，一个 `list` 对象返回|所有匹配到的字符串列表|
|`re.finditer(pattern, string[, flags=0])` |找到 `RE` 匹配的所有字符串，将其作为一个迭代器返回||
|`re.sub(pattern, repl, string[, count=0, flags=0]`|替换匹配到的字符串|完成替换后的新字符串|
|`re.subn(pat,repl, string[,count=0,flags])`|在替换字符串后，同时报告替换的次数|完成替换后的新字符串及替换次数|
|`re.split(pattern, string[, maxsplit=0])`|按指定字符串分割|分割后的字符串列表|
|`re.finditer(pattern, string,flags)`|将所有匹配到的项生成一个迭代器|所有匹配到的字符串组合成的迭代器|

参数说明：

```text
- pattern	：正则表达式
- string	：需要匹配的字符串
- flags		：修饰符，用于控制正则表达式的匹配方式，如：是否区分大小写，多行匹配等等
- repl		：替换的字符串，也可作为一个函数
- count		：模式匹配后替换的最大次数，默认0表示替换所有匹配
- maxsplit	：最大分割次数
```

- `re.compile(pattern[, flags=0])`

`re.compile()` 用于编译正则表达式，生成一个正则表达式 `Pattern` 对象，便于复用。该对象拥有一系列方法用于正则表达式匹配和替换。

|修饰符|说明|
|:-|:-|
|`re.I`|忽略大小写|
|`re.L`|表示特殊字符集 `\w`、`\W`、`\b`、`\B` 依赖于当前环境|
|`re.M`|多行模式，影响 `^` 和 `$`|
|`re.S`|使 `.` 匹配包括换行在内的所有字符|
|`re.U`|根据Unicode字符集解析字符。这个标志影响 `\w`、`\W`、`\b`、`\B`|
|`re.X`|该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。|

* `MatchObject` 可调用的方法

`re.match()` 与 `re.search()` 成功匹配后返回的均是 `Match Object`。`Match Object` 具有以下方法：

|方法|作用|
|:-|:-|
|`group(num=0)`|获得指定分组匹配的字符串。参数默认值为0，获取整个匹配结果|
|`groups()`|获取整个匹配结果|
|`groupdict()`|将命名捕获组的名称与匹配结果组合成一个 `dict`返回|
|`start()`|起始匹配位置，参数默认值为 0|
|`end()`|结束匹配位置，参数默认值为 0|
|`span()`|返回 (start(group), end(group))|

* `re.match(pattern, string[, flags=0])`

返回 `MatchObject` 或 `None`。

```python
test = "abc"
print(re.match(r'a', test).group())	# a
print(re.match(r'a', test).span())	# (0, 1)
print(re.match(r'b', test))			# None
```

```python
test = "2 is bigger than 1"
regEx = re.compile(r'(.*) is (.*?) .*')
print(regEx.match(test).group())		# 2 is bigger than 1
print(regEx.match(test).span())			# (0, 18)
print(regEx.match(test).group(1))		# 2
print(regEx.match(test).group(2))		# bigger
regEx1 = re.compile(r'(.*) is (.*) .*')
print(regEx1.match(test).group(2))		# bigger than
```

留意其中，第二个捕获中使用非贪婪模式带来的影响。

* `re.search(pattern, string[, flags=0])`

返回 `MatchObject` 或 `None`。

```python
test = "abc"
print(re.search(r'a', test).span())	# (0, 1)
print(re.search(r'b', test).span())	# (1, 2)
```

* `re.findall(pattern, string[, flags=0])`

返回 `list`。

浏览全部字符串，匹配所有合规则的字符串，匹配到的字符串放到一个列表中，未匹配成功返回空列表。

一旦匹配成，再次匹配，是从前一次匹配成功的，后面一位开始的，也可以理解为匹配成功的字符串，不在参与下次匹配。

```python
test = "a1b2c3"
print(re.findall(r'[a-z]', test))	# ['a', 'b', 'c']
```

**`re.findall()`和`re.match()`、`re.search()`的区别：**

后两者都是单值匹配，找到一个就忽略后面，直接返回不再查找了。并且，返回Match Object。

`re.findall()`是全文查找，返回值是一个所有匹配到的字符串的列表。

`re.match()`和`re.search()`的不同之处在于：

`re.match`只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；

而`re.search`匹配整个字符串，直到找到一个匹配。

* `re.finditer(pattern, string[, flags=0])`

与 `re.findall()` 作用相同，但返回的是一个 `iter`，同时对于每一次匹配，迭代器都返回一个 `Match object`

```python
test = "a1b2c3"
matches = re.finditer(r'[a-z]', test)
for i in matches:
    print(i.group())
```

* `re.sub(pattern, repl, string[, count=0, flags=0])`

将字符串中匹配结果，用另一个字符串 `repl` 替换。如果匹配过程失败，则不进行替换。`Repl` 既可以是字符串也可以是一个函数或者变量。

若未指定 `count`，则默认替换全部。

```python
test = "a1b2c3"
print(re.sub(r'2', 'b', test))	# a1bbc3
```

分组引用:

将形式为 11/27/2012 的日期字符串改成 2012-11-27：

```python
In [31]: text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'

In [32]: re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
Out[32]: 'Today is 2012-11-27. PyCon starts 2013-3-13.'
```

自定义替换函数：

替换日期

```python
import re

def change_date(m):
    from calendar import month_abbr
    mon_name = month_abbr[int(m.group(1))]
    return '{} {} {}'.format(m.group(2), mon_name, m.group(3))

s = 'We will fly to Thailand on 10/31/2016'
pattern = r'(\d+)/(\d+)/(\d+)'
print(re.sub(pattern, change_date, s))	# We will fly to Thailand on 31 Oct 2016
```

忽略大小写的方式搜索与替换文本字符串，跟被匹配字符串的大小写保持一致：

```python
def matchcase(word):
    def replace(m):
        text = m.group()
        if text.isupper():
            return word.upper()
        elif text.islower():
            return word.lower()
        elif text[0].isupper():
            return word.capitalize()
        else:
            return word
    return replace

text = 'UPPER PYTHON, lower python, Mixed Python'
text_sub = re.sub('python', matchcase('snake'), text, flags=re.IGNORECASE)
print(text_sub)	# UPPER SNAKE, lower snake, Mixed Snake
```

* `re.subn(pattern, repl, string[, count=0, flags=0])`

与 `re.sub()` 调用相同，但 `re.subn()` 返回的是一个 `tuple`，包含替换后的字符串和替换次数。

```python
test = "a1b2c3"
print(re.sub(r'2', 'b', test))	# ('a1bbc3', 1)
```

* `re.split(pattern, string[, maxsplit=0])`

`re.split()` 用来分割字符串，re模块的`re.split()`方法和字符串的`str.split()`方法很相似，都是利用特定的字符去分割字符串。但是re模块的split()可以使用正则表达式。

利用分组的概念，`re.split()`方法还可以保存被匹配到的分隔符，这个功能非常重要！

```python
In [2]: s = "8+7*5+6/3"

In [3]: a_list = re.split(r'([\+\-\*\/])', s)

In [4]: a_list
Out[4]: ['8', '+', '7', '*', '5', '+', '6', '/', '3']
```

***

参考博客：

[ 正则基础 ](https://blog.csdn.net/lxcnn/article/category/538256)，-过客-

[浅析正则表达式—（原理篇）](https://www.cnblogs.com/dwlsxj/p/Regex.html)，battleheart