# 切片
切片操作不会因为下标越界而抛出异常，而是简单地在列表尾部截断或者返回一个空列表

* python3支持切片操作的数据类型有list、tuple、string、unicode、range
* 切片返回的结果类型与原对象类型一致
* 切片不会改变原对象，而是重新生成了一个新的对象

切片的语法表达式为：[`start_index` : `end_index` : `step`]，其中：

* `start_index`表示起始索引，从0开始计算
* `end_index`表示结束位置 （截取的数据不包含结束位置）
* `step`表示步长，步长不能为0，默认值为1(当步长省略时，可以省略最后一个冒号)

```python
a = [0,1,2,3,4,5,6,7]

a[:3]表示从索引0开始取，直到索引3为止，但不包括索引3。即索引[0, 1, 2] 正好是3个元素。

a[1:3]表示 从索引1开始，取出下标为 1、2 两个元素出来：[1, 2]

a[-2:]表示 从倒数第2个元素开始，取出2个元素出来：[6, 7]

a[-2:-1]表示 从倒数第2个元素开始，取出1个元素出来：[6]

a[::2]表示 每隔2个取一个元素： [0, 2, 4, 6]

a[:5:2]表示 前5个每隔2个取一个元素： [0, 2, 4, 6]

a[::-1]表示 倒序取元素：[7, 6, 5, 4, 3, 2, 1, 0]
```