---
layout: post
title: R 语言
---

# 基础

## 变量赋值

```r
x <- 1
1 -> x
```

## 包管理

install.packages("gclus")

library(gclus)

## 数据结构
对象（object）是指可以赋值给变量的任何事物，包括常量、数据结构、函数，甚至图形。

### 向量
向量是用于存储数值型、字符型或逻辑型数据的一维数组。单个向量中的数据必须拥有相同的类型或
模式（数值、字符型或逻辑型）。

执行组合功能的函数c()可用来创建向量：

```r
# 赋值
a <- c(1,2,4,8,-1)
b <- c("one", "apple")
c <- c(TRUE, FALSE, TRUE, TRUE)

# 读取
> a[1]
[1] 1
> a[c(1,2)]
[1] 1 2
> a[2:4]
[1] 2 3 4
```

### 矩阵
矩阵是一个二维的数组，每个元素都有相同的模式。

可以用 matrix 函数创建，一般格式是：

```R
myMatrix = matrix(向量, nrow=行数, ncol=列数, byrow=是否安行填充,
  dimnames=list(row名称向量, col名称向量))


> m <- matrix(1:20, nrow=4, ncol=5, dimnames=list(c("r1", "r2", "r3", "r4"), c("c1", "c2", "c3", "c4", "c5")))
> m
   c1 c2 c3 c4 c5
r1  1  5  9 13 17
r2  2  6 10 14 18
r3  3  7 11 15 19
r4  4  8 12 16 20
> m[,1]
[1] 1 2 3 4
> m[1,]
[1]  1  5  9 13
```

矩阵都是二维的，多维是不妨使用数组，元素有多重模式时，使用数据框。

### 数组
数组是矩阵的一个自然推广，每个元素也需要有相同的形式。

用array函数创建：


```R
myArray <- array(数据向量, dimensions=各个纬度下标最大值向量, dimnames=各纬度名称列表)

> dim1 <- c("A1", "A2")
> dim2 <- c("B1", "B2", "B3")
> dim3 <- c("C1", "C2", "C3", "C4")
>
> a <- array(1:24, c(2,3,4), list(dim1, dim2, dim3))
>
> a
, , C1

   B1 B2 B3
A1  1  3  5
A2  2  4  6

, , C2

   B1 B2 B3
A1  7  9 11
A2  8 10 12

, , C3

   B1 B2 B3
A1 13 15 17
A2 14 16 18

, , C4

   B1 B2 B3
A1 19 21 23
A2 20 22 24

> a[,,3]
   B1 B2 B3
A1 13 15 17
A2 14 16 18
```

### 数据框（data frame）
数据框有些想数据库里的一个表，不同的列可以包含不同的模式。

```R
myData <- data.frame(col1, col2, col3, ...)

> userID <- c(1,2,3,4)
> age <- c(16,17,18,19)
> name <-c("Lucy", "Jack", "Tom", "Han")
> users <- data.frame(userID, age, name)
> users
  userID age name
1      1  16 Lucy
2      2  17 Jack
3      3  18  Tom
4      4  19  Han

> users[1,2]
[1] 16

> users[1:2]
  userID age
1      1  16
2      2  17
3      3  18
4      4  19

> users[c("name", "age")]
  name age
1 Lucy  16
2 Jack  17
3  Tom  18
4  Han  19

> users["age"]
  age
1  16
2  17
3  18
4  19

> users$age
[1] 16 17 18 19

> table(users$name, users$age)

       16 17 18 19
  Han   0  0  0  1
  Jack  0  1  0  0
  Lucy  1  0  0  0
  Tom   0  0  1  0

```

### 因子（factor
类别（名义型）和有序类别（有序型）变量在R中称为因子（factor）。

- 名义型变量是没有顺序之分的类别变量。
- 有序型变量表示一种顺序关系，而非变量关系。
- 连续型变量可以呈现某个范围内的任意值，并同时表示了顺序和数量。

用factor函数可以将向量变成因子。

```R
> types <- c("t1", "t2", "t1", "t4")
> types <- types(status)
> types
[1] t1 t2 t1 t4
Levels: t1 t2 t4

> status = c("poor", "improved", "excellent", "poor")
> status <- factor(status, order=TRUE)
> status
[1] poor      improved  excellent poor     
Levels: excellent < improved < poor

> users <- data.frame(name, age, status, types)
> users
  name age    status types
1 Lucy  16      poor    t1
2 Jack  17  improved    t2
3  Tom  18 excellent    t1
4  Han  19      poor    t4
> str(users)  # 显示对象的结果
'data.frame':	4 obs. of  4 variables:
 $ name  : Factor w/ 4 levels "Han","Jack","Lucy",..: 3 2 4 1
 $ age   : num  16 17 18 19
 $ status: Ord.factor w/ 3 levels "excellent"<"improved"<..: 3 2 1 3
 $ types : Factor w/ 3 levels "t1","t2","t4": 1 2 1 3
> summary(users)  # 显示对象的统计摘要
   name        age              status  types
 Han :1   Min.   :16.00   excellent:1   t1:2  
 Jack:1   1st Qu.:16.75   improved :1   t2:1  
 Lucy:1   Median :17.50   poor     :2   t4:1  
 Tom :1   Mean   :17.50                       
          3rd Qu.:18.25                       
          Max.   :19.00                       
>
```

### 列表
列表是一些对象的有序集合。

列表是R语言中最复杂的一种，也是重要的：

- 以一种简单的方式组织和重新调用不相干的信息。
- 许多R函数都是以列表返回结果。

用list创建。

```R
> title <- "A List"
> m <- matrix(1:20, 4, 5)

> myList <- list(title=title, age=age, m)
> myList
$title
[1] "A List"

$age
[1] 16 17 18 19

[[3]]
     [,1] [,2] [,3] [,4] [,5]
[1,]    1    5    9   13   17
[2,]    2    6   10   14   18
[3,]    3    7   11   15   19
[4,]    4    8   12   16   20

> myList[1]
$title
[1] "A List"

> myList[2]
$age
[1] 16 17 18 19
> myList[[2]]
[1] 16 17 18 19
```

### 数据输入

```R
# 键盘输入——edit()
> mydata <- data.frame(age=numeric(0), gender=character(0))
> mydata <- edit(mydata)

# csv文件
> grades <- read.table("studentgrades.csv", header=TRUE, sep=",", row.names="GRADE")

# Excel文件
install.packages("RODBC")
library(RODBC)
channel <- odbcConnectExcel("excel.xls")
mydata <- sqlFetch(channel, "sheet")
odbcClose(channel)

# 对于xlsx文件，可以read.xlsx
```


# 参考
- 《R语言实战》
