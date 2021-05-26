	// cascadingVC 主要实现滚动的是 caasdasda

```
import BaseExtensions
import UIKit

open class CascadingVC: UIViewController {

    public let cascadeScrollView = CascadeScrollView()

    private var componentFrameCache = [ObjectIdentifier: CGRect]()
    private var componentVisiblityObservation = [ObjectIdentifier: NSKeyValueObservation]()
    private var componentContainerViews = [ObjectIdentifier: UIView]()

    private(set) var needsUpdateComponents: Bool = true

    open var cascadableComponentsInOrder: [CascadableComponentType] = [] {
        didSet {
            // MARK: 过滤出 oldValue 有而 newValue 没有的元素

            let componentDeletes = oldValue.filter { oldItem in !cascadableComponentsInOrder.contains(where: { $0 === oldItem }) }

            for component in componentDeletes {
                let id = identifierForComponent(component)

                if let vc = component as? UIViewController, vc.parent == self {
                    vc.willMove(toParent: nil)
                }

                componentFrameCache.removeValue(forKey: id)
                componentVisiblityObservation.removeValue(forKey: id)?.invalidate()
                componentContainerViews.removeValue(forKey: id)?.removeFromSuperview()

                if let vc = component as? UIViewController {
                    vc.removeFromParent()
                }
            }

            let hasComponentsChanged = !cascadableComponentsInOrder.elementsEqual(oldValue, by: ===)

            if hasComponentsChanged {
                setNeedsUpdateComponents()
            }
        }
    }

    public convenience init(componentsInOrder: [CascadableComponentType] = []) {
        self.init(nibName: nil, bundle: nil)
        cascadableComponentsInOrder = componentsInOrder
    }

    deinit {
        contentSizeObservation?.invalidate()

        componentVisiblityObservation.values.forEach {
            $0.invalidate()
        }

        cascadableComponentsInOrder
            .compactMap { $0 as? ScrollableCascadableComponentType & ScrollViewOwner }
            .forEach { $0.contentSizeObservation?.invalidate() }
    }
}

// MARK: - [ViewController] Life Cycle

extension CascadingVC {
    override open func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .cFFFFFF

        cascadeScrollView
            .config {
                $0.multicastDelegate.addDelegate(self)
            }
            .adhere(toSuperview: view)
    }

    override open func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        cascadeScrollView.updateFrameIfNeeded(view.bounds)
        updateComponentsIfNeeded()
    }

    @available(iOS 11, *)
    override open func viewSafeAreaInsetsDidChange() {
        super.viewSafeAreaInsetsDidChange()
        setNeedsUpdateComponents()
    }
}

// MARK: - [ViewController] Cascading Implementation

extension CascadingVC {
    open func updateComponents() {
        needsUpdateComponents = false
        loadComponentsIfNeeded()
        updateComponentsHiddenStatus()
        calculateViewVisibleArea()
    }

    open func updateComponentsIfNeeded() {
        guard needsUpdateComponents else { return }
        updateComponents()
    }

    open func setNeedsUpdateComponents() {
        needsUpdateComponents = true
        view.setNeedsLayout()
    }

    private var availableCascadableComponentsInOrder: [CascadableComponentType] {
        cascadableComponentsInOrder
    }

    /// 将 cascadeScrollView 移动到该 component 的 contentOffsetY 这个位置
    /// - Parameters:
    ///   - component: 目标 component
    ///   - contentOffsetY: 目标 component 其自身的 contentOffset
    public func scrollToComponent(component: ScrollableCascadableComponentType, at contentOffsetY: CGFloat) {
        assert(cascadableComponentsInOrder.contains { $0 === component })
        assert(containerCascadingVC == nil)
        let _contentOffset = cascadeScrollViewContentOffsetY(component: component, scrollableComponentOwnContentOffsetY: contentOffsetY)
        cascadeScrollView.contentOffset = CGPoint(x: 0, y: _contentOffset)
    }

    /// 计算 cascadeScrollView 中移动到 component 位置所需要的 contentOffsetY
    /// - Parameters:
    ///   - component: 目标 component
    ///   - scrollableComponentOwnContentOffsetY: 目标 component 其自身的 contentOffset
    /// - Returns: CascadeScrollView 需要移动的 contentOffsetY
    private func cascadeScrollViewContentOffsetY(component: ScrollableCascadableComponentType, scrollableComponentOwnContentOffsetY: CGFloat) -> CGFloat {
        assert(cascadableComponentsInOrder.contains { $0 === component })

        let componentMinY = frameForComponent(component)?.minY ?? .zero

        return componentMinY + scrollableComponentOwnContentOffsetY
    }

    /// 判断目标 component 是否完全显示在 CascadingVC.cascadeScrollView 中
    /// - Parameter component: 目标 component
    /// - Returns: component 是否完全显示
    public func isFullyShowingComponent(_ component: ScrollableCascadableComponentType) -> Bool {
        assert(cascadableComponentsInOrder.contains { $0 === component })

        // - visiableAreaRect 是 CascadingVC.cascadeScrollView 的可视范围
        // - scrollableComponentAreaRect 是 component 与 visiableAreaRect 的交集
        let visiableAreaRect = component.contentCascadingView.rect(inUpperView: view)
        let scrollableComponentAreaRect = cascadeScrollView.frame.intersection(visiableAreaRect ?? .zero)

        // 通过使用 rounded() 用于解决精度问题
        let roundedVisiableAreaRectHeight = visiableAreaRect?.height.rounded()
        let roundedScrollableComponentAreaRectHeight = scrollableComponentAreaRect.height.rounded()

        return roundedVisiableAreaRectHeight == roundedScrollableComponentAreaRectHeight
    }

        let containerVisibleArea = cascadeScrollView.bounds

        for component in availableCascadableComponentsInOrder {
            let id = identifierForComponent(component)
            let componentContainerView = containerViewForComponent(id: id)
            let contentArea = componentFrameCache[id] ?? .zero

            guard componentContainerView.isVisible else { continue }

            guard includingComponentsNotInVisualRange || contentArea.intersects(containerVisibleArea) else {
                continue
            }

            guard let scrollableComponent = component as? ScrollableCascadableComponentType else {
                componentContainerView.updateFrameIfNeeded(contentArea)
                component.contentCascadingView.updateFrameIfNeeded(componentContainerView.bounds)
                continue
            }

            let isLastComponent = component === availableCascadableComponentsInOrder.last

            // MARK: Frame

            let y = cascadeScrollView.contentOffset.y
            let minY = contentArea.minY
            let _maxY = isLastComponent ? y : contentArea.maxY - containerVisibleArea.height
            let maxY = max(minY, _maxY)

            let newFrame = CGRect(
                x: 0,
                y: y.constrain(to: minY...maxY),
                width: containerVisibleArea.width,
                height: min(containerVisibleArea.height, contentArea.height)
            )

            componentContainerView.updateFrameIfNeeded(newFrame)
            component.contentCascadingView.updateFrameIfNeeded(componentContainerView.bounds)

            // MARK: Offset

            let contentOffsetY = y - contentArea.minY
            let minContentOffsetY = 0 as CGFloat
            let _maxContentOffsetY = isLastComponent
                ? containerVisibleArea.minY - contentArea.minY
                : contentArea.height - containerVisibleArea.height
            let maxContentOffsetY = max(minContentOffsetY, _maxContentOffsetY)
            let validContentOffsetRange = minContentOffsetY...maxContentOffsetY

            let newContentOffetY = contentOffsetY.constrain(to: validContentOffsetRange)
            if scrollableComponent.contentOffset.y != newContentOffetY {
                scrollableComponent.contentOffset.y = newContentOffetY
            }
        }
    }

    private func updateComponentsHiddenStatus() {
        for component in cascadableComponentsInOrder {
            let id = identifierForComponent(component)
            let containerView = containerViewForComponent(id: id)
            containerView.isHidden = component.contentCascadingView.isHidden || !component.shouldCascade
        }
    }

    private func loadComponentsIfNeeded() {
        for component in availableCascadableComponentsInOrder {
            let id = identifierForComponent(component)
            let componentContainerView = containerViewForComponent(id: id)

            if let vc = component as? UIViewController, vc.parent != self {
                // 先 addChild，以便让 vc 在 viewDidLoad 里能够访问到 parent
                addChild(vc)
            }

            guard component.contentCascadingView.superview != componentContainerView else { continue }

            #warning("弃用")
            let visibilityObservation = component.contentCascadingView.observe(\.isHidden, options: [.old, .new]) {
                [weak self] _, change in
                guard !change.oldValueEqualToNewValue else { return }
                self?.setNeedsUpdateComponents()
            }

            componentVisiblityObservation[id] = visibilityObservation

            component.setObserver { [weak self] in
                self?.setNeedsUpdateComponents()
            }

            componentContainerView.addSubview(component.contentCascadingView)
            componentContainerView.isHidden = component.contentCascadingView.isHidden
            cascadeScrollView.addSubview(componentContainerView)

            // UIViewController 加载 view 时会设置 autoresizingMask，
            // 导致一些意料之外的布局行为，所以需要先清除掉。
            component.contentCascadingView.autoresizingMask = []

            if let vc = component as? UIViewController {
                vc.didMove(toParent: self)
            }
        }
    }

    private func calculateViewVisibleArea() {
        componentFrameCache.removeAll(keepingCapacity: true)

        var contentHeight: CGFloat = 0

        for component in availableCascadableComponentsInOrder {
            let id = identifierForComponent(component)
            let height: CGFloat = {
                var height = cascadeScrollView.bounds.height
                if let scrollableComponent = component as? ScrollableCascadableComponentType {
                    height -= scrollableComponent.contentInset.vertical
                }
                if component === availableCascadableComponentsInOrder.last {
                    height -= view.safeAreaInsetsIfPresent.bottom
                }
                return height
            }()
            let size = CGSize(width: cascadeScrollView.bounds.width, height: height)
            let fittingSize = CGSize(
                width: cascadeScrollView.bounds.width,
                height: component.contentSize(fitting: size).height
            )
            var visibleFrame = CGRect(
                origin: CGPoint(x: 0, y: contentHeight),
                size: fittingSize
            )

            if let scrollableComponent = component as? ScrollableCascadableComponentType {
                visibleFrame.size.height += scrollableComponent.contentInset.vertical
            }

            componentFrameCache[id] = visibleFrame

            if containerViewForComponent(id: id).isVisible {
                contentHeight += visibleFrame.height
            }
        }

        let newContentSize = CGSize(
            width: view.bounds.size.width,
            height: contentHeight
        )

        if cascadeScrollView.contentSize != newContentSize {
            cascadeScrollView.contentSize = newContentSize
        }
    }
}

public extension CascadingVC {
    func frameForComponent(_ component: CascadableComponentType) -> CGRect? {
        componentFrameCache[identifierForComponent(component)]
    }

    private func containerViewForComponent(id: ObjectIdentifier) -> UIView {
        if let componentContainerView = componentContainerViews[id] {
            return componentContainerView
        }

        let componentContainerView = UIView()
        componentContainerViews[id] = componentContainerView
        return componentContainerView
    }

    private func identifierForComponent(_ component: CascadableComponentType) -> ObjectIdentifier {
        ObjectIdentifier(component)
    }
}

extension CascadingVC: UIScrollViewDelegate {
    open func scrollViewDidScroll(_ scrollView: UIScrollView) {
    }

    open func scrollViewWillBeginDragging(_ scrollView: UIScrollView) {}
    open func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {}

    open func scrollViewWillBeginDecelerating(_ scrollView: UIScrollView) {}
    open func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {}

    open func scrollViewDidEndScrollingAnimation(_ scrollView: UIScrollView) {}
}

extension CascadingVC: ScrollableCascadableComponentType, ScrollViewOwner {

    public var ownedScrollView: UIScrollView { cascadeScrollView }

    public var shouldCascade: Bool { cascadableComponentsInOrder.contains(where: \.shouldCascade) }
}
```