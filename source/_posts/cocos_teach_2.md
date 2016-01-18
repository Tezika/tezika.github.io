title: Cocos小游戏《Rabbit Escape》教程(二) 游戏界面和障碍物
date: 2015-02-12 21:44:41
tags:
- 开发教程
- cocos2d-x
---
# 一、前言
 &#160;&#160;&#160;&#160;因为最近在学点东西，忘记更新该教程了。争取能在春节前更完吧。今天主要讲讲cocos2d-x与cocosstudio如何联系起来，以及实现游戏中添加障碍物。废话不多说，我们开始吧！       
  
  
                    
__________
# 二、cocos2d-x和cocosstudio的如何联系起来搭建游戏界面
  &#160;&#160;&#160;&#160;首先建好一个cocos2d-x新项目，用cocosstudio2打开附件中的Rabbit资源文件Rabbit.ccs，UI界面我早已给大家设计好。
  ![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_42eec0c9335cc00.png)  
  &#160;&#160;&#160;&#160;主要关注一下，节点树结构。因为会和等下的程序中的调用有很大的联系。然后将文件夹res的所有资源添加到新项目的Resources中(是Resources,而不是其中的res,这样操作主要是考虑到不用设置路径)。然后，来撸代码 !  
&#160;&#160;&#160;&#160;修改后的HelloWorld.h
```c++
#ifndef __HELLOWORLD_SCENE_H__
#define __HELLOWORLD_SCENE_H__
#include "cocos2d.h"
 
class HelloWorld : public cocos2d::Layer
{
public:
    // there's no 'id' in cpp, so we recommend returning the class instance pointer
    static cocos2d::Scene* createScene();
 
    // Here's a difference. Method 'init' in cocos2d-x returns bool, instead of returning 'id' in cocos2d-iphone
    virtual bool init();
     
    // a selector callback
    void menuCloseCallback(cocos2d::Ref* pSender);
     
    // implement the "static create()" method manually
    CREATE_FUNC(HelloWorld);
public:
    bool onTouchBeganCallBack(cocos2d::Touch* t,cocos2d::Event* event);
private:
    //存放所有子节点
    Node* m_pNodes;
     
    //存放兔子的节点
    Node* m_pNode_rabbit;
     
    //存放障碍的节点的容器
    cocos2d::Vector<Node* > m_pNodes_obstacle;
};
#endif // __HELLOWORLD_SCENE_H__
```
&#160;&#160;&#160;&#160;HelloWorld.cpp，这里我只截取了重要的init函数：  
```c++
// on "init" you need to initialize your instance
bool HelloWorld::init()
{
    //////////////////////////////
    // 1. super init first
    if ( !Layer::init() )
    {
        return false;
    }
    //获取和添加场景节点
    auto pNode_scene = CSLoader::createNode("MainScene.csb");
    this->addChild(pNode_scene);
     
     
    //获取所有子节点
    m_pNodes= pNode_scene->getChildByName("Sprite_bg");
     
    //分别的到兔子和障碍节点
    for (auto it = m_pNodes->getChildren().begin(); it!=m_pNodes->getChildren().end(); it++) {
        if ((*it)->getName()=="Sprite_Rabbit") {
            m_pNode_rabbit = (*it);
        }
         
        else{
            m_pNodes_obstacle.pushBack((*it));
        }
         
    }
     
    return true;
}
```
&#160;&#160;&#160;&#160;首先如果要进行cocosstudio和cocos2d-x的联系的话，首先要添加头文件
```c++
#include "editor-support/cocostudio/CocoStudio.h"
#include "ui/CocosGUI.h"
```
&#160;&#160;&#160;&#160;然后可以使用CSLoader工具中的create方法中节点的Node* 获取.csb文件中的场景节点。
```c++
//获取和添加场景节点
    auto pNode_scene = CSLoader::createNode("MainScene.csb");
    this->addChild(pNode_scene); ```
  
&#160;&#160;&#160;&#160;我们可以看看cocosstudio中的节点状态树就可以理解接下来的操作，  
![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_046f71b7df79e40.png)
  
&#160;&#160;&#160;&#160;从父子关系我们可以看出：  **scene节点下有bg节点，而bg节点下则是其他的所有节点。**    

&#160;&#160;&#160;&#160;我们分别获取其中的兔子节点和障碍节点方便后面逻辑层对它们的操作:  
```c++
//获取所有子节点
    m_pNodes= pNode_scene->getChildByName("Sprite_bg");
     
    //分别的到兔子和障碍节点
    for (auto it = m_pNodes->getChildren().begin(); it!=m_pNodes->getChildren().end(); it++) {
        if ((*it)->getName()=="Sprite_Rabbit") {
            m_pNode_rabbit = (*it);
        }
         
        else{
            m_pNodes_obstacle.pushBack((*it));
        }
         
    }

```
&#160;&#160;&#160;&#160; 其实更多关于两者的联系，大家可以看下本站关于cocostduio的教程,链接:<http://www.cocoachina.com/bbs/read.php?tid-265757.html>以及更好的就是多去阅读官方给的例子的源码。  

&#160;&#160;&#160;&#160;最后，我们还需要调整一下设计的分辨率，因为本人用的是5s，所以是按照它的分辨率来设计的。这部分我直接贴代码.如果不懂的，找些教程来看吧:  
```c++
//屏幕缩放因子
const int CONFIG_SCALEFACTOR = 2;
 
//实际资源规格
const int RESOURCE_BG_WIDTH = 640;
const int RESOURCE_BG_HEIGHT = 1137;
 
bool AppDelegate::applicationDidFinishLaunching() {
    // initialize director
    auto director = Director::getInstance();
    auto glview = director->getOpenGLView();
    if(!glview) {
        glview = GLViewImpl::create("My Game");
        glview->setFrameSize(RESOURCE_BG_WIDTH/CONFIG_SCALEFACTOR, RESOURCE_BG_HEIGHT/CONFIG_SCALEFACTOR);
        director->setOpenGLView(glview);
        glview->setDesignResolutionSize(RESOURCE_BG_WIDTH, RESOURCE_BG_HEIGHT, ResolutionPolicy::FIXED_HEIGHT);
    }
 
    // turn on display FPS
    director->setDisplayStats(true);
 
    // set FPS. the default value is 1.0/60 if you don't call this
    director->setAnimationInterval(1.0 / 60);
 
    // create a scene. it's an autorelease object
    auto scene = HelloWorld::createScene();
 
    // run
    director->runWithScene(scene);
 
    return true;
}
```
 &#160;&#160;&#160;&#160;此时，再编译运行程序就可以看到:  
 ![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_42a35dc4b3e8e8c.png)  
 
 &#160;&#160;&#160;&#160;游戏界面已经搭建好了！！！
  __________
# 三、如何添加实现添加障碍物
&#160;&#160;&#160;&#160;界面搭好了过后，其实如何实现点击障碍物实现添加呢？其实也很简单了，首先我们写一个触摸事件回调函数。 
```c++
public:
    bool onTouchBeganCallBack(cocos2d::Touch* t,cocos2d::Event* event);
```
  
```c++
   bool HelloWorld::onTouchBeganCallBack(Touch *t, Event* event){
    for (auto it =m_pNodes_obstacle.begin();it!=m_pNodes_obstacle.end() ; it++) {
        if((*it)->getBoundingBox().containsPoint(t->getLocation())&&(!(*it)->isVisible())){
            //add obt
            auto pObt = (*it);
            pObt->setVisible(true);
            break;
    }
   }
     return false;
}
```
&#160;&#160;&#160;&#160;最后再在init方法中添加触摸事件的监听，代码如下：
```c++
//添加监听
    auto listener = EventListenerTouchOneByOne::create();
    listener->onTouchBegan = CC_CALLBACK_2(HelloWorld::onTouchBeganCallBack,this);
    Director::getInstance()->getEventDispatcher()->addEventListenerWithSceneGraphPriority(listener, this);
```

&#160;&#160;&#160;&#160;**这里利用的原理简而言之就是之前我们在cocosstudio中把所有障碍物设置为不可见，然后遍历当前的触摸点在哪个障碍物的范围内，就将其可见性设置为可见。**完成之后，再次编译运行我们就可以自由的添加障碍了。  

______

# 四、最后说两句
* **此次上传我上传的文件和上次差不多主要是这次的源代码和cocosstudio资源文件夹，使用方法请见上次教程。** 

* **附件地址：<http://pan.baidu.com/s/1mgGD0U0>, 密码：tec0**

