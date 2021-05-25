# CascadingVC 里的 Pager 保留各个子 ViewController 的 contentOffset

### 假设

为了方便后续的描述，下文统一使用以下假设：

把 A 假设为**开始滑动时**所在的 Page，

把 B 假设为**滑动结束时**所在的 Page。

### 简介

- 问题
	
	现在项目里存在一个问题：在不同的 `Page` 中所使用的是同一个 `contentOffset
	
	
	**在切换的 `Page` 过程中，B 会得到 A 的 `contentOffset`, 导致有类似于使用同一个 `contentOffset` 的假象。**

### 问题出现的原因

1. CascadingVC 在水平方向上滑动切换到下一个 `Page` 的瞬间会触发 `updateComponent()`， 其中调用了`layoutScrollView(...)` 函数。
    
    ``` swift
    extension CascadingVC {
        open func updateComponents() {
            ...
            layoutScrollView(includingComponentsNotInVisualRange: true)
        }
        ...
    }
    ```

	> `layoutScrollView(...)`中涉及了 `component` 的 `cotentOffset` 的更新，
		
	> 当 `component` 的 `contentOffset` 更新之后，会根据 `currentIndex` 的值，更新对应的 `Page` 的`ContentOffset`。

	
2. 当从 A 滑动到 B 时，由于 `layoutScrollView(...) 和  Page 的 ContentOffset (set方法)`  **不能** 保证其是一个原子操作，因此就造成了在 `这一系列操作` 执行的过程中， 本该保持不变的变量 `currrentIndex` 发生了变化。也就导致了根据 A 的 `contentSize` 计算的结果 `contentOffset` 值，经过一系列操作，最终赋值到了 B。 

    ``` swift
    //  CascadingVC.swift
    extension CascadingVC {
        open func layoutScrollView(includingComponentsNotInVisualRange: Bool) {
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
 
### 解决的思路

1. 在滑动开始时，记录当前 `Page` 的 `currentIndex` 为 `modifyIndex`。

    ``` swift 
    //  PagerTabStripViewController.swift
    open class PagerTabStripViewController: UIViewController, UIScrollViewDelegate {
        ...
        open var modifyIndex: Int = 0
    }
    ```
    
    ``` swift
    // ResearchHomeVC+CombinedList.swift
    override func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {
        modifyIndex = currentIndex
    }
    ```
  
2. 滑动过程中， 触发 `layoutScrollView()` ，计算各个 Component 的 `frame` 和 `contentOffset`， 将计算之后的结果根据 `modifyIndex` 返回。

    
    ``` swift 
    // CascadableComponent+XLPagerTabStrip.swift
    public extension CascadableComponentType where Self: PagerTabStripViewController {
        ...
        var currentModifyComponent: CascadableComponentType? {
            return viewControllers(for: self)[safe: modifyIndex] as? CascadableComponentType
        }
    }
    ```
    ``` swift
    // CascadableComponent+XLPagerTabStrip.swift
    public extension ScrollableCascadableComponentType where Self: PagerTabStripViewController {
        ...
        var currentModifyScrollableComponent: ScrollableCascadableComponentType? {
            currentModifyComponent as? ScrollableCascadableComponentType
        }
    }
    ```
    
    ``` swift
    public extension ScrollableCascadableComponentType where Self: ButtonBarPagerTabStripViewController {
        ...
        var contentOffset: CGPoint {
            get { currentModifyScrollableComponent?.contentOffset ?? .zero }
            set { currentModifyScrollableComponent?.contentOffset = newValue }
        }
    }
    ```
  
    
3. 滑动结束之后，首先判断 A 的 `CombinedListVC` 是否正在完全显示，记录这个 Bool 结果，然后再更新 `modifyIndex`。


    ``` swift 
    // ResearchHomeVC+CombinedList.swift
    
    class CombinedListVC: PageControlButtonBarVC {
        func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
            
            let isFullyShowingPage: Bool = {
                return contentOffset.y > 0
            }()
            
            modifyIndex = currentIndex
            
            let targetOffset = isFullyShowingPage ? contentOffset.y : 0
            containerCascadingVC?.scrollTo(self, contentOffset: targetOffset, isFullyShowingPage: isFullyShowingPage)
        }
    }
    ```
    
4. 根据这个是否完全显示的Bool值，如果未完全显示，则需要移动 `CombinedListVC` 中的每一个 `Page` 的 `contentOffset` 设置为0（置顶）；如果完全显示，则需要移动`cascadeScrollView` 的 `bounds` ，移动至 B 的 `contentOffset` 的位置。


    ``` swift 
    open func scrollTo(_ component: ScrollableCascadableComponentType, contentOffset: CGFloat, isFullyShowingPage: Bool) {
        // 修改 cascadeScrollView 的位置
        var currentVisiableAreaContentOffsetY = isFullyShowingPage ? frameForComponent(component)?.minY : self.contentOffset.y
        
        var _contentOffsetY = isFullyShowingPage ? ((currentVisiableAreaContentOffsetY ?? .zero) + contentOffset) : self.contentOffset.y
        cascadeScrollView.bounds = CGRect(origin: CGPoint(x: CGFloat(0), y: _contentOffsetY), size: cascadeScrollView.bounds.size)
        layoutScrollView(includingComponentsNotInVisualRange: true)
    }
    ``` 
    
    
## 还未解决的问题

1. 通过按钮切换页面的时候currentIndex 不能正确。