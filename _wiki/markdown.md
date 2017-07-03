---
layout: wiki
title: Markdown
categories: Markdown
description: Markdown 常用语法示例。
keywords: Markdown
---

**目录**

* TOC
{:toc}

### 超链接

```
[靠谱-ing](http://mazhuang.org)

<http://mazhuang.org>
```

[靠谱-ing](http://mazhuang.org)  

<http://mazhuang.org>

### 列表

```
1. 有序列表项 1

2. 有序列表项 2

3. 有序列表项 3
```

1. 有序列表项 1

2. 有序列表项 2

3. 有序列表项 3

```
* 无序列表项 1

* 无序列表项 2

* 无序列表项 3
```

* 无序列表项 1

* 无序列表项 2

* 无序列表项 3

- [x] 任务列表 1
- [ ] 任务列表 2

### 强调

```
~~删除线~~

**加黑**

*斜体*
```

~~删除线~~

**加黑**

*斜体*

### 标题

```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

Tips: `#` 与标题中间要加空格。

### 表格

```
| HEADER1 | HEADER2 | HEADER3 | HEADER4 |
| ------- | :------ | :-----: | ------: |
| content | content | content | content |
```

| HEADER1 | HEADER2 | HEADER3 | HEADER4 |
| ------- | :------ | :-----: | ------: |
| content | content | content | content |

1. :----- 表示左对齐
2. :----: 表示中对齐
3. -----: 表示右对齐

### 代码块

```python
print 'Hello, World!'
```

1. list item1

2. list item2

   ```python
   print 'hello'
   ```

### 图片

```
![本站favicon](/favicon.ico)
```

![本站favicon](/favicon.ico)

### 锚点

```
* [目录](#目录)
```

* [目录](#目录)

### Emoji

:camel:
:blush:
:smile:

### Footnotes

This is a text with footnote[^1].

[^1]: Here is the footnote 1 definition.


Test Markdown document
======================

Text
----

Here is a paragraph with bold text. **This is some bold text.** Here is a
paragraph with bold text. __This is also some bold text.__

Here is another one with italic text. *This is some italic text.* Here is
another one with italic text. _This is some italic text._

Here is another one with struckout text. ~~This is some struckout text.~~


Links
-----

Autolink: <http://example.com>

Link: [Example](http://example.com)

Reference style [link][1].

[1]: http://example.com  "Example"


Images
------

Image: ![My image](http://www.foo.bar/image.png)

Headers
-------

# First level title
## Second level title
### Third level title
#### Fourth level title
##### Fifth level title
###### Sixth level title

### Title with [link](http://localhost)
### Title with ![image](http://localhost)

Code
----

```
This
  is
    code
      fence
```

Inline `code span in a` paragraph.

This is a code block:

    /**
     * Sorts the specified array into ascending numerical order.
     *
     * <p>Implementation note: The sorting algorithm is a Dual-Pivot Quicksort
     * by Vladimir Yaroslavskiy, Jon Bentley, and Joshua Bloch. This algorithm
     * offers O(n log(n)) performance on many data sets that cause other
     * quicksorts to degrade to quadratic performance, and is typically
     * faster than traditional (one-pivot) Quicksort implementations.
     *
     * @param a the array to be sorted
     */
    public static void sort(byte[] a) {
        DualPivotQuicksort.sort(a);
    }

Quotes
------

> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.


> A list within a blockquote:
>
> *	asterisk 1
> *	asterisk 2
> *	asterisk 3


> Formatting within a blockquote:
>
> ### header
> Link: [Example](http://example.com)



Html
-------

This is inline <span>html</html>.
And this is an html block.

<table>
  <tr>
    <th>Column 1</th>
    <th>Column 2</th>
  </tr>
  <tr>
    <td>Row 1 Cell 1</td>
    <td>Row 1 Cell 2</td>
  </tr>
  <tr>
    <td>Row 2 Cell 1</td>
    <td>Row 2 Cell 2</td>
  </tr>
</table>

Horizontal rules
----------------

---

___


***


Lists
-----

Unordered list:

*	asterisk 1
*	asterisk 2
*	asterisk 3


Ordered list:

1.	First
2.	Second
3.	Third


Mixed:

1. First
2. Second:
	* Fee
	* Fie
	* Foe
3. Third


Tables:

| Header 1 | Header 2 |
| -------- | -------- |
| Data 1   | Data 2   |
