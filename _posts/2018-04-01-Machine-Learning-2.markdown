---

title: "Machine-Learning-2"
date: 2018-04-01
categories: 机器学习

layout: post
comments: true

---

本章笔记涉及的涉及的知识

线性回归 linear regression 

局部加权回归 locally weighted regression loess/lowess

分类算法 classification algorithm

回归 logistic regression logistic

牛顿方法 newton's method



## 参数学习算法
定义：有固定数目的参数以用来进行数据拟合的算法


特征选择算法

***

### 欠拟合 和 过拟合 的概念

#### 适当的拟合模型

![适当的拟合模型]({{site.baseurl}}/img/2-1.jpg)

#### 拟合模型过于简单 ----欠拟合
![拟合模型过于简单]({{site.baseurl}}/img/2-2.jpg)


#### 拟合模型过于复杂 ----过拟合
![拟合模型过于复杂]({{site.baseurl}}/img/2-3.jpg)


## 非参数学习算法
定义：算法的维持是基于整个训练集合的，即使是在学习以后。



非参数学习算法可以缓解你们对选择特征的需求

that will help to alleviate the need somewhat for you to choose features very carefully.

the number of parameters grows with M

M is the size of the training set

算法所需的参数随着样品数量的增加而改变

M 是样品总数量

中心极限定律 

