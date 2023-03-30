# Python基础语法

## 列表

```python
color = ['red', 'yellow', 'green']
print(color)
##运行结果['red', 'yellow', 'green']
```

### 访问列表元素

访问方式类似于Java的数组。

```python
color = ['red', 'yellow', 'green']
print(color[0]) ## red
print(color[-1]) ## green
```

### 增加列表元素

```python
## color = ['red', 'yellow', 'green']
color.append('blue')
## color = ['red', 'yellow', 'green', 'blue']
color.insert(0, 'origin')
## color = ['origin', 'red', 'yellow', 'green', 'blue']
```

### 删除列表元素

```python
## color = ['red', 'yellow', 'green']
## 1.del
del color[0]
## color = ['yellow', 'green']

## 2.pop
color.pop() ## 可获得弹出值
## color = ['red', 'yellow']

## 3.pop指定位置
color.pop(0) ## 可获得弹出值
## color = ['yellow', 'green']

## 4.remove
color.remove('red')
## color = ['yellow', 'green']
```



### 修改列表元素

```python
## 直接对已有元素赋值
## color = ['red', 'yellow', 'green']
color[0] = 'blue'
## color = ['blue', 'yellow', 'green']
```

### 排序列表元素

```python
## color = ['red', 'yellow', 'green']
## 1.sort
color.sort()
## color = ['green', 'red', 'yellow']

## 2.sort(reverse=True)
color.sort(reverse=True)
## color = ['yellow', 'red', 'green']

## 3.sorted ## 原列表不变
print(sorted(color))
## color = ['green', 'red', 'yellow']

## 4.reverse
color.reverse()
## color = ['green', 'yellow', 'red']
```

### 获得列表长度

```python
## color = ['red', 'yellow', 'green']
len(color)
## 3
```

### 切片

```python
## color = ['red', 'yellow', 'green']
color[1:2]
color[0:1]
color[:]
```



## 数组列表

## 元组

```python

```

