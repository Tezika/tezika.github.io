title: Unity中的一些常见的问题(忽视点)
date: 2016-03-20 14:28:49
tags:
- Unity
---
**一、什么是协同程序?**    
在主线程运行时同时开启另一段逻辑处理，来协助当前程序的执行。换句话说，开启协程就是开启一个可以与程序并行的逻辑。可以用来控制运动、序列以及对象的行为。  

**二、碰撞器和触发器的区别**    
碰撞器是触发器的载体，而触发器只是碰撞器身上的一个属性。   
物体属性`isTrigger = false`时候，碰撞器会触发物理碰撞，产生碰撞效果，可以调用`onCollisionEnter/Stay/Exit`函数处理相应的逻辑。  
当`isTrigger = true`,碰撞器被物理引擎所忽略，没有碰撞效果，可以调用`OnTriggerEnter/Stay/Exit`函数。  
  
**三、物体发生碰撞的必要条件**   
两个物体都必须带有碰撞器Collider，其中一个物体还必须带有Rigidbody刚体。  
  
**四、ArrayList和List<Type>的区别**   
ArrayList存在不安全的类型:   
- ArrayList会把插入其中的数据类型都当做Object处理。  
- 装箱和拆箱的操作。  
    
**五、请简述GC（垃圾回收）产生的原因，并描述如何避免？**    
原因：回收堆区的内存。  
避免：
1. 减少new的使用。  
2. 使用公共的对象(全局，静态)。  
3. String->StringBuilder。   

**六、反射的实现原理？**  
- 导入`using System.Reflection`
- `Assembly.Load("程序集");`//加载程序集,返回类型是一个Assembly，得到程序集中所有类的名称。
 
- ```c#
 foreach (Type type in assembly.GetTypes())
{
  string t = type.Name;
} 
```
- `Type type = assembly.GetType("程序集.类名");`//获取当前类的类型
- `Activator.CreateInstance(type); `//创建此类型实例
- `MethodInfo mInfo = type.GetMethod("方法名");`//获取当前方法  
- `mInfo.Invoke(null,方法参数)`  

**七、简述四元数Quaternion的作用，四元数对欧拉角的优点？**
四元数用于表示旋转

相对欧拉角的优点： 
- 避免万向锁[万向锁的解释](https://zh.wikipedia.org/wiki/%E7%92%B0%E6%9E%B6%E9%8E%96%E5%AE%9A)  
- 能进行增量旋转  
- 给定万向锁的表达方式有两种，互为负(欧拉角)  
  
**八、如何安全的在不同工程间安全地迁移asset数据？三种方法**   
1. 打包成packge
2. 将Assets和Library一起迁移。
3. unity自带的Assets Server.
  
**九、`Awake`,`OnEnable`,`Start`运行时顺序？哪些可能在一个周期中反复发生？**  
Awake->OnEnable->Start。  
OnEnable在同一周期中可以反复的发生。  
  
**十、MeshRender中material和sharedmaterial的区别**  
修改sharedmaterial将修改使用这个材质的外观，
并且也改变储存在工程里的材质设置。  

**十一、TCP/IP协议栈各个层次及分别的功能**  
- 网络接口层：这是协议栈的最低层，对应OSI的物理层和数据链路层，主要完成数据帧的实际发送和接受。
- 网络层：处理分组在网络中的活动，例如路由的选择和转发等，这一层主要把包括IP协议，ARP,ICMP协议等。
- 传输层：主要功能是提供应用程序之间的通信，这一层主要是TCP/UDP协议。
- 应用层：用来处理特定的应用，针对不同的应用提供不同的协议，例如进行文件传输时用到的FTP协议，发送emial用到的SMTP等。

**十二、Unity提供了几种广元，分别是什么？**  
四种，平行光，点光源，聚光灯和区域光源。  

**十三、简述一下对象池，你觉得FPS里哪些东西适合使用对象池？**
对象池就存放需要反复调用资源的一个空间，比如游戏中需要经常被大量复制的对象，子弹，敌人，以及任何重复出现的对象。

**十四、CharacterController和Rigbody的区别？**  
Rigbody具有完全真实的物理特性，而CharacterController可以说是受限的Rigidbody具有一定的物理特效但不是完全真实的。  
     
**十五、移动相机动作在哪个函数里，为什么在这个函数里？**
LateUpdat，是所有的Update结束后才调用，比较很适合命令脚本的执行。因为要等所有操作都完城后再进行摄像机的跟进，不然就可能能出现摄像机已经更近了，但是视角里还没有角色的空帧。  

**十六、简述prefab的用处**
prefab,官方翻译为预设。在游戏运行时进行实例化，prefab相当于一个模板，对你已有的蔬菜、脚本、参数做一个默认的配置，以便于以后的修改，同时prefab打包内容简化了导出操作，便于团队交流。  
  
**十七、sealed关键字在类申明和函数申明的作用**  
类申明的时候无法被继承，方法申明的时候可防止派生类重写此方法。  

**十八、请简述private，public，protected，internal的区别。**  
public:对任何类和成员都公开，无限制访问。  
private:仅对该类公开  
protected：对该类和其派生类公开  
internal:只能在包含该类的程序集中访问该类。  

**十九、什么是渲染管道**  
是指在显示器为了显示出图像而经过的一些列必要操作。渲染管道中的很多步骤，都要将几何物体从一个坐标系变换到另一个坐标系中去。

**二十、如何在Unity中优化内存**
1. 压缩自带类库。
2. 将暂时不用的以后还需要使用的物体隐藏起来而不是Destory掉。
3. 释放AssetBundle占用的资源。
4. 降低模型的片面数，降低模型的骨骼数量，降低贴图的大小。
5. 使用光照贴图，LOD,使用着色器shader,使用prefab。  
  
**二十一、动态添加资源的方法**
1. `Resources.Load(...);`
2. AssetBundle
  
**二十二、使用Unity实现2D游戏的方式？**
1.自带U2D.
2.使用2d插件，2DToolKIT,NGUI  
  
**二十三、unity物理引擎中，施加力的方式**
1. `rigidbody.AddForce(...)`  
2. `rigidbody.AddForceAtPosition(...)`
  
**二十四、什么叫做链条关节**
Hinge Joint，可以模拟两个物体间用一根链条连接在一起的情况，能保持两个物体在一个固定的距离内部相互移动而不产生作用力，但是达到固定距离后就会产生拉力。  
  
**二十五、Unity脚本的生命周期**    
Awake->OnEnable->Start->Update->FixedUpdate(一般放物理相关的)->LateUpdate(移动相机等)->OnGUI->Reset->OnDisable->OnDestroy


**二十六、物理更新一般放在哪个系统函数里面**  
FixedUpdate,FixedUpdate是渲染帧执行，渲染效率低下的时候FixedUpdate的调用次数会跟着下降。  
  
**二十七、请描述为什么Unity3d中会发生在组件上出现数据丢失的情况**  
一般是组件上绑定的物体对象被删除了。  

  
** 二十八、 请问下列代码运行中会产生多少个临时对象？**
```c#
string a = new string("abc");
a = (a.ToUpper() + "123").Substring(0, 2);
```

0个，因为在C#中第一行会出错，应该这样初始化
`string b = new string(new char[]{'a','b','c'});`  
  
**二十九、  下列代码在运行中会发生什么问题？该如何避免？**  
```c#
List<int> ls = new List<int>(new int[] { 1, 2, 3, 4, 5 });
foreach (int item in ls)
{
    Console.WriteLine(item * item);
    ls.Remove(item);
}
```  
会出现运行时的错误，因为foreach是只读的。
  
**三十、.net和Mono的关系**
mono是.net的一个开源的跨平台工具，类似于java的虚拟机。java本身不是跨平台的语言，但是跑在虚拟机上就可以跨平台。.net只能在windows下运行,mono却可以实现跨平台编译和运行，如Linux，Unix，Mac OS等。





 
  
  







 
