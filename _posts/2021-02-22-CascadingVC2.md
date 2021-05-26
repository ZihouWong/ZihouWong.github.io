# CascadingVC 里的 Pager 保留各个子 ViewController 的 contentOffset

## 前提: 

### 假设
为了方便后续的描述，下文使用以下假设：

把 `A` 假设为**开始滑动时**所在的 `Page`，把 `B` 假设为**滑动结束后**所在的 `Page`。

### 须知

1. 每个 `Page` 拥有自己独立的 `contentOffset`，可以通过 `Index` 值可以获取对应的 `Page`

2. `currentIndex` 记录的是当前屏幕中轴线所在的 `Page` 的 `Index`，currentIndex 是一个**随时**变化的变量。

3. `CascadingVC` 会根据 `currentIndex` 找到对应的 `Page`, 再根根据此 `Page` 的 `contentSize`，计算此 `Page` 的 `contentOffset`，并且经过一段过程修改到 `Page` 的 `contentOffset`。

## 简介

### 问题现象
	
在底部 `CombinedListVC` 中，当从 `A` 滑动 `B` 的时候，在屏幕中轴线压在 `B` 上的瞬间，`B` 的显示内容发生了瞬间移动，其中 `B` 显示内容的偏移量与 `A` 显示内容的偏移量相同。
	
情景: 	
如果当用户正在 `A` 查看第七条内容，若此时用户切换到 `B` ，合理的逻辑是 `B` 应该显示的是第一条内容，但是未修改前的应用会直接显示第七条内容。

这样的现象会在一定程度上影响用户的使用体验。

## 问题出现的原因

#### 简略：
	
由于 `currentIndex` 是随时发生变化的，在 `A` 切换到 `B`的瞬间，也就是在屏幕中轴线压在 `B` 上的瞬间，
`currentIndex` 发生了变化，这时候会影响到 `currentIndex` 对应的 `Page` 的 `contentOffset` 赋值的这一个过程。导致 `B` 的 `contentOffset` 是通过 `A` 的 `contentSize` 计算得出的。

1. 在切换的 `Page` 过程中，`B` 的 `contentOffset` 更新时使用了 `A` 的 `contentOffset`	
2. 在不同的 `Page` 中所使用的是同一个 `contentOffset`

#### 详细

    
    ``` swift
    extension CascadingVC {
        open func updateComponents() {
            ...
        }
        ...
    }
    ```
	
		
	> `currentIndex` 的实现是跟随屏幕中心所在的 Page 来确定值的, 也就是说当从 A 滑动到 B，在屏幕中心点刚转换到 B 的时候，此时的 currentIndex 已经发生变化，切换到 B 的 Index。
	
	> 当 `component` 的 `contentOffset` 更新之后，会根据 `currentIndex` 的值，更新对应的 `Page` 的`ContentOffset`。


    ``` swift
    //  CascadingVC.swift
    extension CascadingVC {
            ...
            for component in availableCascadableComponentsInOrder {
            ...
                guard let scrollableComponent = component as? ScrollableCascadableComponentType else {
                ...
            }
            ...
            if scrollableComponent.contentOffset.y != newContentOffetY {
                scrollableComponent.contentOffset.y = newContentOffetY
            }
        }
    }    
    ```
    
    ``` swift
    //  CascadableComponent+XLPagerTabStrip.swift
    public extension CascadableComponentType where Self: PagerTabStripViewController {
        var currentComponent: CascadableComponentType? {
            viewControllers(for: self)[safe: currentIndex] as? CascadableComponentType
        }
    }
    
    public extension ScrollableCascadableComponentType where Self: PagerTabStripViewController {
        var currentScrollableComponent: ScrollableCascadableComponentType? {
            currentComponent as? ScrollableCascadableComponentType
        }
    }
    
    public extension ScrollableCascadableComponentType where Self: ButtonBarPagerTabStripViewController {
        ...
        var contentOffset: CGPoint {
            get { currentScrollableComponent?.contentOffset ?? .zero }
            set { currentScrollableComponent?.contentOffset = newValue }
        }
    }
    ```
 
## 解决的思路

1. 首先是关于 `contentOffset` 的实现。`Page` 的 `contentOffset` 是根据 `currentComponentIndex` 进行修改的

    ``` swift 
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    public var currentComponent: CascadableComponentType? {
        viewControllers(for: self)[safe: currentComponentIndex] as? CascadableComponentType
    }
    ```
    
    ``` swift 
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    public var currentScrollableComponent: ScrollableCascadableComponentType? {
    currentComponent as? ScrollableCascadableComponentType
    }
    ```
    
    ``` swift 
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    public var contentOffset: CGPoint {
        get { currentScrollableComponent?.contentOffset ?? .zero }
        set { currentScrollableComponent?.contentOffset = newValue }
    }
    ```
 
2. 而 `currentComponentIndex` 的更新是在 `Page` 切换完成之后进行。

	切换 `Page` 由两种操作完成：

	-  滑动页面切换
	
	``` swift
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    open func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        updateAndRearrangeNextPage()
    }
	```
	- 点击按钮切换

    ``` swift 
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    override open func scrollViewDidEndScrollingAnimation(_ scrollView: UIScrollView) {
        super.scrollViewDidEndScrollingAnimation(scrollView)
        updateAndRearrangeNextPage()
    }
    ``` 

    ``` swift
    // ButtonBarPagerTabStripVCWithComponentSupport.swift
    private func updateAndRearrangeNextPage() {
        // 更新 currentComponentIndex
        currentComponentIndex = currentIndex
        ...
    }
    ```
    
2. 在 `currentComponentIndex` 更新完成之后，需要更新 `CascadeScrollView` 的 `contentOffset`。


	``` swift
	// CascadingVC.swift
	    let containerVisibleArea = cascadeScrollView.bounds
	        ...
	}
	```

	``` swift
	// ButtonBarPagerTabStripVCWithComponentSupport.swift
    private func updateAndRearrangeNextPage() {
        ...
        guard let compnent = currentScrollableComponent else { return }
        updateCascadeScrollView(ToComponent: compnent)
    }
	```
	    
3. 更新 CascadeScrollView，需要判断 A 的 `CascadingVC` 中的可视的位置是否显示都是 `ScrollableCascadableComponent` 的内容。

	``` swift
	//  ButtonBarPagerTabStripVCWithComponentSupport.swift
	    private func updateCascadeScrollView(ToComponent component: ScrollableCascadableComponentType) {
	
	    ...
	    let isFullyShowingPage = cascadingVC.isFullyShowingComponent(component)
	
	    ...
	    if isFullyShowingPage {
		    let contentOffsetY = min(max(currentComponent.contentOffset.y, 0), component.contentSize(fitting: .zero).height)
		    cascadingVC.scrollToComponent(component: self, at: contentOffsetY)
	    } else {
	    }
	}
	```
	
	``` swift
	// CascadingVC.swift
	public func isFullyShowingComponent(_ component: ScrollableCascadableComponentType) -> Bool {
	
		let visiableAreaRect = component.contentCascadingView.rect(inUpperView: view)
		let scrollableComponentAreaRect = cascadeScrollView.frame.intersection(visiableAreaRect ?? .zero)
		
		// 通过使用 rounded() 用于解决精度问题
		let roundedVisiableAreaRectHeight = visiableAreaRect?.height.rounded()
		let roundedScrollableComponentAreaRectHeight = scrollableComponentAreaRect.height.rounded()
		
		return roundedVisiableAreaRectHeight == roundedScrollableComponentAreaRectHeight
	}
	```
    


    ``` swift 
    // CascadingVC.swift
	public func scrollToComponent(component: ScrollableCascadableComponentType, at contentOffsetY: CGFloat) {
	
		let _contentOffset = getCascadeScrollViewContentOffsetY(component: component, scrollableComponentOwnContentOffsetY: contentOffsetY)
		cascadeScrollView.contentOffset = CGPoint(x: 0, y: _contentOffset)
    }    
    ``` 
    
    ``` swift 
    // CascadingVC.swift
    private func getCascadeScrollViewContentOffsetY(component: ScrollableCascadableComponentType, scrollableComponentOwnContentOffsetY: CGFloat) -> CGFloat {

	    let componentMinY = frameForComponent(component)?.minY ?? .zero
	
	    return componentMinY + scrollableComponentOwnContentOffsetY
}
```    

## 结果

每一页的 `Page` 对应的 `contentOffset` 都各自独立。
	
如果 `CascadingVC` 不只显示 `ScrollableCascadableComponent` 的内容的时候，则会将当前 `component` 的 `contentOffset` 设置为0

## 后续问题

1. 由于新一页的 `Page` 是需要滑动结束之后，才能更新，但是从用户体验的角度来说，应该是一开始滑动的时候就进行更新。

2. 多个 `CascadingVC` 嵌套的问题