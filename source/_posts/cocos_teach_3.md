title: Cocos小游戏《Rabbit Escape》教程(三) 让兔子动起来
date: 2015-02-26 21:44:41
tags:
- 开发教程
- cocos2d-x
---
# 一、前言
 &#160;&#160;&#160;&#160; 首先预祝大家新年快乐！最近都在忙着过年，因为回老家了，教程很久没更新了。首先感谢cocoschina社区对我教程的支持（今天意外的在中文引擎网上看到了自己的教程），以后我会把我学习cocos的过程做的一些demo项目都开源做成一些教程和大家一起学习。  
 &#160;&#160;&#160;&#160;经过上一讲的讲解，我们已经可以将UI响应用户的交互操作了(添加石头)，但是这都只是在UI界面上的一些操作，今天我们会讲一讲如何创建逻辑层，并将UI层和游戏逻辑层进行联系，以及实现简单的让兔子动起来。          
__________
# 二、创建GameMap
  &#160;&#160;&#160;&#160;首先在我们项目中要新建一个名叫GameCtrl的类，来管理游戏的逻辑。  
  .h文件:  
  ```c++
  #ifndef __RabbitTech__GameCtrl__
#define __RabbitTech__GameCtrl__
 
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;
 
const int NUM_MAPROWANDCOW  = 9;       //表示行列数     
const int VAL_MAX =  0x0FFFFFFF;              //表示最大值
 
//方向枚举
enum class Dir{
    up,
    down,
    left,
    right,
    upLeft,
    upRight,
    downLeft,
    downRight,
    none
};
 
//地图矢量
struct Vec{
    int hor; //横向
    int ver; //纵向
    Vec(){
        hor =  0;
        ver = 0;
    }
    Vec(int _ver,int _hor){
        hor = _hor;
        ver =  _ver;
    }
    bool operator == (const Vec& vec ){
        return vec.hor==hor&&vec.ver==ver;
    }
     
    Vec operator + (const Vec& vec){
        return Vec(vec.ver+ver,vec.hor+hor);
    }
     
};
 
//地图位置
struct Pos{
    //是否是障碍;
    bool isObt;
    //位置信息
    Vec vec;
     
    Pos(){
        vec.ver = 0;
        vec.hor = 0;
        isObt = false;
    }
    Pos(int _ver,int _hor){
        vec.ver = _ver;
        vec.hor = _hor;
        isObt = false;
    }
 
};
class GameCtrl{
public:
    GameCtrl();
    ~GameCtrl();
public:
    //添加石头
    void addObtacle(int tag);
    //移动兔子
    int rabbitMove();
private:
    //游戏地图
    vector<vector<Pos *>>m_gameMap;
    //移动距离
    std::unordered_map<int,Vec*> m_moveVec;
    std::unordered_map<Vec*,int> m_moveDirs;
    //初始化地图移动矢量
    void initMoveVec();
    //初始花地图信息
    void initMapInfo();
 
   Vec m_rabbitPos; //兔子的位置
};
 
 
#endif /* defined(__RabbitTech__GameCtrl__) */
  ```
  .cpp文件:
  ```c++
  #include "GameCtrl.h"
GameCtrl::GameCtrl(){
    this->initMoveVec();
    this->initMapInfo();
}

void GameCtrl::initMoveVec(){
    //创建移动HASH
    m_moveVec[(int)Dir::up] = new Vec(-1,0);
    m_moveVec[(int)Dir::down] = new Vec(1,0);
    m_moveVec[(int)Dir::left] = new Vec(0,-1);
    m_moveVec[(int)Dir::right] = new Vec(0,1);
    m_moveVec[(int)Dir::upLeft] = new Vec(-1,-1);
    m_moveVec[(int)Dir::upRight] = new Vec(-1,1);
    m_moveVec[(int)Dir::downLeft] = new Vec(1,-1);
    m_moveVec[(int)Dir::downRight]= new Vec(1,1);
    
    //创建映射hash
    m_moveDirs[m_moveVec[(int)Dir::up]]=(int)Dir::up;
    m_moveDirs[m_moveVec[(int)Dir::down]]=(int)Dir::down;
    m_moveDirs[m_moveVec[(int)Dir::left]]=(int)Dir::left;
    m_moveDirs[m_moveVec[(int)Dir::right]]=(int)Dir::right;
    m_moveDirs[m_moveVec[(int)Dir::upLeft]]=(int)Dir::upLeft;
    m_moveDirs[m_moveVec[(int)Dir::upRight]]=(int)Dir::upRight;
    m_moveDirs[m_moveVec[(int)Dir::downLeft]]=(int)Dir::downLeft;
    m_moveDirs[m_moveVec[(int)Dir::downRight]]=(int)Dir::downRight;
}
void GameCtrl::initMapInfo(){
    
    //初始化地图
    for (int i = 0; i<NUM_MAPROWANDCOW; i++) {
        vector<Pos *> curRow;
        for (int j = 0; j<NUM_MAPROWANDCOW; j++) {
            auto pNewPos = new Pos(i,j);
            curRow.push_back(pNewPos);
            
        }
        m_gameMap.push_back(curRow);
    }
    
    //初始化兔子信息
    m_rabbitPos.hor=m_rabbitPos.ver = NUM_MAPROWANDCOW/2;
}

void GameCtrl::addObtacle(int tag){
    m_gameMap[tag/NUM_MAPROWANDCOW][tag%NUM_MAPROWANDCOW]->isObt = true;
}

int GameCtrl::rabbitMove() {
    //假设让兔子一直向上
    m_rabbitPos=(*m_moveVec[(int)Dir::up])+m_rabbitPos;
    return m_rabbitPos.ver*NUM_MAPROWANDCOW+m_rabbitPos.hor;
}

GameCtrl::~GameCtrl(){
    //析构地图信息
    for (int i  = 0; i<NUM_MAPROWANDCOW; i++) {
        for (int j = 0; j<NUM_MAPROWANDCOW; j++) {
            delete m_gameMap[i][j];
        }
    }
    
    //析构移动矢量
    for (auto& it:m_moveVec) {
        delete it.second;
    }

    
}
  ```
   &#160;&#160;&#160;&#160;**PS:因为整个逻辑控制是脱离cocos2d-x架构的，为了降低耦合，我没有使用cocos2d-x的头文件。**  
   
&#160;&#160;&#160;&#160;玩过《围住神经猫》的童鞋应该都知道，它的地图看起来不是很对称，但是其实可以抽象为一个9X9的矩阵（后面称为GameMap）。所以该GameMap一共有9*9=81个位置，所以为了表示每一个位置的信息，我抽象出了 一个叫Vec(地图矢量)的结构来表示。  

```c++
//地图矢量
struct Vec{
    int hor; //横向
    int ver; //纵向
    Vec(){
        hor =  0;
        ver = 0;
    }
    Vec(int _ver,int _hor){
        hor = _hor;
        ver =  _ver;
    }
    bool operator == (const Vec& vec ){
        return vec.hor==hor&&vec.ver==ver;
    }
     
    Vec operator + (const Vec& vec){
        return Vec(vec.ver+ver,vec.hor+hor);
    }
     
};
  ```
  &#160;&#160;&#160;&#160;为了在后面的操作方面我重写了它的两个运算符。  
  
  &#160;&#160;&#160;&#160;而为了进行后面能够存储表示每个位置在A*寻路算法中的各种信息（下次教程会具体介绍），我将它进行了再次的封装。  
 
  ```c++
    //地图位置
struct Pos{
    //是否是障碍;
    bool isObt;
    //位置信息
    Vec vec;
     
    Pos(){
        vec.ver = 0;
        vec.hor = 0;
        isObt = false;
    }
    Pos(int _ver,int _hor){
        vec.ver = _ver;
        vec.hor = _hor;
        isObt = false;
    }
 
};
  ```
  
  &#160;&#160;&#160;&#160;OK，现在可以创建我们的游戏地图了，这里我用了一个的vector来模拟二维的地图。
    
  ```c++
       //游戏地图
    vector<vector<Pos *>>m_gameMap;
    //初始化地图
    for (int i = 0; i<NUM_MAPROWANDCOW; i++) {
        vector<Pos *> curRow;
        for (int j = 0; j<NUM_MAPROWANDCOW; j++) {
            auto pNewPos = new Pos(i,j);
            curRow.push_back(pNewPos);
             
        }
        m_gameMap.push_back(curRow);
  } ```
   &#160;&#160;&#160;&#160;如果对vector如何模拟二维动态数组的童鞋，可以百度一下相关内容进行了解。  
  __________
#三、兔子在逻辑层的移动
&#160;&#160;&#160;&#160;有些童鞋肯定以为神经猫中的猫一共只有四个移动方向，但是其实事实上仔细观察的话就会发现，其实是有8个方向可以移动的。所以我做了一个枚举类来表示其移动的方向。
```c++
//方向枚举
enum class Dir{
    up,
    down,
    left,
    right,
    upLeft,
    upRight,
    downLeft,
    downRight,
    none
};```
  
&#160;&#160;&#160;&#160;有了方向枚举，怎么能够和地图矢量联系起来呢？  

![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_a915c0de33256ca.png)  

&#160;&#160;&#160;&#160;相信如果有点数学基础的童鞋，都应该看得懂这张图。这就是用二维的数组来模拟平面的点的情况。如，当前rabbit的Vec(假设为rabbitVec)的话，在rabbit上面一行的up位置的Vec应该为(rabbitVec.ver-1,rabbitVec.hor)。我们要用个成员变量记录rabbit位置并且进行一开始初始化:
```c++
   Vec m_rabbitPos; //兔子的位置
  //初始化兔子信息
   m_rabbitPos.hor=m_rabbitPos.ver = NUM_MAPROWANDCOW/2;```
   
 &#160;&#160;&#160;&#160;一开始将其放在（4，4）的地方，即正中间。  
 
 &#160;&#160;&#160;&#160;为了以后进行位置变化方便，我这里用了两个hash map来保存移动的信息。  
 
```c++
 //移动距离
    std::unordered_map<int,Vec*> m_moveVec;
    std::unordered_map<Vec*,int> m_moveDirs;
 

void GameCtrl::initMoveVec(){
    //创建移动HASH
    m_moveVec[(int)Dir::up] = new Vec(-1,0);
    m_moveVec[(int)Dir::down] = new Vec(1,0);
    m_moveVec[(int)Dir::left] = new Vec(0,-1);
    m_moveVec[(int)Dir::right] = new Vec(0,1);
    m_moveVec[(int)Dir::upLeft] = new Vec(-1,-1);
    m_moveVec[(int)Dir::upRight] = new Vec(-1,1);
    m_moveVec[(int)Dir::downLeft] = new Vec(1,-1);
    m_moveVec[(int)Dir::downRight]= new Vec(1,1);
     
    //创建映射hash
    m_moveDirs[m_moveVec[(int)Dir::up]]=(int)Dir::up;
    m_moveDirs[m_moveVec[(int)Dir::down]]=(int)Dir::down;
    m_moveDirs[m_moveVec[(int)Dir::left]]=(int)Dir::left;
    m_moveDirs[m_moveVec[(int)Dir::right]]=(int)Dir::right;
    m_moveDirs[m_moveVec[(int)Dir::upLeft]]=(int)Dir::upLeft;
    m_moveDirs[m_moveVec[(int)Dir::upRight]]=(int)Dir::upRight;
    m_moveDirs[m_moveVec[(int)Dir::downLeft]]=(int)Dir::downLeft;
    m_moveDirs[m_moveVec[(int)Dir::downRight]]=(int)Dir::downRight;
}```
&#160;&#160;&#160;&#160;一个用于存储移动的Vec，一个用于进行映射（后面会有用到）。这样操作的话会大大减少代码的复杂度。
______
#四、UI层和逻辑层联系起来
&#160;&#160;&#160;&#160;  讲了肿么多，你可能会问了，你怎么能将你逻辑层的障碍，兔子和你表现在UI上面的兔子和障碍联系起来了呢？哈哈哈，这里我主要用到了对逻辑标签tag进行数学中的除和取余操作实现的，看了下面这张图也许你就会懂了。

![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_c1b5a79528acf92.png)

&#160;&#160;&#160;&#160; 有了上面的图解释，关于添加障碍的代码，也应该可以理解了。 
```c++
void GameCtrl::addObtacle(int tag){
    m_gameMap[tag/NUM_MAPROWANDCOW][tag%NUM_MAPROWANDCOW]->isObt = true;
}```
&#160;&#160;&#160;&#160; 而关于UI层的兔子移动是根据逻辑层的兔子移动后Vec中的ver,hor来反推出tag，然后在UI中找到该tag对应的sprite,将兔子移动到相应的sprite的position。  
```c++
int GameCtrl::rabbitMove() {
    //假设让兔子一直向上
    m_rabbitPos=(*m_moveVec[(int)Dir::up])+m_rabbitPos;
    return m_rabbitPos.ver*NUM_MAPROWANDCOW+m_rabbitPos.hor;
}
    //rabbit move
    int moveTag = m_ctrl->rabbitMove();
    if(m_pNodes->getChildByTag(moveTag)){
       m_pNode_rabbit->runAction(MoveTo::create(0.1, m_pNodes->getChildByTag(moveTag)->getPosition()));
}```
&#160;&#160;&#160;&#160;这里我们先假设兔子是只能向上移动，下次我会讲解如何根据兔子能够沿着正确的方向进行移动。  

&#160;&#160;&#160;&#160;最后需要修改一下在HelloWorld中添加我们的控制类GameCtrl,以及修改触摸响应函数。
```c++
private:
    //游戏逻辑控制
    GameCtrl* m_ctrl;
    
bool HelloWorld::onTouchBeganCallBack(Touch *t, Event* event){
    for (auto it =m_pNodes_obstacle.begin();it!=m_pNodes_obstacle.end() ; it++) {
        if((*it)->getBoundingBox().containsPoint(t->getLocation())&&(!(*it)->isVisible())){
            //add obt
            auto pObt = (*it);
            pObt->setVisible(true);
            m_ctrl->addObtacle(pObt->getTag());
             
            //rabbit move
            int moveTag = m_ctrl->rabbitMove();
            if(m_pNodes->getChildByTag(moveTag)){
                m_pNode_rabbit->runAction(MoveTo::create(0.1, m_pNodes->getChildByTag(moveTag)->getPosition()));
            }
             
            break;
     }
   }
     return false;
}```
&#160;&#160;&#160;&#160;别忘了对m_ctrl进行内存管理哦，因为不参与cocos的内存管理，所以需要自己对其进行内存管理。
__________
#五、最后说两句
* **因为电脑重装系统了，所以这次上传的东西是用v3.4和cocos studio 2.1版本。所以如果有的童鞋还用的是3.3版本的话，可能编译的时候cocosstudio的资源没办法加载。这种情况的话，将ccs用老版的cocosstudio打开后重新发布下资源，就可以使用了。** 
* **本教程仅供交流和学习，切勿用于商业开发，转载请注明出处。如果有疑问和更好建议的童鞋可以给我留言或者微博(@Tezika)私信我，我也会在最快做出回复。**
* **附件地址：<http://pan.baidu.com/s/1jG3Q830>, 密码：l90h**

