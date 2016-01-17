title: 用drawAtPoint绘制文字
date: 2014-07-07 15:29
categories: iOS 
---
从iOS7开始，drawAtPoint:WithFont:等方法已经deprecated，取而代之应该使用drawAtPoint:WithAttributes方法，来设置字体的颜色和大小
<!--more-->

```
NSDictionary *attrs = [NSDictionary dictionaryWithObjectsAndKeys:[UIFont systemFontOfSize:14], NSFontAttributeName, [UIColor colorWithRed:114/255.0f green:128/255.0f blue:137/255.0f alpha:1.0f], NSForegroundColorAttributeName, nil];
[item.title drawAtPoint:textPoint withAttributes:attrs];
```

支持配置的属性名，定义在NSAttributedString.h里：

```
UIKIT_EXTERN NSString *const NSFontAttributeName NS_AVAILABLE_IOS(6_0);               
UIKIT_EXTERN NSString *const NSParagraphStyleAttributeName NS_AVAILABLE_IOS(6_0);      
UIKIT_EXTERN NSString *const NSForegroundColorAttributeName NS_AVAILABLE_IOS(6_0);     
UIKIT_EXTERN NSString *const NSBackgroundColorAttributeName NS_AVAILABLE_IOS(6_0);     
UIKIT_EXTERN NSString *const NSLigatureAttributeName NS_AVAILABLE_IOS(6_0);            
UIKIT_EXTERN NSString *const NSKernAttributeName NS_AVAILABLE_IOS(6_0);               
UIKIT_EXTERN NSString *const NSStrikethroughStyleAttributeName NS_AVAILABLE_IOS(6_0); 
UIKIT_EXTERN NSString *const NSUnderlineStyleAttributeName NS_AVAILABLE_IOS(6_0);      
UIKIT_EXTERN NSString *const NSStrokeColorAttributeName NS_AVAILABLE_IOS(6_0);        
UIKIT_EXTERN NSString *const NSStrokeWidthAttributeName NS_AVAILABLE_IOS(6_0);        
UIKIT_EXTERN NSString *const NSShadowAttributeName NS_AVAILABLE_IOS(6_0);             
UIKIT_EXTERN NSString *const NSTextEffectAttributeName NS_AVAILABLE_IOS(7_0);          
UIKIT_EXTERN NSString *const NSAttachmentAttributeName NS_AVAILABLE_IOS(7_0);          
UIKIT_EXTERN NSString *const NSLinkAttributeName NS_AVAILABLE_IOS(7_0);               
UIKIT_EXTERN NSString *const NSBaselineOffsetAttributeName NS_AVAILABLE_IOS(7_0);      
UIKIT_EXTERN NSString *const NSUnderlineColorAttributeName NS_AVAILABLE_IOS(7_0);      
UIKIT_EXTERN NSString *const NSStrikethroughColorAttributeName NS_AVAILABLE_IOS(7_0); 
UIKIT_EXTERN NSString *const NSObliquenessAttributeName NS_AVAILABLE_IOS(7_0);        
UIKIT_EXTERN NSString *const NSExpansionAttributeName NS_AVAILABLE_IOS(7_0);          
UIKIT_EXTERN NSString *const NSWritingDirectionAttributeName NS_AVAILABLE_IOS(7_0);    
UIKIT_EXTERN NSString *const NSVerticalGlyphFormAttributeName NS_AVAILABLE_IOS(6_0);
```