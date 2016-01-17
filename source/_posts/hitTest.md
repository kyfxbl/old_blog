title: hitTest
date: 2015-12-27 15:48:08
categories: iOS
---
hitTest方法，简而言之，就是给定一个点，返回一个view或layer，判定当前是哪一个view或layer被点中了
<!--more-->

# 原理

当用户触摸屏幕的时候，系统会依次调用view层次中各个子view的hitTest方法，来判断当前是哪个view被点中，决定谁是first responder。关于这点，这篇文章总结得不错：[iOS事件分发机制（一） hit-Testing](http://suenblog.duapp.com/blog/100031/iOS%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6%EF%BC%88%E4%B8%80%EF%BC%89%20hit-Testing)

默认的实现就是触摸点是否在此view的范围内，开发者可以重写此方法，来实现不同的逻辑。具体来说有2种常见的场景

# 重写UIView的hitTest方法

比如为了扩大某个view的点击区域，就可以用这个方法
```
override func hitTest(point: CGPoint, withEvent event: UIEvent?) -> UIView? {

    let (x, y) = (bounds.maxX - point.x, bounds.maxY - point.y)
    let r = x * x + y * y
    let lo = sliderRadius - sliderWidth - hitAreaOffset
    let hi = sliderRadius + sliderWidth + hitAreaOffset
    if (r >= lo * lo && r <= hi * hi) {
        return super.hitTest(point, withEvent: event)
    }
    return nil
}
```
如上，通常来说view的hitTest方法不需要手动调用，而是iOS系统在需要的时候调用

# 重写CALayer的hitTest方法

有时自定义的view包含多个layer，所以当此view被点中的时候，还希望知道具体是哪个layer区域被点中，从而进行不同的处理。这时候可以通过重写layer的hitTest方法进行判断

```
override func hitTest(p: CGPoint) -> CALayer? {

    let (dx, dy) = (p.x - position.x, p.y - position.y)
    let r = radius + markerBorderWidth + hitAreaOffset
    return (dx * dx + dy * dy <= r * r) ? self : nil
}
```

与view的hitTest不同的是，layer的hitTest方法通常需要开发者自己调用。比如在自定义的view上绑定了一个UITapGestureRecognizer，并且设置了delegate，那么当tap事件触发之后，就会进入以下方法
```
override func gestureRecognizerShouldBegin(gr: UIGestureRecognizer) -> Bool
```

需要注意，如果view上绑定了多个recognizer，那么每个手势都会触发进入此方法一次。在此方法中，可以手动调用layer的hitTest方法，来判断是哪个layer被点中
```
override func gestureRecognizerShouldBegin(gr: UIGestureRecognizer) -> Bool {
        
    let pos = gr.locationInView(self)
        
    if let group = selectedMarker as? WheelMarkerGroup where group.expanded {
               
        let p = layer.convertPoint(pos, toLayer: group.expandLayer)
        
        for marker in group.zones {
            if (marker.hitTest(p) != nil) {
                activeMarker = marker
                return true
            }
        }
    }
    
    return false
}
```