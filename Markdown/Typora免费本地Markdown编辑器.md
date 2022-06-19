## Typora 免费本地 Markdown 编辑器

　圈子里的大部分朋友在工作中，都在实用这款Markdown编辑器。虽然新版本已经开始收费，但是免费版本足够我们平时工作用。Markdown在技术开发方面之所以盛行，关键在于它是纯文本，不仅仅是文件超小、更在于它可以做差分处理，对项目管理非常有益。

Typora 免费版官方下载链接: [Windows + Linux](https://0xo.net/go/aHR0cHM6Ly93d3cudHlwb3JhLmlvL3dpbmRvd3MvZGV2X3JlbGVhc2UuaHRtbD91dG1fc291cmNlPTB4by5uZXQ)、[macOS](https://0xo.net/go/aHR0cHM6Ly93d3cudHlwb3JhLmlvL2Rldl9yZWxlYXNlLmh0bWw_dXRtX3NvdXJjZT0weG8ubmV0) 

- Windows

  https://download.typora.io/windows/typora-setup-x64-0.11.18.exe

- 所有版本 - **Windows + Linux**  
  https://www.typora.io/windows/dev_release.html?utm_source=0xo.net  
  1. [Windows x64](https://download.typora.io/windows/typora-update-x64-1117.exe)
  2. [Windows x86](https://download.typora.io/windows/typora-update-ia32-1117.exe)
  3. [Linux x64](https://download.typora.io/linux/typora_0.11.18_amd64.deb)



## 简介

- MarkDown  
  Markdown 是一种轻量级标记语言，它允许人们使用**易读易写**的纯文本格式编写文档。它看来更为简明、不易出错且易扩展。而且，它很容易做到只用键盘编辑（这对于不间断的打字有帮助）。Markdown非常适合**大量文本的写作或技术性的文章**，并且只需要很少的时间即可学习。

- Typora  
  Typora是一款轻量级Markdown编辑器，适用于macOS、Windows和Linux三种操作系统，是一款**免费**软件。与其他Markdown编辑器不同的是，Typora没有采用源代码和预览双栏显示的方式，而是采用所见即所得的编辑方式，实现了即时预览的功能，但也可切换至源代码编辑模式。
  - Typora删除了预览窗口，以及所有其他不必要的干扰，取而代之的是实时预览。
  - Markdown的语法因不同的解析器或编辑器而异，Typora使用的是[GitHub Flavored Markdown](https://link.segmentfault.com/?enc=j6s6DiuERNQQB6bxlEpPFQ%3D%3D.dPQJLO0O5oEqF6EJE2zOkjpd%2FysmvAmPGTxSL4HrVI9mtNLsLmOWNyAiHZft1LiLgTOjwIeHzyqcgWvI%2Fkbljw0GsWlrC57GwKxmnpnfTcM%3D)。


## 设置

- 菜单 →【**视图**】→【**大纲**】  
  勾选之后，对于编辑过程中找到相应的文件、及主题非常有帮助。



## Typora常用快捷键

- 加粗： `Ctrl/Cmd + B`
- 标题： `Ctrl/Cmd + H`
- 插入链接： `Ctrl/Cmd + K`
- 插入代码： `Ctrl/Cmd + Shift + C`
- 行内代码： `Ctrl/Cmd + Shift + K`
- 插入图片： `Ctrl/Cmd + Shift + I`
- 无序列表： `Ctrl/Cmd + Shift + L`
- 撤销： `Ctrl/Cmd + Z`
- 一级标题：快捷键为`Crtl + 1`，多级标题以此类推
- 切换原文和语法：`Ctrl/Cmd + /`

## 块元素

#### 换行符

在markdown中，段落由多个空格分隔。在Typora中，只需回车即可创建新段落。

#### 标题级别

\# 一级标题 快捷键为 Ctrl + 1
\## 二级标题 快捷键为 Ctrl + 2
......
\###### 六级标题 快捷键为 Ctrl + 6

#### 引用文字

\> + 空格 + 引用文字

> 引用效果

#### 无序列表

使用 * + - 都可以创建一个无序列表

- AAA
- BBB
- CCC

#### 有序列表

使用 ‘1.’+‘空格’ 2. 3. 创建有序列表

写法：`1. AAA`

1. AAA
2. BBB
3. CCC

#### 任务列表

```
- [ ] 不勾选
- [x] 勾选
```

- [ ] 不勾选
- [x] 勾选

#### 代码块

在Typora中插入程序代码的方式有两种：使用反引号 `（~ 键）、使用缩进（Tab）。

- 插入行内代码，即插入一个单词或者一句代码的情况，使用 `code` 这样的形式插入。
- 插入多行代码输入3个反引号（`） + 回车，并在后面选择一个语言名称即可实现语法高亮。

java

```java
public class Hello{
    public static void main(String[] args) {
        System.out.println("hello!");
    }
}
```

python

```python
def helloWorld():
    print("hello, world!")
```

#### 数学表达式

当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。根据需要加载 Mathjax 对数学公式进行渲染。

按下 `$$`，然后按下回车键，即可进行数学公式的编辑。

```shell
$$
\mathbf{V}_1\times\mathbf{V}_2 = \mathbf{X}_3
$$
```

\mathbf{V}_1\times\mathbf{V}_2 = \mathbf{X}_3**V**1×**V**2=**X**3

#### 插入表格

输入 `| id | name | number |`并回车。即可创建一个列表；快捷键 `Ctrl + T`弹出对话框；也可直接复制表粘贴进来。

|  id  | name | number |
| :--: | :--: | :----: |
|  1   | kxds |  101   |

#### 脚注

```
[^1]:脚注内容
```

你可以创建一个脚注，像这样[1](https://segmentfault.com/a/1190000040575971#fn-1).

注意：该例子脚注标识是1，脚注标识可以为字母数字下划线，但是暂不支持中文。脚注内容可为任意字符，包括中文。

#### 分割线

输入 `***` 或者 `---` 再按回车即可绘制一条水平线，如下：

------

#### 目录（TOC）

输入 `[toc]` 然后回车，即可创建一个“目录”。TOC从文档中提取所有标题，其内容将自动更新。

> Typora支持TOC自动生成目录

```
[toc]
```

非常可惜的是，这个功能在Github上不受支持，仅仅在Gitlab上支持。而且各个浏览器在加挂Markdown Viewer后，都不支持改功能。

### 3.跨度元素

跨度元素即图片，网址，视频等，在Typora中输入后，会立即载入并呈现。

#### 链接

这是一个带有标题属性的 `[链接](https://www.typora.io/ "标题")`
这是一个没有标题属性的 `[链接]https://www.typora.io/)`

这是一个带有标题属性的[链接](https://link.segmentfault.com/?enc=2wXtThD1ftoBCKSkMlXQAg%3D%3D.jXsHiTZi9dV5G4E7NfjUH%2B%2FVJkkJ0SI4MD6BTp%2B5LIY%3D).
这是一个没有标题属性的[链接](https://link.segmentfault.com/?enc=58exPiaF%2Be9EPSh7M5BNWw%3D%3D.0WFgzXgLwVf5nXu5bpAcxboMm8BV7wb%2BMgjAj36yhQU%3D).

**注：ctrl+鼠标右键访问链接**

#### 网址

Typora允许用<括号括起来>, 把URL作为链接插入。

百度：`<www.baidu.com>`

百度：www.baidu.com

#### 图片

```less
![显示的文字](C:\Users\Hider\Desktop\echart.png)
![显示的文字](C:\Users\Hider\Desktop\echart.png “图片标题”)
```

除了以上2种方式之外，还可以直接将图片拖拽进来，自动生成链接。

![演示图](https://segmentfault.com/img/remote/1460000040575974)

#### 斜体

使用 `*单个星号*` 或者 `_单下划线_` 可以字体倾斜。快捷键 `Ctrl + I`

*斜体*

#### 加粗

使用 `**两个星号**` 或者 `__两个下划线__` 可以字体加粗。快捷键 `Ctrl + B`

**加粗**

#### 加粗斜体

使用`***加粗斜体***`可以加粗斜体。

***加粗斜体\***

#### 代码标记

标记代码使用大于符。

`>`使用该 `printf()`功能

> 使用该 `printf()`功能

#### 删除线

使用`~~删除线~~` 快捷键 `Alt + Shift + 5`

```
~~删除线~~
```

~~删除线~~

#### 下划线

\下划线 -- 无法执行

参考另一篇文章，可执行。

通过`<u>下划线的内容</u>` 或者 快捷键`Ctrl + U`可实现下划线

<u>下划线的内容</u>

#### 表情符号

Github的Markdown语法支持添加emoji表情，输入不同的符号码（两个冒号包围的字符）可以显示出不同的表情。

```
:smile
```



#### 下标

`H~2~O` (需在设置中打开该功能)

H~2~O

可以使用 `<sub>文本</sub>`实现下标。

```
H<sub>2</sub>O
```

H2O

#### 上标

`X^2^`(需在设置中打开该功能)

X^2^

可以使用`<sup>文本</sup>`实现上标。

```
X<sup>2</sup>
```

X2

#### 高亮

`==高亮==`(需在设置中打开该功能)

==我是最重要的==

#### 文本居中

使用 `<center>这是要居中的内容</center>`可以使文本居中

<center>这是要居中的文本内容</center>

#### 转义

Markdown 使用了很多特殊符号来表示特定的意义，如果需要显示特定的符号则需要使用转义字符，Markdown 使用反斜杠转义特殊字符：

**文本加粗**
**正常显示星号**

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

```markdown
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```

### 4.HTML

支持HTML

不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。

目前支持的 HTML 元素有：`<kbd> <b> <i> <em> <sup> <sub> <br>`等 ，如：

```css
使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑
<kbd> </kbd> -- 白色框框
```

效果：使用 Ctrl+Alt+Del 重启电脑

-- 白色框框

#### 嵌入内容

支持iframe-based嵌入代码

```xml
<iframe src="//www.runoob.com"></iframe>
```

<iframe src="//www.runoob.com"></iframe>

##### 视频

> < video src=”[https://typora.io/img/beta.mp4](https://link.segmentfault.com/?enc=PVNzyrkNRr31IaqO9wPAag%3D%3D.Yg4TuTllDpfQv%2BS%2BSXuEinieFe1Zwhu8gBnj1dJ0sek%3D)” />

<video src="https://typora.io/img/beta.mp4" />

### 5.参考

> 参考链接1：[Typora入门（中文版）](https://link.segmentfault.com/?enc=wX%2FyuIn%2Bo2RY%2Bh7bSj0S%2Fw%3D%3D.oh1UxUrLgCwgAy1%2BQj9kwaawalpWfvbrXu692dUiozg%2B2lQCV6%2FcY6ZpNKMQJWPOsn%2F8cOT%2FOL2acP42ZqYpa56pc5vBzrHS63maVi4ZSoUql0vDI3Xv0mODSep9LB4QxvZBpj5hwlgFJEW2G%2BGvng%3D%3D)
>
> 参考链接2：https://segmentfault.com/a/1190000040575971