# 修改component 的高度
所有确定大小的内容，都是由父类对子类进行范围性约束，然后子类根据内容的大小进行显示


VC继承 scrollableCOmponent // component

Contentsize(target: CGsize) 

Target.with

$0.height = 200


SectionController 

Inset 修改 SectionController 和 collectionView之间的边距
Linespacing 修改 item之间间距


-----
｜				inset
 | 		_____________ 				______________					|
|		|			|				|			
	

		|	
|	inset	|			|.      linespacing	|			|		inset	|
|		_____________				_________________				|
|																	|
|																	|
|___________________________________________________________________|