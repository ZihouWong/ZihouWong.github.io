#

程序 进行状态记录

首先是： 

父view fundDetail 在初始化的时候，实例化了 highlightsVC 和 risksVC

当 ViewDidLoad 的时候， 进入 loadData() 

loadData() 再进入 loadBaseInfo()

loadBaseInfo() 中会进入 loadExtraData()

loadExtraData() 进入 loadEvaluation()

loadEvaluation 会调用预先准备的 API  fundApi.fundEvaluation(innerCode: innerCode)

然后获取到 JSON 数据，

在 API 准备的时候，需要准备获取的数据类型，这个数据类型需要根据返回值来进行决定

#### API准备 - 返回值的数据类型

这次的返回值的结果是 一个数据多个数组的数据，

每一个数组 都是一个单独的数据类型，所以需要根据每一个数组的数据类型，实现一个独立的数据类型，

因为返回的是多个数组

所以 FundEvaluation 中，需要实例多个数据数组

此处复习一下 enum 的语法

``` swift
public enum RiskType: String, Codable {
	case highlights
	case risks
}
```

以及路径

当获取到数据之后，回进入 onSuccess 的 回调中，回调中会 调用VC中提前准备好的updateModel() 

updateModel() 中包括了不不同类型的数据的更新（从业，稳定性，年化收益，表现，估值，集中程度）

#### updateModel() 的实现 

> 这个方法 主要实现了 更新 VC 中的 keyValues 

首先是 创建一个空的元组数组 去暂存这些数据 

元组数组类型是 `[(NSAttributedString, NSAttributedString)]`

通过处理原始数据 返回一个元组元素，然后向元组数组末尾添加一个新的元素

最后将 这个元组数组 替换掉 self.元组数组

元组数据 使用监视器 didSet 当数据发生更改的时候，

会将 keyValue 中的新的元素，创建一个新的keyValueView，添加进入 keyValueView的队列中

2021-03-08 11:18 待续...










