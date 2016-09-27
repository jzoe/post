---
layout: post
title: MarkDown基础语法
categories: [MarkDown]
tags: [MarkDown]
---

# MarkDown基础语法

## 代码高亮
- bash

```bash

[gongjz@localhost ~]$ cd Downloads/
[gongjz@localhost Downloads]$ tar zxvf percona-xtrabackup-2.0.8-587.tar.gz 
[gongjz@localhost Downloads]$ mv percona-xtrabackup-2.0.8 ~/app/

```
- java

```java

System.out.println("Hello Java");

```

- cpp
```cpp
/* CAUTION(?): Don't rename file_per_table during backup */
 static void
 xtrabackup_backup_func(void)
 {
 	struct stat stat_info;
 	LSN64 latest_cp;
 
 #ifdef USE_POSIX_FADVISE
 	fprintf(stderr, "xtrabackup: uses posix_fadvise().\n");
 #endif
 
 	/* cd to datadir */
 	if (chdir(mysql_real_data_home) != 0)
 	{
 		fprintf(stderr, "xtrabackup: cannot my_setwd %s\n", mysql_real_data_home);
 		exit(EXIT_FAILURE);
 	}
 	fprintf(stderr, "xtrabackup: cd to %s\n", mysql_real_data_home);
   ...
 }
```

## To-do List
- [x] 已完成
    - [ ] 未完成
    - [x] 完成
- [ ] 未开始

## 流程图 

> TB - top bottom（自上而下）

> BT - bottom top（自下而上）

> RL - right left（从右到左）

> LR - left right（从左到右）

```
graph LR
A[Kobe]-->B(Play Basketball)
B-->Traning
```


```
graph TD
A[Kobe]-->B(Play Basketball)
B-->C{Playoffs?}
C-->|Round-1| D[Supers]
C-->|Round-2| D[Supers]
C-->|Round-3| E((Rockets))

```


```
graph LR
A---B
C-->D
E-->| hello|F
H-- OK -->G
J==>K
L-. TEXT .->M
```

```
graph LR
    id1(Start)-->id2(Stop)
    style id1 fill:#f9f,stroke:#333,stroke-width:4px;
    style id2 fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5;
    click id1 callback "Tooltip for a callback"
    click id2 "http://www.github.com" "This is a tooltip for a link"
```

```
graph TD
    B["fa:fa-twitter for peace"]
    B-->C[fa:fa-ban forbidden]
    B-->D(fa:fa-spinner);
    B-->E(A fa:fa-camera-retro perhaps?);
```





## 序列图

```
sequenceDiagram
A->>B: How are you?
B-->>A: Great!
```

```
sequenceDiagram
    LOOP Condition
        A->>B: How are you?
        B->>A: Great!
    END
```

## 甘特图

```
gantt
dateFormat YYYY-MM-DD
title 甘特图示例
section 阶段一
需求收集: 2014-01-01, 7d
section 阶段二
开发: 2014-01-05, 5d
section 阶段三
测试: 2014-01-09, 9d
验收: 2014-01-17, 1d
```
## 表格

| 姓名   | 成绩   | 学号        |
| :----- | -----: | :---------: |
| Jim    | 99     | 007         |
| Bryant | 149    | 8           |
| 左对齐 | 右对齐 | 居中        |

## 标题
---
# 一级标题
## 二级标题
### 三级标题
---

## 有序列表
1. 列表1
2. 列表2
    1. 列表2.1
    2. 列表2.2
        1. 列表2.2.1

##　无序列表
- 列表1
    - 列表1.1
        - 列表1.1.1
        - 列表1.1.2
    - 列表1.2
- 列表2

## 引用
> 这是一段引用文字
-- by Kobe
> 然而并没有自动换行

> 这是下一段

## 粗体斜体
注意无需空格

*斜体*

**粗体**

***粗斜体***

## 插入链接与图片

[有道官网](http://note.youdao.com/)

![有道图标](http://note.youdao.com/favicon.ico)

## 分隔文段

这是段落1
***
这是段落2
---
这是段落3

---
这是段落4





