2021-02-09 工作记录


open func scrollTo(_ component: ScrollableCascadableComponentType, contentOffset: CGFloat, includingComponentsNotInVisualRange: Bool) {
//        calculateViewVisibleArea()
    
    // 修改 cascadeScrollView 的位置
    var combinedListVCOffset = frameForComponent(component)!.minY + contentOffset
    
    cascadeScrollView.bounds = CGRect(origin: CGPoint(x: CGFloat(0), y: combinedListVCOffset), size: cascadeScrollView.bounds.size)
}

遇到了contentOffset = 0 的判断边界的问题

由于根据combiendListVC的contentOffset 判断 combinedListVC是否完全显示的问题，combiendListVC是由于常为0，当切换新的Page时，若再此切换page就会出现问题，
// 不通过 combinedListVC.contentOffset.y > 0 (combinedListVC的是否完全显示的判断)

所以问题解决的办法。


C02V859EHV2L



#按键切换的思路

按键切换是由于点击事件，

