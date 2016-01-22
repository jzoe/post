---
layout: post
title: "markdown基础语法"
description: ""
category: 工具
tags: [markdown]
---
个人认为文章尽可能的简洁是增强可读性的最优方法，故日后我的文章中只推崇用最少的语法实现最好的可读性。

这篇文章主要介绍了MarkDown的常用语法，在文章中尽可能不要使用超出以下语法的范围。

## 空行
> `<br  />`

效果是这样的：
>空行1<br  />
>
>空行2
><br  />

##  代码段
下面是一段C语言代码段，将所有的代码行缩进一个tab，并在代码之前空一行，就可将缩进区域转为code块：

```c
#include <stdio.h>
int main(int argc, char* argv[])
{
	unsigned int a = -20;
	int b = 10;
	if( a + b > 6 )
	{
		puts(">6");
	}
	else
	{
		puts("<6");
	}
	return 0;
}
```
在代码中，使用```标记的效果如下：
```cpp
#include <stdio.h>
int main(int argc, char* argv[])
{
	unsigned int a = -20;
	int b = 10;
	if( a + b > 6 )
	{
		puts(">6");
	}
	else
	{
		puts("<6");
	}
	return 0;
}
```

在代码中用`{% hightlight %}... {% endhightlight %}`标记结尾，效果如下：

{% highlight cpp linenos %}
#include <stdio.h>
int main(int argc, char* argv[])
{
	unsigned int a = -20;
	int b = 10;
	if( a + b > 6 )
	{
		puts(">6");
	}
	else
	{
		puts("<6");
	}
	return 0;
}
{% endhighlight java %}


在代码前添加一个tab，效果如下：

	#include <stdio.h>
	int main(int argc, char* argv[])
	{
		unsigned int a = -20;
		int b = 10;
		if( a + b > 6 )
		{
			puts(">6");
		}
		else
		{
			puts("<6");
		}
		return 0;
	}

如果想在行内高亮某行代码，可用`...`来注释，效果是这样的 `这是代码`，`void log()`。

## 链接
1. 在文字后直接添加链接地址：
>`[这是一个链接](http://www.gongjingzhou.cn)`

	`[这是一个链接](http://www.gongjingzhou.cn)`

效果是这样的：[这是一个链接](http://www.gongjingzhou.cn)

2. 在文章内添加类似于`[1]`这样的标记：
	
>`[这是链接1][1]`
>
>`[这是链接2][2]`

	[这是链接1][1]
	[这是链接2][2]
	
在文章末尾添加如下记录：

>`#注意需要在连接列表前添加一个空行，且列表需写在文章最末尾`
>
>`[1]: http://www.gongjingzhou.cn`
>
>`[2]: http://www.gongjingzhou.cn`

	#注意需要在连接列表前添加一个空行，且列表需写在文章最末尾
	
	[1]: http://www.gongjingzhou.cn
	[2]: http://www.gongjingzhou.cn

效果是这样的：[这是链接1][1], [这是链接2][2]

## 插入图片
- 在文字后直接添加照片地址：
>`![这是一张照片](http://7xq43l.com1.z0.glb.clouddn.com/test-photo.jpg)`

	![这是一张照片](http://7xq43l.com1.z0.glb.clouddn.com/test-photo.jpg)

效果是这样的：

>![这是一张照片](http://7xq43l.com1.z0.glb.clouddn.com/test-photo.jpg)

- 类似于链接的方式：
>`![这是一张照片][3]`

	![这是一张照片][3]

效果是这样的：

>![这是一张照片][3]

## 关于列表
### 有序列表
>`1. 这是列表项1`
>
>`2. 这是列表项2`
>
>`3. 这是列表项3`

	1. 这是列表项1
	2. 这是列表项2
	3. 这是列表项3

效果是这样的：
1. 这是列表项1
	这是项目1的内容；
2. 这是列表项2
	这是项目2的内容；
3. 这是列表项3
	这是项目3的内容；

### 有序级联列表

>`1. 这是列表项1`
>
>	`1. 这是列表项1.1`
>
>	`2. 这是列表项1.2`
>
>`2 这是列表项2`

	1. 这是列表项
		1. 这是列表项1.1
		2. 这是列表项1.2
	2 这是列表项2

效果是这样的：
1. 这是列表项1
	1. 这是列表项1.1
	2. 这是列表项1.2
2. 这是列表项2

### 无序列表
>`- 无序列表项目1`
>
>`+ 无序列表项目2`
>
>`* 无序列表项目3`

	- 无序列表项目1
	+ 无序列表项目2
	* 无序列表项目3

效果是这样的：
> - 无序列表项目1
> + 无序列表项目2
> * 无序列表项目3

### 无序级联列表
>`* 无序列表项目1`
>
>`	* 无序列表项目1.1`
>
>`* 无序列表项目2`
>
>`	* 无序列表项目2.1`

	* 无序列表项目1
		* 无序列表项目1.1
	* 无序列表项目2
		* 无序列表项目2.1

效果是这样的：
>* 无序列表项目1
>	* 无序列表项目1.1
>* 无序列表项目2
>	* 无序列表项目2.1

## 关于粗体、斜体
>`**这是粗体**`
>
>`*这是斜体*`
>
> `***这是粗斜体***`

	**这是粗体**
	*这是斜体*
	***这是粗斜体***

效果是这样的：
>**这是粗体**
>
>*这是斜体*
>
>***这是粗斜体***

## 关于分割线
>`下面细线粗字：`
>
>`---`
>
>`添加一个空行后是粗线：`
>
>`---`

	下面细线粗字：
	---
	添加一个空行后是粗线：
	
	---

效果是这样的：
>细线：
>
>粗线：
>
>---

## 关于标题
标题语法如下：
>`# 这是一级标题`
>
>`## 这是二级标题`
>
>`### 这是三级标题`
>
>`#### 这是四级标题`

	# 这是一级标题
	## 这是二级标题
	### 这是三级标题
	#### 这是四级标题
	
效果是这样的：
># 这是一级标题
>## 这是二级标题
>### 这是三级标题
>#### 这是四级标题


## 参考
1. [github风格的markdown语法](https://github.com/riku/Markdown-Syntax-CN/blob/master/syntax.md)

[1]: http://www.gongjingzhou.cn
[2]: http://www.gongjingzhou.cn
[3]: http://7xq43l.com1.z0.glb.clouddn.com/test-photo.jpg