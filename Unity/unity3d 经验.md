# unity3d 经验

> 本文记录了使用、学习Unity3d过程中遇到的关键问题记录，目前采用的Unity5.5进行使用。会持续更新。
>
> winxos 2017-01-20

### gameObject查找

1. 手动拖放绑定

   此方法速度快，对于少量物体比较适用。

2. 用GameObject.Find()方式

   常用的方法，但是未激活物体无法找到。

3. 用transform.Find()方式

   推荐的做法，能够查找为激活物体，但是要求根目录物体必须激活。使用时先用GameObject.Find()找到根目录所在物体，然后利用物体的gameObject.transform.Find()进行查找。

> #### 注意：无论是GameObject还是transform方式，在Find时都是进行路径匹配，需要完整路径，并非包含子路径下的嵌套查找。

### 脚本间函数调用

推荐采用发送消息方式来做，格式如下：

```c#
gameObject.GetComponent<ChangeDisplay>().SendMessage("f1",params);
```

> 注意：脚本一定要绑定到gameObject上才进行了实例化，所以发送消息就是给脚本组件进行消息发送。

对于全局变量的实现，只需要声明成static就可以，但是要注意，gameObject声明成static是不可以手工绑定赋值的。

### UI多层叠加

存在多个页面时，在根目录建立ui的空gameObject，默认激活，如果有多个canvas，都放置到其下，多个界面切换激活属性来进行。

在单层canvas上时，如果有多层UI时，比如动画特效等，通过Pos Z可以进行前后层的设置，但是当特效层位于控件上层时，控件会无法响应，只需要将不需要参与交互的UI的Raycast Target属性设置为false，就不会对后面控件的事件进行拦截。





### 添加按键音效

1. 将声音文件添加到Assets中。
2. 在顶层gameObject绑定Audio source组件。
3. 将声音资源添加到Audio Sourced -> AudioClip。
4. 在按钮点击事件界面，拖入绑定Audio source的gameObject，函数选择AudioClip-> play()。
5. 注意：音效要进行波形观察，防止有前导静音，造成点击音效延迟。

