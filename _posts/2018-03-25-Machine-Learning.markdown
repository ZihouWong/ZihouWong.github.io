---

title: "Machine Learning"
date: 2018-03-25
categories: 机器学习 

layout: post
comments: true

---

学习算法的任务：通过训练样本，选择或学习得到参数θ

* #### 监督学习(Supervised Learning)
* #### 非监督学习(Unsupervised Learning)

监督学习：通过已有训练样本进行训练，得到一个拟合效果最好的基本模型，再使用该模型，对新的训练样本计算出相应的输出结果，对输出结果进行判断实现分类的目的。并通过大量的迭代后，最后得到最终模型。

```
简单来说：
通过给算法提供一组标准答案，然后希望算法计算得到标准输入和标准输出之间的联系，然后返回更多的标准答案。
```

* #### 非监督学习(Unsupervised Learning)
非监督学习：事先没有任何训练样本，需要直接对数据进行建模

* #### 半监督学习(Semi-Supervised Learning, SSL)
半监督学习：是监督学习与无监督学习相结合的一种学习方法。半监督学习使用大量的未标记数据，以及同时使用标记数据，来进行模式识别工作。

---

# 回归函数：
在支持向量机的算法中，把一些具有数据的点，在无限维的空间中画出来，这样就可以把数据在在空间中更加直观的显示出来。 

---
* #### 梯度下降算法

* #### 随机（增量）梯度下降算法
通常用于样本数据量大的场景
最终收敛所得的参数值，会在局部最优点的参数 附近徘徊

对于数据量大的情况，随机梯度下降算法比简单梯度下降算法更高效

批梯度下降的算法过程：通过利用第一个样品数据，更新所有的参数，然后利用第二个样品数据，继续更新之前的所有参数


notation：``` 符号注释： ```
m表示训练样品的总数量 ``` m = training examples ```
x表示输入变量 ``` x = input variables (features) ```
y表示目标变量 ``` y = output variables(target) ```
(x, y)表示一个训练样品 ``` (x, y)  = one training example ```
第i个训练样品 (xi, yi) ``` i th training example = (xi, yi)```

![公式一](http://upload-images.jianshu.io/upload_images/11317235-041f73aac3649794..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
Daum Equation Editor:
h( x )= { h } _ { \theta } ( x )  = { \theta  }_{ 0 }+{ \theta  }_{ 1 }{ x }_{ 1 }+{ \theta  }_{ 0 }{ x }_{ 2 }+…
```

训练样本 通过 学习算法，生成假设函数（历史，习惯）。假设函数的作用是接受数据，然后输出计算后的结果，``` And Repeating till convergence ```，不断重复算法执行直到参数得到收敛，进而得到一个更通用的函数

* 对于梯度下降算法，不同的初始值，可能导致出现不同的局部最优解
---
θi 指的是学习算法中的实数参数 ``` θi are called parameter```
α 指的是学习速度 ``` α is a parameter of the algorithm called learning rate  ```

![公式二](http://upload-images.jianshu.io/upload_images/11317235-f85783d8856526dd..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
Daum Equation Editor:
{ \theta  }_{ i }={ \theta  }_{ i }-\alpha ({ h }_{ \theta  }-y)\cdot { x }_{ i }
```

```

how large step you take
你迈的步子有多大（选取最合适的值，过大过小都不一定为最好）
```

* 假设在一个碗状的模型中，只存在一个局部最优点，学习速度不断减小，当达到局部最优点时，梯度下降速度(学习速度α)会减少到0

---
### 收敛的两种判断方式
* 观察两次迭代中，两次迭代中常量的值是否变化很大，如果变化不大，可以认为其收敛
* 观察你希望收敛的值，若该值变化量不再增大，则可认为其收敛




