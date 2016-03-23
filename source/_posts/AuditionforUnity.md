title: Unity面试中的一些常见问题
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

**十五、CharacterController和Rigbody的区别？**  
Rigbody具有完全真实的物理特性，而CharacterController可以说是受限的Rigidbody具有一定的物理特效但不是完全真实的。







 
