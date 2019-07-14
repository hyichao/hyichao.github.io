---
layout: post
title:  "学习使用facebook pop动画"
tags: [iOS]
---

## 1 简介
转载一篇简介性质的短文，有图有真相，讲述了facebook pop在iOS做Animation的时候能够为我们带来什么。
<http://blog.csdn.net/hyichao_csdn/article/details/41724935>

## 2 弹簧特效 Spring Animation
假如要对一个view进行一种animation，使得这个view往下移动一段距离，弹一弹然后停住，就要利用POPSpringAnimation这个类。
很大程度上参考了popping那份代码。感谢

直接上代码：

``` 
-(void)moveDownView:(UIView *)view
{
    /*
     kPOPLayerPosition意思是这个animation对象要干的事情是移动layer的Position
     还有很多其他类型的spring animation
     kPOPLayerOpacity 透明度渐变
     kPOPLayerRotation 旋转渐变
     kPOPLayerScaleXY 大小渐变
     kPOPLayerTranslationXY 仿射变换渐变
     等等等等。。实在太多不宜列举
    */
    POPSpringAnimation *positionAnimation = [POPSpringAnimation animationWithPropertyNamed:kPOPLayerPosition];
   
    /*
     toValue是设置变化后的参数，例如PositionAnimation的话，toValue就是终点的坐标
     1.springBounciness 弹簧弹力 取值范围为[0, 20]，默认值为4
     2.springSpeed 弹簧速度，速度越快，动画时间越短 [0, 20]，默认为12，和springBounciness一起决定着弹簧动画的效果
     3.dynamicsTension 弹簧的张力
     4.dynamicsFriction 弹簧摩擦
     5.dynamicsMass 质量 。张力，摩擦，质量这三者可以从更细的粒度上替代springBounciness和springSpeed控制弹簧动画的效果
     */
    positionAnimation.toValue = [NSValue valueWithCGPoint:CGPointMake(view.center.x,view.center.y+100)];
    positionAnimation.springSpeed = 1.0f;
    positionAnimation.springBounciness = 20.0f;
   
    /*
     设置好参数后，就可以用函数pop_addAnimation: forKey来开始animation。这里的key应该是记录这次animation的设置，保存到某个堆栈，在需要的时候重新调用出来。
     */
    [view.layer pop_addAnimation:positionAnimation forKey:@"layerPositionAnimation"];
   
   
    POPSpringAnimation *scaleAnimation = [POPSpringAnimation animationWithPropertyNamed:kPOPLayerScaleXY];
    scaleAnimation.toValue = [NSValue valueWithCGSize:CGSizeMake(0.5, 0.5)];
    scaleAnimation.springBounciness = 10.f;
    [view.layer pop_addAnimation:scaleAnimation forKey:@"scaleAnimation"];
}
```

调用这个函数，就可以使得view开始animation，例如配合一个touchUpInside

```
- (void)touchUpInside:(UIControl *)sender {
    AnimationInfo animationInfo = [self animationInfoForLayer:sender.layer];
    BOOL hasAnimations = sender.layer.pop_animationKeys.count;
   
    if (hasAnimations && animationInfo.progress < 0.98) {
        [self pauseAllAnimations:NO forLayer:sender.layer];
        return;
    }
   
    [sender.layer pop_removeAllAnimations];

    [self moveDownView:sender];
}
```

* 特别备注一：
测试了几组参数，先描述如下

| springSpeed| springBounciness| description  |
| :--------: |:---------------:| :-----------:|
| 1	          | 20     | 总体速度缓慢，振动比较明显|
| 1          | 5      | 总体速度缓慢，振动非常不明显。|
| 5          | 20     | 速度提升，振动明显|
| 15         | 20     | 速度相当快，振动明显 |


POPSpringAnimation里面除了位置，大小之外还有很多其他的animation效果。
从别人那里摘录下来，虽然没有试验过，但是我读了下代码基本是对的，应该没有太大问题。

* 这个动效将按钮旋转

``` 
POPSpringAnimation *rotationAnimation = [POPSpringAnimation animationWithPropertyNamed:kPOPLayerRotation];  
   rotationAnimation.beginTime = CACurrentMediaTime() + 0.2;  
   rotationAnimation.toValue = @(1.2);  
   rotationAnimation.springBounciness = 10.f;  
   rotationAnimation.springSpeed = 3;  
   [view.layer pop_addAnimation:rotationAnimation forKey:@"rotationAnim"];  
```

* 这个改变透明度：
 
```
POPBasicAnimation *opacityAnimation = [POPBasicAnimation animationWithPropertyNamed:kPOPLayerOpacity];  
    opacityAnimation.toValue = @(0.5);  
    [view.layer pop_addAnimation:opacityAnimation forKey:@"opacityAnimation"];  
```

记得toValue一定是个对象，所以传入的值要加上一个@在前面，或者用NSValue把CGRect，CGPoint等等封装起来。

## 3 衰减特效 Decay Animation
除了上面提到的几种Spring Animation，Decay Animation的效果也非常炫酷。
Decay就是衰减的意思，例如

```
POPDecayAnimation *anim = [POPDecayAnimation animWithPropertyNamed:kPOPLayerPositionX];   
anim.velocity = @(100.0);   
anim.fromValue =  @(25.0);   
//anim.deceleration = 0.998;   
anim.completionBlock = ^(POPAnimation *anim, BOOL finished) {   
  if (finished) {NSLog(@"Stop!");}
  };   
```
这个动画会使得物体从 X 坐标的点 25.0 开始按照速率 100点／s 做减速运动。 这里非常值得一提的是，velocity 也是必须和你操作的属性有相同的结构，如果你操作的是 bounds，想实现一个水滴滴到桌面的扩散效果，那么应该是

`[NSValue valueWithCGRect:CGRectMake(0, 0,20.0, 20.0)]`
 
如果 velocity 是负值，那么就会反向递减。
 
deceleration （负加速度） 是一个你会很少用到的值，默认是就是我们地球的 0.998，如果你开发给火星人用，那么这个值你使用 0.376 会更合适。

> 特别备注：
这里的velocity就是起始速度，默认衰减速度是重力加速度，然后fromValue就是开始的位置。。所以，view结束的位置还要通过计算得出，比较麻烦。

