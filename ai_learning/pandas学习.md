### pandas

#### 常用数据类型
1. series 一维，带标签数组
2. DataFrame 二维，Series容器
   

#### series创建、切片、索引

![Alt text](image-23.png)
![Alt text](image-24.png)
列表 对象range 字典，可以创建Series

![Alt text](image-25.png)
![Alt text](image-26.png)

#### pandas读取外部数据
![Alt text](image-27.png)

#### dataframe
![Alt text](image-28.png)
![Alt text](image-29.png)
![Alt text](image-30.png)

```python
df.sort_values(by="Count_AnimalName",ascending=False)
# 排序
```

#### pandas取行取列
![Alt text](image-31.png)
- 方括号写数组，表示取行，对行进行操作
- 写字符串，表示的取列索引，对列进行操作

1. df.loc通过**标签**索引行数据
2. df.iloc通过**位置**获取行数据
![Alt text](image-32.png)
![Alt text](image-33.png)

#### 布尔索引
![Alt text](image-34.png)