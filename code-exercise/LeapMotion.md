# LeapMotion Unity使用

在Unity下，使用LeapServiceProvider获取Frame对象来解析手和手指的位姿、速度和方向。

## 添加一个手到场景
1. 将LeapHandController prefab放置到场景中期望手出现的位置处。可以改变HandMovementScale来在放大手的占用体积。
2. 创建空的GameObject命名为HandModels，这个对象与LeapHandController并列层级。
3. 拖动想要的hand prefab到HandModels下。
4. 设置LeapMotionController的HandPool的ModelsParent为HandModels，设置HandPool的Model Pool为2。

Leap使用示例代码
```C#
LeapProvider provider = FindObjectOfType\<LeapProvider\>() as LeapProvider;
Frame frame = provider.CurrentFrame;
foreach(Hand hand in frame.Hands) {
    if(hand.IsLeft)
    {
        transform.position = hand.PalmPosition.ToVector3() + hand.PalmNormal.ToVector3() * (transform.localScale.y * .5f + .02f);
        transform.rotation = hand.Basis.Rotation();
    }
}
```

## 检测器
使用DetectorLogicGate脚本来组合复杂的Detector，一个逻辑门可以包括任意数量的其他检测器作为输入并输出一个boolean，可以分发OnActivate和OnDeactive事件。




