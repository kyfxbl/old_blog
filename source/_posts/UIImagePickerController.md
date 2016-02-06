title: UIImagePickerController
date: 2014-01-24 21:28
categories: iOS 
---
这个组件跟PopoverController一样，也是直接使用，不需要也不允许sub-classing的类，但是相对来说比PopoverController要复杂一些
<!--more-->

The <span class="s1">UIImagePickerController</span> class supports portrait mode only. <span style="color:#ff0000">This class is intended to be used as-is and does not support subclassing.</span> The view hierarchy for this class is private and must not be modified, with one exception. You can assign a custom view to the <span class="s2">cameraOverlayView</span> property and use that view to present additional information or manage the interactions between the camera interface and your code.

开发者不是通过继承的方式来定义此控件的行为，而是通过设置它的属性，以及实现其delegate方法

下面是最简单的示例代码，可以读取本地相册中的照片和视频：

```
- (void)buttonPressed
{
    UIImagePickerController *imagePicker = [[UIImagePickerController alloc] init];
    imagePicker.delegate = self;
    imagePicker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
    imagePicker.mediaTypes = [UIImagePickerController availableMediaTypesForSourceType:UIImagePickerControllerSourceTypePhotoLibrary];
    imagePicker.allowsEditing = YES;

    popoverController = [[UIPopoverController alloc] initWithContentViewController:imagePicker];
    popoverController.delegate = self;
    [popoverController presentPopoverFromRect:CGRectMake(20, 20, 400, 400) inView:self.view permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
}
```
上面的代码用到了PopoverController，这是因为在ipad环境下，只能通过popover的方式来展现ImagePicker组件，否则会抛出异常

下面总结ImagePicker最重要的几个属性：

sourceType，该属性决定ImagePicker的功能，只有3个枚举值。该属性是必须设置的

```
typedef NS_ENUM(NSInteger, UIImagePickerControllerSourceType) {
    UIImagePickerControllerSourceTypePhotoLibrary,
    UIImagePickerControllerSourceTypeCamera,
    UIImagePickerControllerSourceTypeSavedPhotosAlbum
};
```
mediaTypes，此属性设置能够展示的媒体类型，如图片，视频等，一般是通过UIImagePickerController的类方法availableMediaTypesForSourceType来得到

allowsEditing，此属性默认为NO，那么选中照片之后，就会立刻调用delegate方法（见下文），如果设置为YES，则会进入图片编辑的界面

delegate，设置delegate，一般是设置为当前的ViewController。此对象不仅要实现ImagePickerControllerDelegate，还需要实现NavigationControllerDelegate，因为ImagePickerController是继承自NavigationController的

```
@property(nonatomic,assign)    id <UINavigationControllerDelegate, UIImagePickerControllerDelegate> delegate;
```
下面是关键的delegate方法：

```
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info;
```
info里的key在xcode文档里有详细的说明

一般还需要实现这个方法：

```
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    [[UIApplication sharedApplication] setStatusBarHidden:YES];
}
```
因为弹出ImagePickerController的时候，会自动显示出status bar，所以如果你的应用是希望隐藏状态栏，就需要在这个delegate method中再次设置状态栏隐藏