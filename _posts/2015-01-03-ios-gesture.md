---
layout: post
title:  "iOS的手势控制"
tags: [iOS]
---


手势自然就是在真机上面，除了点击这种短暂动作之外的操作，主要包括：

### Pan
Pan就是最常见的拖动了。单个手指从屏幕滑动，系统会识别出这是一个pan gesture，当然代码里面需要包含一些固有的定义，才可以对这个手势做反应。 

> 注：在xcode的模拟器上面，只需要按住鼠标（or触摸板）不放，然后拖动，就是模仿真机中的pan了。

### Rotation
当两个手指同时触摸在屏幕，然后做同旋转方向的曲线拖动，就是Rotation 这个gesture

> 注：在xcode的模拟器上面，需要按住键盘的Option，同时按住鼠标（or触摸板）并开始拖动，模拟器上会模拟出两个小白圆，作为rotation。

### Pinch
当两个手指同时触摸在屏幕，然后做反向直线拖动，就是Pinch

> 注：在xcode的模拟器上面，需要按住键盘的Option，同时按住鼠标（or触摸板）并开始拖动，模拟器上会模拟出两个小白圆，作为rotation。

前端时间做了一个剪切图片的cropper，其中用到了手势控制，直接整个.m分享出来，应该能看懂的。。

``` 
#import "HYCImageCropper.h"

@interface HYCImageCropper ()
{
    UIImageView *imageView;
}

//rotation gesture需要的中间变量，记录上一个rotation参数
@property CGFloat lastRotation;

@end

@implementation HYCImageCropper

/*
// Only override drawRect: if you perform custom drawing.
// An empty implementation adversely affects performance during animation.
- (void)drawRect:(CGRect)rect {
    // Drawing code
}
*/

-(id)initWithFrame:(CGRect)frame Image:(UIImage *)image Area:(CGRect)area
{
    self = [super initWithFrame:frame];
    self.backgroundColor = [UIColor whiteColor];
   
    //add image view and its image
    imageView = [[UIImageView alloc]initWithFrame:self.bounds];
    [imageView setImage:image];
    [self addSubview: imageView];
    [imageView setUserInteractionEnabled:YES];
   
    //add gestures on imageView
    UIRotationGestureRecognizer *rotateGes = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotateImage:)];
    rotateGes.delegate = self;
    [imageView addGestureRecognizer:rotateGes];
   
    UIPinchGestureRecognizer *scaleGes = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(scaleImage:)];
    scaleGes.delegate = self;
    [imageView addGestureRecognizer:scaleGes];
   
    UIPanGestureRecognizer *moveGes = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(moveImage:)];
    moveGes.delegate = self;
    [imageView addGestureRecognizer:moveGes];

    return self;
}


- (void)moveImage:(UIPanGestureRecognizer *)sender
{
    NSLog(@"move!");
   
    //使目标view升到表层
    [self bringSubviewToFront:imageView];
    //求当前gesture在主view中的坐标
    CGPoint location = [sender locationInView:self];
    //利用center属性直接定义目标view位置
    sender.view.center = CGPointMake(location.x,  location.y);
   
}

- (void)scaleImage:(UIPinchGestureRecognizer *)sender
{
    NSLog(@"scale!");
   
    //使目标view升到表层
    [self bringSubviewToFront:sender.view];
    //求当前gesture在主view中的坐标
    CGPoint location = [sender locationInView:self];
    sender.view.center = CGPointMake(location.x, location.y);
    //用core graph里面封装好的CGAffineTransformScale做scale变换
    sender.view.transform = CGAffineTransformScale(sender.view.transform, sender.scale, sender.scale);
    sender.scale = 1;

}

- (void)rotateImage:(UIRotationGestureRecognizer *)sender
{
    NSLog(@"rotate");
   
    //使目标view升到表层
    [self bringSubviewToFront:sender.view];
    //求当前gesture在主view中的坐标
    CGPoint location = [sender locationInView:self];
    sender.view.center = CGPointMake(location.x, location.y);
    //如果没有旋转，就把lastRotate参数置为0
    if ([sender state] == UIGestureRecognizerStateEnded) {
        self.lastRotation = 0;
        return;
    }
    //找到上一次的transform参数，以便接着旋转
    CGAffineTransform currentTransform = imageView.transform;
    //求出新的旋转参数，并使用CGAffineTransformRotate
    CGFloat rotation = 0.0 - (self.lastRotation - sender.rotation);
    CGAffineTransform newTransform = CGAffineTransformRotate(currentTransform, rotation);
    imageView.transform = newTransform;
    //当rotate gesture停了，就记录这时候的状态到lastRotate这个变量，一边下一次接着旋转
    self.lastRotation = sender.rotation;
}

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer {
    return  YES;
}

@end

```

