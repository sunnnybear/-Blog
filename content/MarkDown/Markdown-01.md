
## Markdown 基本语法

### 一、标题

```
#标题 1
##标题 2
###标题 3
####标题 4
#####标题 5
```

#标题 1
##标题 2
###标题 3
####标题 4
#####标题 5

### 二、粗斜体

```
*斜体*   _斜体_
**粗体**   __粗体__
***粗斜体***   ___粗斜体___
```

*斜体*   _斜体_  
**粗体**   __粗体__  
***粗斜体***   ___粗斜体___

### 三、代码块
    只需要段落前加四个空格；
    或者
    ```
    内容
    ```

### 四、链接 & 图片

```
文字链接 [文字](http:xxxxx)
显示图片 ![图片描述](http:xxxxx)
```

### 五、列表

**有序列表**

```
1. 内容 [数字+'.'+空格]
2. 内容 [数字+'.'+空格] 
3. 内容 [数字+'.'+空格]
```

1. 内容
2. 内容
3. 内容

**无序列表**

```
- 内容 [减号+空格]
+ 内容 [加号+空格]
* 内容 [星号+空格]
```

- 内容 [减号+空格]
+ 内容 [加号+空格]
* 内容 [星号+空格]

**列表嵌套**

```
1. 列出所有元素：
    - 无序列表元素 A
        1. 元素 A 的有序子列表
    - 前面加四个空格
2. 列表里的多段换行：
    前面必须加四个空格，
    这样换行，整体的格式不会乱
3. 列表里引用：

    > 前面空一行
    > 仍然需要在 >  前面加四个空格
4. 列表里代码段：
    ```
    前面四个空格，之后按代码语法 ``` 书写
    ```
        或者直接空八个，引入代码块
```

1. 有序列表 内容
    * 无序列表 内容
    * 无序列表 内容  
        1. 有序列表 内容 
        2. 有序列表内容
2. 列表里多段换行：  
    前边必须加4个空格  
    这样格式不会乱
3. 列表里引用： 

    >前边空一行  
    >\> 前加4个空格

4. 列表里代码段：

    
        每行加4个空格，其它一样；
        或者直接加8个空格

### 六、引用

**普通引用**

    > 内容 [大于号 + 空格]

> 内容  

**引用里嵌套引用**

    > 最外层引用
    > > 第二层引用
    > > > 第三层引用

> 最外层引用  
> > 第二层引用  
> > > 第三层引用

**引用里嵌套列表**

    > 1. 第一条
    > 2. 第二条

> 1. 第一条  
> 2. 第二条

### 七、换行

    换行如果想换行，在行结尾加两个空格[空格][空格]  
    就像这样；

换行如果想换行，在行结尾加两个空格[空格][空格]  
就像这样

### 八、转义

    特殊符号前，加 \ 符号

* 我是转义前  

\* 我是转义后

>我是转义前

\>我是转义后

### 九、标记

    用 ` `符号
    在这句话中，`我是标记`，`我也是标记`。

在这句话中，`我是标记`，`我也是标记`。

### 十、表格

    | name  | age   | class  |
    | :---- | :--- :| -----: | 
    |bob    |12     |  A-4   |

    :--  左对齐
    :--: 居中
    --:  右对齐

| name  | age   | class  |
| :---- | :--- :| -----: |
|bob    |12     |  A-4   |

### 十一、删除线

    大家好，~我是被删除的内容~

大家好，~~我是被删除的内容~~










