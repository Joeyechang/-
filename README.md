# -
手势解锁&amp;涂鸦画板

#### 手势解锁
------------- 基本思路 --------------
- 1. 搭建界面，九宫格算法
- 2. 处理按钮选中状态
- 3. 按钮之间画线
- 4. 手指和按钮之间画线
- 5. 判断解锁密码是否正确
------------- 基本思路 --------------


1. 拖拽图片素材

2. 设置控制器 view 的背景色为"HomeButtomBG"图片平铺后的效果
self.view.backgroundColor = [UIColor colorWithPatternImage:[UIImage imageNamed:@"HomeButtomBG"]];

3. 在控制器中拖拽一个 UIView, 设置宽高都是300, 并设置约束(在父容器中水平、垂直居中对齐)。

4. 自定义一个 view, 设置这个自定义 view 与界面上的 view 相关联

5. 在自定义 view 中添加一个 buttons 属性, 在懒加载中生成9个 UIButton, 并且把这写 Button 都添加到 view 中。
- 创建每个 UIButton
- 设置每个 UIButton的默认情况下的背景图片、 selected 下的背景图片、disabled 下的背景图片

6. 在 layoutSubviews 方法中根据九宫格的方式计算每个按钮的 frame


7. 实现触摸到某个按钮的时候, 让这个按钮的状态变成 selected
7.1 在自定义 view 的 touchesBegan:方法中根据当前的触摸点, 判断是否触碰到了某个按钮
- 设置这个按钮的 selected状态
- 把这个按钮记录下来(添加到 selectedButtons 集合中)

7.2 在自定义 view 的 touchesMoved:方法中根据当前的触摸点, 判断是否触碰到了某个按钮
- 设置这个按钮的 selected状态
- 把这个按钮记录下来(添加到 selectedButtons 集合中), 同时要判断这个按钮是否已经是 selected = YES了, 避免重复添加。
- 同时记录下本次触摸的 point到 currentPoint属性中。
** 注意: 此处必须设置按钮禁止与用户交互(btn.userInteractionEnabled = NO;), 否者手指触摸到按钮上以后, 按钮会捕获这个触摸事件, 就不会触发 view的 touchesBegan:事件了。


8. 在 drawRect:方法中绘制线条
- 判断如果 self.selectedButtons长度为0, 那么直接返回不需要绘制任何内容
- 如果 self.selectedButtons长度不为0:
1> 循环 self.selectedButtons集合中的每个按钮
2> 判断如果是第一个按钮那么移动到起点（起点为第一个按钮的 center point）
3> 如果不是第一个按钮那么直接添加线段到这个按钮的 center point
4> 绘制完毕按钮以后, 最后再添加一条线段到 currentPoint。
// 线条的颜色
#define SteveZLineColor [UIColor colorWithRed:0.0 green:170/255.0 blue:255/255.0 alpha:1.0]

/** 
 
 错误:
 <Error>: void CGPathAddLineToPoint(CGMutablePathRef, const CGAffineTransform *, CGFloat, CGFloat): no current point.
 
 原因:
 在第一次绘图的时候 path 对象中没有任何点
 
 解决:(判断如果没有需要绘制的按钮, 那么就直接返回)
 if (self.selectedButtons.count == 0) {
    return;
 }
 
 */


9. 在touchesEnded:方法中, 将所有按钮的 selected 状态设置为 NO, 清空 self.selectedButtons 集合, 重新绘制。



10. 为每个按钮添加一个 tag, 用来判断手势解锁是否正确
- 通过代理把解锁的密码传递给控制器, 在控制器中判断解锁是否正确, 如果正确代理方法返回 YES, 否则返回 NO
- 在定义 view 中, 根据代理方法返回的结果判断如果 YES, 那么直接执行第9步
- 如果返回为 NO:
1> 先将 self.selectedButtons中的所有按钮selected状态设置为 NO, enabled 也设置为 NO
2> 在把线条颜色设置为红色
3> 执行重绘
4> 0.5秒钟之后, 在执行第9步。


11. 细节处理
- 透明问题
1> 如果是完全通过代码创建 UIView, 在绘图的时候, 如果指定了控件的透明（opaque = NO）opaque 表示不透明, 那么在绘图时会有问题。
2> 如果希望是透明, 那么要把颜色指定为 clearColor, 而不是 opaque = NO


- NSNumber 的使用问题
在对整数进行字符串拼接的时候不要使用 int,NSInteger等, 要使用NSNumber
/**
    避免出现下面代码:
    [strPassword appendFormat:@"%d", [self.selectedButtons[i] tag]];
    
    原因: 因为在 iPhone5s 以下都是32位, 从 iPhone5s 开始都成了64位, 所以 NSInteger 在 iPhone5s 以下模拟器为32位, iPhone5s（含）以上都是64位
 
    解决: (统一使用 NSNumber)
    [strPassword appendFormat:@"%@", @([self.selectedButtons[i] tag])];
 
 */

- NSInteger 的使用问题
1> 一般对象的属性、方法的参数可以使用使用 NSInteger (可以保证在不同平台上使用不同的整数（32位、64位）)
2> 方法内的局部变量, 一般使用 int
3> 以上是苹果官方示例程序中的代码习惯




#### 涂鸦画板

------------- 基本思路 --------------
1. 搭建界面

2. 实现画线功能
2.1 实现画单笔
2.2 实现画多笔
2.3 实现设置线宽
2.4 实现设置线条颜色

3. 实现清屏
4. 实现回退
5. 实现橡皮擦
6. 实现保存到相册功能
7. 实现插入照片功能

------------- 基本思路 --------------

1. 使用自动布局搭界面
- 顶部的 UIToolBar
- 底部的自定义 View
- 中间的绘图用的 View

2. 实现基本的绘图功能
- 通过自定义 UIView 实现
1> 重写自定义 View 的3个触摸方法
- touchBegan, 在这个方法中 moveToPoint
- touchMove, 在这个方法中 addLine
- drawRect, 在这个方法中把拼接好的路径渲染到 UIView上。

- 先实现绘制"单笔"
/** 参考代码:
 
 @interface SteveZPaintView ()
 @property (nonatomic, strong) UIBezierPath *path;
 @end
 
 
 @implementation SteveZPaintView
 
 
 - (CGPoint)pointWithTouches:(NSSet *)touches
 {
 UITouch *touch = touches.anyObject;
 return [touch locationInView:touch.view];
 }
 
 - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
 {
 self.path = [[UIBezierPath alloc] init];
 CGPoint point = [self pointWithTouches:touches];
 [self.path moveToPoint:point];
 }
 
 - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
 {
 CGPoint point = [self pointWithTouches:touches];
 [self.path addLineToPoint:point];
 // 重绘
 [self setNeedsDisplay];
 }
 
 
 
 
 - (void)drawRect:(CGRect)rect {
 // Drawing code
 [self.path stroke];
 }
 
 */

- 实现画"多笔"
/** 参考代码:
 @interface SteveZPaintView ()
 //@property (nonatomic, strong) UIBezierPath *path;
 @property (nonatomic, strong) NSMutableArray *paths;
 @end
 
 
 @implementation SteveZPaintView
 
 
 - (CGPoint)pointWithTouches:(NSSet *)touches
 {
 UITouch *touch = touches.anyObject;
 return [touch locationInView:touch.view];
 }
 
 - (NSMutableArray *)paths
 {
 if (_paths == nil) {
 _paths = [NSMutableArray array];
 }
 return _paths;
 }
 
 - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
 {
 UIBezierPath *path = [[UIBezierPath alloc] init];
 CGPoint point = [self pointWithTouches:touches];
 [path moveToPoint:point];
 [self.paths addObject:path];
 }
 
 - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
 {
 UIBezierPath *currentPath = [self.paths lastObject];
 
 CGPoint point = [self pointWithTouches:touches];
 [currentPath addLineToPoint:point];
 // 重绘
 [self setNeedsDisplay];
 }
 
 - (void)drawRect:(CGRect)rect {
 // Drawing code
 [self.path stroke];
 }

 
 */


- 实现设置"线宽"
1> 在 slider 的值改变事件中, 把线宽传递给自定义 paintView
2> 需要在自定义 view 中添加一个 lineWidth的属性
3> 在绘图的时候设置 path 的线宽
/** 参考代码
 - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
 {
     UIBezierPath *path = [[UIBezierPath alloc] init];
     path.lineWidth = self.lineWidth;
     
     CGPoint point = [self pointWithTouches:touches];
     [path moveToPoint:point];
     [self.paths addObject:path];
 }
 
 - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
 {
     UIBezierPath *currentPath = [self.paths lastObject];
     
     CGPoint point = [self pointWithTouches:touches];
     [currentPath addLineToPoint:point];
     // 重绘
     [self setNeedsDisplay];
 }
 
 - (void)drawRect:(CGRect)rect {
     // Drawing code
     for (UIBezierPath *path in self.paths) {
         path.lineCapStyle = kCGLineCapRound;
         path.lineJoinStyle = kCGLineJoinRound;
         [path stroke];
     }
 }
 
 */

- 实现设置"颜色"
* 因为每条线的颜色可能都不相同, 所以要为每条线保存一个颜色, 颜色要与 UIBezierPath对象保存在一起, 所以需要自定义一个类继承自 UIBezierPath, 添加一个颜色属性
* 然后在每次绘制的时候, 设置 path 对象的线条颜色
/** 参考:
 
 - (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
 {
     SteveZBezierPath *path = [[SteveZBezierPath alloc] init];
     path.lineColor = self.lineColor;
     path.lineWidth = self.lineWidth;
     
     CGPoint point = [self pointWithTouches:touches];
     [path moveToPoint:point];
     [self.paths addObject:path];
 }
 
 - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
 {
     SteveZBezierPath *currentPath = [self.paths lastObject];
     
     CGPoint point = [self pointWithTouches:touches];
     [currentPath addLineToPoint:point];
     // 重绘
     [self setNeedsDisplay];
 }
 
 - (void)drawRect:(CGRect)rect {
     // Drawing code
     for (SteveZBezierPath *path in self.paths) {
         path.lineCapStyle = kCGLineCapRound;
         path.lineJoinStyle = kCGLineJoinRound;
         [path.lineColor setStroke];
         [path stroke];
     }
 }
 
 
 */


- 清屏、回退、橡皮擦、保存
* 思路: 在控制器中调用在自定义 View中的下列方法
* 注意: 橡皮擦功能, 只需要设置笔的背景颜色与 绘图 view 的背景色一致。 然后绘图的时候就相当于"橡皮擦".
/** 参考:
 
 // 清屏
 - (void)clearScreen
 {
     [self.paths removeAllObjects];
     [self setNeedsDisplay];
 }
 
 // 回退（撤销）
 - (void)undo
 {
     [self.paths removeLastObject];
     [self setNeedsDisplay];
 }
 
 
 // 橡皮擦
 - (void)erase
 {
    self.lineColor = self.backgroundColor;
 // 此处不需要重绘, 点完橡皮擦以后，在 view 上move 的时候才需擦除
 }
 
 // 保存到相册
 - (void)save
 {
     UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0.0);
     CGContextRef ctx = UIGraphicsGetCurrentContext();
     [self.layer renderInContext:ctx];
     UIImage *imgPhoto = UIGraphicsGetImageFromCurrentImageContext();
     UIImageWriteToSavedPhotosAlbum(imgPhoto, nil, nil, nil);
 }
 
 
 */




- "选择照片"
* 思路:
0. 为"照片"按钮注册单击事件
1. 弹出选择照片的控制器
- 弹出 UIImagePickerController *pkVc = [[UIImagePickerController alloc] init];选择照片
- 设置 UIImagePickerController控制器的类型
pkVc.sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
- 设置控制器的代理(在代理方法中获取用户选择的图片, 并且把图片添加到"绘图 view 上", 然后关闭被 modal 出来的控制器)
- 把选择照片的控制器 modal 出来
/** 参考:
 // 照片按钮的单击事件
 - (IBAction)pickPhoto:(id)sender {
 // 选择图片控制器
 UIImagePickerController *pkVc = [[UIImagePickerController alloc] init];
 
 
 // 设置照片的来源, 相册还是拍照等
 // UIImagePickerControllerSourceTypePhotoLibrary 按照相册（相簿）来选择
 // UIImagePickerControllerSourceTypeSavedPhotosAlbum 直接从照片库中选（按照时间）
 pkVc.sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
 
 
 // 设置代理(在代理方法中, 编写完成照片选择后要做的事情)
 pkVc.delegate = self;
 
 // 通过 modal 的方式展示控制器
 // modal 出来的控制器一般都是与主要操作不相关, 临时用一下
 [self presentViewController:pkVc animated:YES completion:nil];
 }
 
 // UIImagePickerController的代理方法
 - (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
 {
 // 查看字典中的内容
 //NSLog(@"%@", info);
 
 // 获取选择的图片
 UIImage *img = info[UIImagePickerControllerOriginalImage];
 
 // 创建图片框
 UIImageView *imgView = [[UIImageView alloc] initWithImage:img];
 
 // 把图片添加到绘图 viwe 中
 [self.paintView addSubview:imgView];
 
 // 关闭 modal 出的控制器
 [self dismissViewControllerAnimated:YES completion:nil];
 }
 
 */




* 补充一个知识点:
- 关于 modal 出的控制器如何关闭的问题
1> 大多数情况下在被 modal 出来的控制器内部关闭这个控制器是没有问题的
2> 只有在极少数情况下有问题。
3> 其实当在被 modal 出的控制器内部调用 dismiss方法的时候, 最终还是把这个 dismiss消息发送给了 modal出当前控制器的控制器
4> 最正确的情况是"谁 Modal 出控制器, 谁负责 dismiss 控制器"
5> 但是如果"谁 Modal 出控制器, 谁负责 dismiss 控制器", 那么势必很多情况下要使用代理, 所以为了简单, 大多数情况下都是"在被 Modal 出的子控制器内部执行 dismiss"
/*
 关于 dismiss 被 modal 出来的控制器的讨论
 Discussion
 The presenting view controller is responsible for dismissing the view controller it presented. If you call this method on the presented view controller itself, it automatically forwards the message to the presenting view controller.
 */






2. 对添加到"绘图 View"中的 UIImageView控件实现"捏合"、"缩放"、"拖动"、"长按"各种效果
- 通过为 UIImageView添加各种"手势", 实现上面的功能

- 注意: 在"长按"UIImageView的时候, 把UIImageView中的图片集成到"绘图 View"中
* 思路:
1> 直接把选中的图片 drawInRect: 绘制到"绘图 View"上
2> 但是如果这个图片已经被缩放、旋转后, 再把图片直接绘制到"绘图 View"上就比较困难了
3> 解决: 先把 UIImageView添加到一个透明的 UIView 上, 然后再把这个透明的 UIView 内容绘制到一个 UIImage 上, 然后再把这个 UIImage绘制到"绘图上下文"中。
* 步骤:
1> 当选择好一个图片的时候, 创建一个自定 view（与"绘图 View"一样大）, 然后把 UIImageView添加到这个自定义 View 中。



3. 为照片添加手势, 捏合、缩放、(拖动)平移、长按
* 注意: 默认图片框不支持与用户交互, 所以无法识别手势, 需要设置 userInteractionEnabled = YES;
/** 参考代码:
 
 - (instancetype)initWithFrame:(CGRect)frame photo:(UIImage *)image
 {
 if (self = [super initWithFrame:frame]) {
 
 UIImageView *imgViewPhoto = [[UIImageView alloc] initWithImage:image];
 imgViewPhoto.userInteractionEnabled = YES;
 [self addSubview:imgViewPhoto];
 self.imgViewPhoto = imgViewPhoto;
 
 // 为 UIImageView 添加手势识别
 [self addGestures];
 
 }
 
 return self;
 }
 
 
 // 添加手势识别
 - (void)addGestures
 {
 // 1. 拖拽手势
 UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panGesture:)];
 [self.imgViewPhoto addGestureRecognizer:pan];
 
 
 // 2. 旋转手势
 UIRotationGestureRecognizer *rotation = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotationGesture:)];
 rotation.delegate = self;
 [self.imgViewPhoto addGestureRecognizer:rotation];
 
 
 
 // 3. 捏合手势（缩放）
 UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchGesture:)];
 pinch.delegate = self;
 [self.imgViewPhoto addGestureRecognizer:pinch];
 
 
 // 4. 长按手势(把图片绘制到 paint view 上)
 UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPressGesture:)];
 [self.imgViewPhoto addGestureRecognizer:longPress];
 }
 
 
 
 // 长按手势（把图片绘制到 paint view 中）
 - (void)longPressGesture:(UILongPressGestureRecognizer *)recognizer
 {
 if (recognizer.state == UIGestureRecognizerStateBegan) {
 
 // 1. 让图片闪一下, 在图片闪完以后把当前 view 中的内容渲染到 UIImage 中
 [UIView animateWithDuration:0.5 animations:^{
 
 self.imgViewPhoto.alpha = 0.5;
 
 } completion:^(BOOL finished) {
 
 [UIView animateWithDuration:0.5 animations:^{
 
 self.imgViewPhoto.alpha = 1.0;
 
 } completion:^(BOOL finished) {
 
 // 2. 通过调用代理, 把把图片绘制到 paint view 中
 // 把当前 view 渲染到 UIImage 当中, 然后通过代理把 UIIamge 传递回去
 if ([self.delegate respondsToSelector:@selector(photoView:withPhoto:)]) {
 UIImage *photoViewShot = [self imageWithViewShot];
 [self.delegate photoView:self withPhoto:photoViewShot];
 }
 
 }];
 }];
 }
 
 }
 
 // 把当前 view 渲染到 UIImage 中
 - (UIImage *)imageWithViewShot
 {
 UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0.0);
 CGContextRef ctx = UIGraphicsGetCurrentContext();
 [self.layer renderInContext:ctx];
 UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
 UIGraphicsEndImageContext();
 return img;
 }
 
 
 // 捏合手势, 进行缩放
 - (void)pinchGesture:(UIPinchGestureRecognizer *)recognizer
 {
 recognizer.view.transform = CGAffineTransformScale(recognizer.view.transform, recognizer.scale, recognizer.scale);
 recognizer.scale = 1.0;
 
 }
 
 // 旋转手势
 - (void)rotationGesture:(UIRotationGestureRecognizer *)recognizer
 {
 CGFloat rotation =  recognizer.rotation;
 recognizer.view.transform = CGAffineTransformRotate(recognizer.view.transform, rotation);
 recognizer.rotation = 0;
 }
 
 
 // 拖拽手势
 - (void)panGesture:(UIPanGestureRecognizer *)recognizer
 {
 CGPoint translation = [recognizer translationInView:recognizer.view];
 recognizer.view.transform = CGAffineTransformTranslate(recognizer.view.transform, translation.x, translation.y);
 [recognizer setTranslation:CGPointZero inView:recognizer.view];
 }
 
 */


4. 长按照片, 把照片集成到绘图视图中, 实现绘图
* 怎么把照片集成到绘图视图中?
- 直接把照片 drawInRect: 绘制到绘图视图中
- 但是如果照片被旋转、平移后如何把现在的照片绘制到视图中? 思路: 先把照片绘制到一个透明的图层（另外一个自定义 View）上。

* 把包含 UIImageView 的视图绘制到"绘图 view"上的具体步骤:
- 思考: "绘图 view"上的内容是如何绘制出来的
- 回答: 是在 Paint View 的 drawInRect:方法中循环把一个一个的 UIBezierPath中的路径绘制上去的。
- 结论: 那么要想把图片绘制到 paint view 上, 就必须在 paths 集合中添加一个可以绘制图片的 path 对象, 然后在循环的时候, 就会把路径和图片一起绘制到 paint view上了。
- 步骤:
1> 在自定义 BezierPath 对象中添加一个 UIImage 属性, 用来保存要绘制的图片
2> 在paint view的 drawRect: 方法中判断如果当前的 path 对象有 image, 那么就绘制图片[path.image drawInRect:rect];否则就画线[path stroke];
3> 在自定义 paint view 中添加一个 image 属性, 用来接收从控制器中传递过来的要绘制到 paint view 上的图片
4> 重写 paint view 的 setImage 方法, 在这个方法中, 创建一个自定义 BezierPath对象, 并且设置 image属性, 把这个 BezierPath对象添加到 self.paths 集合中, 然后执行重绘

/** 参考代码:
 ------------------- 自定义的 HMPhotoView-------------------
 #import <UIKit/UIKit.h>
 @class HMPhotoView;
 
 @protocol HMPhotoViewDelegate <NSObject>
 
 - (void)photoView:(HMPhotoView *)photoView withPhoto:(UIImage *)image;
 
 @end
 
 @interface HMPhotoView : UIView
 //@property (nonatomic, strong) UIImage *photo;
 
 - (instancetype)initWithFrame:(CGRect)frame photo:(UIImage *)image;
 
 @property (nonatomic, weak) id<HMPhotoViewDelegate> delegate;
 
 @end
 
 
 
 
 
 // 长按手势（把图片绘制到 paint view 中）
 - (void)longPressGesture:(UILongPressGestureRecognizer *)recognizer
 {
 if (recognizer.state == UIGestureRecognizerStateBegan) {
 // 1. 让图片闪一下, 在图片闪完以后把当前 view 中的内容渲染到 UIImage 中
 [UIView animateWithDuration:0.5 animations:^{
 self.imgViewPhoto.alpha = 0.5;
 } completion:^(BOOL finished) {
 [UIView animateWithDuration:0.5 animations:^{
 self.imgViewPhoto.alpha = 1.0;
 } completion:^(BOOL finished) {
 // 2. 通过调用代理, 把把图片绘制到 paint view 中
 // 把当前 view 渲染到 UIImage 当中, 然后通过代理把 UIIamge 传递回去
 if ([self.delegate respondsToSelector:@selector(photoView:withPhoto:)]) {
 UIImage *photoViewShot = [self imageWithViewShot];
 [self.delegate photoView:self withPhoto:photoViewShot];
 }
 
 }];
 }];
 }
 
 }
 ------------------- 自定义的 HMPhotoView-------------------
 
 
 
 
 
 
 
 
 
 
 
 
 
 ------------------- 自定义的 HMBezierPath-------------------
 #import <UIKit/UIKit.h>
 
 @interface HMBezierPath : UIBezierPath
 @property (nonatomic, strong) UIColor *lineColor;
 @property (nonatomic, strong) UIImage *image;  // 添加了一个 UIImage 属性
 @end
 ------------------- 自定义的 HMBezierPath-------------------
 
 
 
 
 
 
 
 
 
 
 ------------------- 自定义的绘图 HMPaintView-------------------
 1. 添加一个@property (nonatomic, strong) UIImage *image;属性
 2. 重写 setImage:方法
 - (void)setImage:(UIImage *)image
 {
 _image = image;
 HMBezierPath *path = [[HMBezierPath alloc] init];
 path.image = image;
 [self.paths addObject:path];
 
 // 重绘
 [self setNeedsDisplay];
 }
 
 3. 修改 drawRect:方法
 - (void)drawRect:(CGRect)rect {
 // 把paths数组中的每一个UIBezierPath对象都stroke一下
 for (HMBezierPath *path in self.paths) {
 
 if (path.image) {
 [path.image drawInRect:rect];
 } else {
 path.lineCapStyle = kCGLineCapRound;
 path.lineJoinStyle = kCGLineJoinRound;
 
 // 每个path对象在渲染之前, 设置使用自己的颜色来渲染
 [path.lineColor set];
 
 [path stroke];
 }
 }
 
 }
 
 ------------------- 自定义的绘图 HMPaintView-------------------
 
 
 
 
 
 
 
 -------------------控制器代码-------------------
 // photoView 的代理方法
 - (void)photoView:(HMPhotoView *)photoView withPhoto:(UIImage *)image
 {
 self.paintView.image = image;
 
 // 删除 HMPhotoView
 [self.imgViewPhoto removeFromSuperview];
 }
 
 - (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
 {
 NSLog(@"%@", info);
 UIImage *photo = info[UIImagePickerControllerOriginalImage];
 HMPhotoView *imgViewPhoto = [[HMPhotoView alloc] initWithFrame:self.paintView.frame photo:photo];
 imgViewPhoto.delegate = self;
 [self.view addSubview:imgViewPhoto];
 self.imgViewPhoto = imgViewPhoto;
 
 // 关闭选择照片控制器
 [self dismissViewControllerAnimated:YES completion:nil];
 }
 
 -------------------控制器代码-------------------
 
 */


####手势操作相关
1.1 UIEvent 对象介绍(了解)【UIEvent是一个事件对象】
- 每产生一个事件，就会产生一个UIEvent对象
- UIEvent: 称为事件对象，记录事件产生的时刻和类型
- 一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数
- 常见属性:(打开看枚举类型, 远程控制事件中又分很多子类型)
@property(nonatomic,readonly) UIEventType     type;
@property(nonatomic,readonly) UIEventSubtype  subtype;

1.2 UIResponder 对象
- 响应者对象, 能与用户交互的就是响应者对象
- UIApplication、UIViewController、UIView都继承自UIResponder，因此它们都是响应者对象，都能够接收并处理事件


1.3 注意:
- 如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象

- 如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象

- 根据touches中UITouch的个数可以判断出是单点触摸还是多点触摸
* 如果一根手指触摸屏幕叫做单点触摸
* 如果多根手指同时触摸屏幕叫做多点触摸



1.4 总结:
- 4个触摸事件处理方法中, 都有NSSet *touches和UIEvent *event两个参数, 一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数。在一次完整的触摸事件中 touches 集合中的 UITouch对象也是在各个阶段共享的。






2. 多点触摸（案例）
- 思路:
0> 把控制器 view的背景色变成黑色
1> 重写控制器的 touchesBegan 方法和 touchesMoved 方法
2> 在控制器中添加 images 属性, 在该属性中保存2张 UIImage 图片
3> 在 touchesBegan 方法中,:
* 获取每一个触摸对象, 以及每个触摸对象的 location
* 对于每个触摸对象创建一个 UIImageView, 并向其中添加一个 UIImage对象
* 最后把这个 UIImageView 添加到 self.view 中
4> 在 touchesMoved 方法中执行与 touchesBegan 方法中同样的代码
5> 在 touchesEnded 方法中输出当前 self.view 中的 UIImageView 的个数
6> 在每次添加完毕一个 UIImageView 后, 执行一个 alpha = 0的动画, 动画执行完毕从 self.view 中移除这个控件
** 注意: 一般默认情况下, 控件都不支持多点触摸(为了提高性能), 所以需要手动设置一个 UIView 允许多点触摸

/** 参考代码:

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    int i = 0;
    
    for (UITouch *touch in touches) {
        
        CGPoint loc = [touch locationInView:self.view];
        UIImageView *imgView = [[UIImageView alloc] initWithImage:self.images[i]];
        imgView.center = loc;
        [self.view addSubview:imgView];
        
        i++;
        
        
        // 慢慢消失
        [UIView animateWithDuration:2.0 animations:^{
            imgView.alpha = 0;
        } completion:^(BOOL finished) {
            [imgView removeFromSuperview];
        }];
    }
}


- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
{
    int i = 0;
    
    for (UITouch *touch in touches) {
        
        CGPoint loc = [touch locationInView:self.view];
        UIImageView *imgView = [[UIImageView alloc] initWithImage:self.images[i]];
        imgView.center = loc;
        [self.view addSubview:imgView];
        
        i++;
        
        
        // 慢慢消失
        [UIView animateWithDuration:2.0 animations:^{
            imgView.alpha = 0;
        } completion:^(BOOL finished) {
            [imgView removeFromSuperview];
        }];
    }
}


*/







3. 介绍控件不能接受用户交互的情况
* 演示要求:在控制器 view 中创建多个 UIView, 并且为每个 UIView 指定一个自定义 View, 实现 touchesBegan 方法.

1> 控件的userInteractionEnabled = NO 的情况下不能与用户交互
** 注意: 如果父容器不能与用户交互, 那么在该容器中的所有子控件也不能与用户交互（例如: 添加在 UIImageView 中的按钮）

2> 透明度小于等于0.01, alpha = 0.01
3> 控件被隐藏的时候, hidden = YES
4> 如果子视图的位置超出了父视图的有效范围, 那么子视图也是无法与用户交互的, 即使设置了父视图的 clipsToBounds = NO, 可以看懂, 但是也是无法与用户交互的
5> 默认情况下, 从控件库中拖拽的 UIImageView 是无法接受用户的触摸事件的
** 演示向 UIImageView 中添加一个按钮, 监听按钮的点击事件。
** UIImageView 默认是不支持多点触摸, 也不响应用户事件的。
补充: 直接从媒体库中把图片拖拽进来（通过这种方式拖进来的UIImageView, 默认即支持多点触摸也支持用户交互, 并且图片框大小就是图片的实际大小）







4. 事件响应链条
4.1 先看现象（查看 test 项目(预03-UIView不接受用户交互的情况)）
- 触摸谁, 谁的触摸事件被执行（父容器的触摸事件也有可能被执行）, 系统是如何找到被触摸的控件的？
- 按钮的单击事件是如何被触发的?


4.2 再看原理
- 什么是响应者? 能与用户交互就是响应者。所有继承自 UIResponder 的类型.
- 监听事件的基本流程:
1> 当应用程序启动以后创建 UIApplication 对象
2> 然后启动“消息循环”监听所有的事件
3> 当用户触摸屏幕的时候, "消息循环"监听到这个触摸事件
4> "消息循环" 首先把监听到的触摸事件传递了 UIApplication 对象
5> UIApplication 对象再传递给 UIWindow 对象
6> UIWindow 对象再传递给 UIWindow 的根控制器(rootViewController)
7> 控制器再传递给控制器所管理的 view
8> 控制器所管理的 View 在其内部搜索看本次触摸的点在哪个控件的范围内
9> 找到某个控件以后(调用这个控件的 touchesXxx 方法), 再一次向上返回, 最终返回给"消息循环"
10> "消息循环"知道哪个按钮被点击后, 在搜索这个按钮是否注册了对应的事件, 如果注册了, 那么就调用这个"事件处理"程序。（一般就是执行控制器中的"事件处理"方法）

** 解释触摸事件和单击事件的关系: 为按钮注册单击事件, 其实都是通过触摸事件找到对应的控件, 然后调用控件的单击事件的处理程序来执行的。
** 注意: 如果父控件不能处理"触摸事件", 那么子控件是不可能接收到"触摸事件"的, 事件链条断了。




- 演示 hitTest方法的执行(系统是通过 hitTest 方法来进行递归搜索的)
1> - (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;  这个方法是 UIView 中的一个方法
2> 参数 point 表示是相对于当前 UIView 的点位置。用来判断这个触摸点是否在当前视图的"有效范围"内。
3> 这个方法的作用, 当控制器 view 接收到"触摸事件"后, 调用自己的 hitTest 方法, 递归搜索子控件, 看看是否有更合适的子控件来处理这个事件（如果这个触摸点在具体的某个子控件的有效范围内, 那么这个子控件就是"更合适的控件"）, 找到某个子控件以后, 这个子控件再调用自己的 hitTest 方法依次类推继续查找"更合适的子控件", 直到没有子控件了或者没有包含这个点的子控件了
4> 如果当前控件的 userInteractionEnabled = NO, 那么就不再搜索它的子控件了。
5> 如果在 hitTest 方法中, 没有调用 super 的 hitTest 方法, 只是直接返回 self, 那么这样就终止了这个搜索过程, 系统会认为当前处理这个触摸的控件就是这个控件。


5. 手势解锁案例
- 参考手势解锁步骤.m


6. 涂鸦案例



7. 手势识别
- 触摸事件只有4个:
1> 按下 touchesBegan
2> 移动 touchesMoved
3> 抬起 touchesEnded
4> 取消 touchesCanceled
- iOS3.2之后, 把触摸事件做了封装, 对常用的手势进行了处理, 封装了6种常见的手势
UITapGestureRecognizer(敲击)
UILongPressGestureRecognizer(长按)
UISwipeGestureRecognizer(轻扫)
UIRotationGestureRecognizer(旋转)
UIPinchGestureRecognizer(捏合，用于缩放)
UIPanGestureRecognizer(拖拽)


- 手势识别是单独添加到某个视图上的。

- 点按、点击、敲击手势
/** 参考:
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UITapGestureRecognizer *tapGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tabGestureHandler)];
    // 需要的最小点击数
    tapGesture.numberOfTapsRequired = 2;
    // 需要的最少触摸点
    tapGesture.numberOfTouchesRequired = 1;
    
    [self.imgView addGestureRecognizer:tapGesture];
}

- (void)tabGestureHandler
{
    NSLog(@"tab..");
}
*/


- 长按手势
/** 参考代码:
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UILongPressGestureRecognizer *longPressGesture = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPressGestureHandler:)];
    
    // 最小的长按时间
    //longPressGesture.minimumPressDuration = 1.5;
    
    [self.imgView addGestureRecognizer:longPressGesture];
}


- (void)longPressGestureHandler:(UILongPressGestureRecognizer *)recognizer
{
    // 父类中的 state 属性来判断当前的状态
    if (recognizer.state == UIGestureRecognizerStateBegan) {
        NSLog(@"长按手势被识别。。。");
        
        [UIView animateWithDuration:1.0 animations:^{
            self.imgView.alpha = 0.3;
        } completion:^(BOOL finished) {
            [UIView animateWithDuration:1.0 animations:^{
                self.imgView.alpha = 1.0;
            }];
        }];
    }
}
*/

- 轻扫手势
/** 参考:
- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 注意, 要想支持同时向多个方向的轻扫, 需要创建2个 SwipeGesture, 一个向左, 一个向右
    UISwipeGestureRecognizer *swipeGestureLeft = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipeGestureHandler:)];
    // 指定轻扫方向
    swipeGestureLeft.direction = UISwipeGestureRecognizerDirectionLeft;
    
    
    UISwipeGestureRecognizer *swipeGestureRight = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipeGestureHandler:)];
    // 不设置, 默认也是向右
    swipeGestureRight.direction = UISwipeGestureRecognizerDirectionRight;
    
    
    [self.imgView addGestureRecognizer:swipeGestureLeft];
    [self.imgView addGestureRecognizer:swipeGestureRight];
    
   
}

- (void)swipeGestureHandler:(UISwipeGestureRecognizer *)recognizer
{
    NSLog(@"轻扫了。。");
    CGPoint from = recognizer.view.center;
    CGPoint to;
    // 判断轻扫的方向
    if (recognizer.direction == UISwipeGestureRecognizerDirectionLeft) {
        to = CGPointMake(-2 * from.x , from.y);
    } else {
        to = CGPointMake(3 * from.x, from.y);
    }
    [UIView animateWithDuration:1.0 animations:^{
        recognizer.view.center = to;
    } completion:^(BOOL finished) {
        [UIView animateWithDuration:1.0 animations:^{
            recognizer.view.center = from;
        }];
    }];
}

*/

- 旋转手势
* 说明: 
1> recognizer.rotation 默认表示的是当前手势旋转了的度数, 如果持续旋转了2圈, 那么这个 rotaion 就是4*π了
2> 每次出发一次旋转手势, 将 rotation 复位到0, 那么每次就是当前本次的旋转度数, 不是从手势一开始选在到现在的总的度数
/** 参考:

	- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 旋转手势
    UIRotationGestureRecognizer *rotationGesture = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotationGestureHandler:)];
    
    [self.imgView addGestureRecognizer:rotationGesture];
   
}


- (void)rotationGestureHandler:(UIRotationGestureRecognizer *)recognizer
{
    //NSLog(@"旋转。。。。");
    NSLog(@"%f", recognizer.rotation);
    //recognizer.view.transform = CGAffineTransformMakeRotation(recognizer.rotation);
    recognizer.view.transform = CGAffineTransformRotate(recognizer.view.transform, recognizer.rotation);
    recognizer.rotation = 0;
}
	
*/


- 捏合手势(缩放)
/** 参考:

- (void)viewDidLoad {
    [super viewDidLoad];
    
    
    UIPinchGestureRecognizer *pinchGesture = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchGestureHandler:)];
    
    [self.imgView addGestureRecognizer:pinchGesture];
   
}

- (void)pinchGestureHandler:(UIPinchGestureRecognizer *)recognizer
{
    
    //recognizer.view.transform = CGAffineTransformMakeScale(recognizer.scale, recognizer.scale);
    
    recognizer.view.transform = CGAffineTransformScale(recognizer.view.transform, recognizer.scale, recognizer.scale);
    recognizer.scale = 1.0;
    
}

//------------------ 同时应用2个手势--------------------- 
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    return YES;
}
- (void)viewDidLoad {
    [super viewDidLoad];
    
    
    // 缩放
    UIPinchGestureRecognizer *pinchGesture = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinchGestureHandler:)];
    //pinchGesture.delegate = self;
    [self.imgView addGestureRecognizer:pinchGesture];
    
    
    // 旋转手势
    UIRotationGestureRecognizer *rotationGesture = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotationGestureHandler:)];
    rotationGesture.delegate = self;
    [self.imgView addGestureRecognizer:rotationGesture];
   
}

- (void)pinchGestureHandler:(UIPinchGestureRecognizer *)recognizer
{
    
    //recognizer.view.transform = CGAffineTransformMakeScale(recognizer.scale, recognizer.scale);
    
    recognizer.view.transform = CGAffineTransformScale(recognizer.view.transform, recognizer.scale, recognizer.scale);
    recognizer.scale = 1.0;
    
}


*/
//------------------ 同时应用2个手势--------------------- 



- 拖动、拖拽手势: UIPanGestureRecognizer(拖拽)
/** 参考:
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //拖动手势
    UIPanGestureRecognizer *panGesture = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panGestureHandler:)];
    [self.imgView addGestureRecognizer:panGesture];
}

- (void)panGestureHandler:(UIPanGestureRecognizer *)recognizer
{
    CGPoint translation = [recognizer translationInView:recognizer.view];
    recognizer.view.transform = CGAffineTransformTranslate(recognizer.view.transform, translation.x, translation.y);
    
    // 复位
    [recognizer setTranslation:CGPointZero inView:recognizer.view];
}
*/

8. 摇一摇
/** 参考:
// 实现加速计事件, 判断如果是 Shake 那么表示"摇一摇"
- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event
{
    if (motion == UIEventSubtypeMotionShake) {
        NSLog(@"摇一摇....");
    }
}

*/




