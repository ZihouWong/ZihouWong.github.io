IGListKit中的EmptyView在开发的初期可以直接穿入Nil返回

当开发后期的时候，需要补充缺省页

所有override的方法都需要调用父类的方法，

override func viewDidDisplay() {
	super.viewDidDisplay()
	// DO SOMETHING
}