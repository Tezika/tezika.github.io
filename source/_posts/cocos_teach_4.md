title: Cocos小游戏《Rabbit Escape》教程(四) A星寻路算法的实现
date: 2015-02-27 21:44:41
tags:
- 开发教程
- cocos2d-x
---
# 一、前言
 &#160;&#160;&#160;&#160; 上一次我们创建游戏逻辑层，并且让它和UI层联系，让兔子能够实现移动。但是实现的移动都是简单的向上方向，这次的话我们要让兔子变“聪明”起来，让它能够沿着正确的方向来进行移动。（这次教程可能会长一点，因为涉及算法的讲解，童鞋们耐心的看下吧~~）。
 __________
# 二、A星算法的介绍和寻路算法的设计

 &#160;&#160;&#160;&#160;这里的话首先我们会用到游戏中经常用到的AI寻路算法的设计。实现AI的自动寻路，其实方法有很多，最无脑的就是暴力的搜索，但是实际中考虑到硬件条件和时间的复杂度应用的很少。这里使用的比较多的一种叫做A星的算法，A星算法其实一种对于经典的最短路径算法Dijkstra算法的一种改良和优化（如果不是很了解最短路径算法的童鞋可以搜索下相关资料来了解下），而对于A星算法，这里如果不了解的童鞋务必请先看下这位外国基佬的介绍（我觉得介绍的很通俗易懂），传送门：<http://blog.csdn.net/ronintao/article/details/17187239>。  
&#160;&#160;&#160;&#160;ok，如果你对A星算法有了了解，接下来我说下的这个寻路算法设计的基本思路吧，简而言之说的话就是，**“兔子没被围住的时候，采用A星算法寻路；兔子被围住的时候，采用最大通路算法”。**  
&#160;&#160;&#160;&#160;详细介绍一下，首先我们需重新的修改一下我们Pos结构体,加入相关和A星寻路中需要的属性值。
```c++
//地图位置
struct Pos{
    //cost代价
    int cost;
    //距离目标位置的的估计值
    int hEstimate;
    //A*中的F值
    int fVal;
    //是否是障碍;
    bool isObt;
    //位置信息
    Vec vec;
     
    Pos(){
        vec.ver = 0;
        vec.hor = 0;
        isObt = false;
        cost = 0;
        hEstimate = 0;
        fVal = VAL_MAX;
    }
    Pos(int _ver,int _hor){
        vec.ver = _ver;
        vec.hor = _hor;
        isObt = false;
        cost = 0;
        hEstimate = 0;
        fVal = VAL_MAX;
     
    }
 
};```

&#160;&#160;&#160;&#160;如果你真的理解了之前那个外国基佬帖子的话，这些属性值应该都知道意思了。  
 
&#160;&#160;&#160;&#160;其次我们需要，在GameCtrl类中加入我们相关A*算法和最大通路算法的需要的函数和变量。  

```c++
    /////////////A*////////////////
    //open集合
    vector<Pos *> m_openPos;
    //close
    vector<Pos *> m_closePos;
    //目标点
    Pos*  m_targetPos;
    //设置目标点
    void setTargetPos(const Dir& d);
    //A*寻路
    void AStart();
    //H估径函数
    int  funcHEstimate(Pos* p1, Pos* p2);
    //刷新周围的点
    void updateDirsFVal(Pos* newPos,int curCost);
    //重置地图F值
    void resetMapFVal();
    //获得当前的可遍历的方向
    void getCurDirs(vector<Vec*>& moveDirs,const Vec& curVec);
     
    /////////////最大通路////////////////
    //获得当前最大通路的方向
    Dir maxRoadMove();```
    
    
&#160;&#160;&#160;&#160;ok，我们来想一下，其实归咎到底，不管是经过任何一个寻路算法神马的，最终决定兔子怎么移动的是什么？没错，就是一个Dir(方向)，只要有一个方向我们就会知道兔子该怎么移动了。  
&#160;&#160;&#160;&#160;所以我在这里封装了一个得到兔子移动方向的函数：
```c++
//得到兔子的移动方向
     Dir getRabbitMoveDir();```

&#160;&#160;&#160;&#160;这样的话我们的兔子，就不是只能向上次一样只是向上移动了，决定它移动的方向就是getRabbitMoveDir函数得到的方向。  

&#160;&#160;&#160;&#160;因此最重要的变成了函数getRabbitMoveDir，是如何获取方向的呢？我先贴出这部分的代码，然后解释。

```c++
Dir GameCtrl::getRabbitMoveDir(){
    Dir minDir = Dir::none;
    int minFVal = VAL_MAX;
    //遍历可走方向,使用A*算法
    vector<Vec* > moveDirs;
    this->getCurDirs(moveDirs,m_rabbitPos);
    for(auto it:moveDirs){
        if(!m_gameMap[m_rabbitPos.ver+it->ver][m_rabbitPos.hor+it->hor]->isObt) {
            this->setTargetPos((Dir)m_moveDirs[it]);
            AStart();
            //如果小于最小价值
            if (minFVal>m_targetPos->fVal) {
                minFVal =m_targetPos->fVal;
                minDir  = static_cast<Dir>(m_moveDirs[it]);
            }
            this->resetMapFVal();
        }
         
    }
    //如果已经被围住,采用最大通路算法
    if (Dir::none==minDir) {
        minDir  = maxRoadMove();
    }
    return minDir;
}```
&#160;&#160;&#160;&#160;首先，不管在任何一个位置，兔子此时都有几个方向可以移动，首先要找出这几个方向（这里要说明的是并不是每个位置兔子都可以向8个方向可以移动，经过我的观察，兔子其实只有6个方向可以移动，这个和其所处位置行数的奇偶有关，关于这点童鞋们可以玩玩神经猫观察一下）。然后对这个几个方向首先进行A星寻路算法的搜索，看能不能找到一条到达边境点的最短路径（因为有可能兔子已经被围起来了），如果有则返回该路径的方向，如果没有就采用最大通路算法进行计算。  
**可以简化为:** 

  * 得到可移动方向。
  * 遍历每一个方向。 
     * A星计算每一个方向从兔子点到边境点的最短路径和方向，并进行比较记录。
     * 计算完一个方向后要初始化地图上每个位置的f值的信息。
  * 如果A星无法得到最短路径（已经被围住）,使用最大通路计算最短路径和方向。
  * 返回最短路径长。  
  
&#160;&#160;&#160;&#160;关于获取当前方向，是根据奇偶得到的，所以这里看看代码就懂了。
```c++
void GameCtrl::getCurDirs(vector<Vec*>& moveDirs,const Vec& curVec){
    if (curVec.ver!=0) {
        moveDirs.push_back(m_moveVec[(int)Dir::up]);
    }
    if (curVec.ver!=NUM_MAPROWANDCOW-1) {
         moveDirs.push_back(m_moveVec[(int)Dir::down]);
    }
    if (curVec.hor!=0) {
         moveDirs.push_back(m_moveVec[(int)Dir::left]);
        //如果当前行数为偶数，增加上左和下左方向
        if (curVec.ver%2==0){
            if(curVec.ver!=0){
                moveDirs.push_back(m_moveVec[(int)Dir::upLeft]);
            }
            if (curVec.ver!=NUM_MAPROWANDCOW-1) {
                moveDirs.push_back(m_moveVec[(int)Dir::downLeft]);
            }
        }
    }
    //如果当前为计数,增加上右和下右方向
    if (curVec.hor!=NUM_MAPROWANDCOW-1) {
         moveDirs.push_back(m_moveVec[(int)Dir::right]);
        if (curVec.ver%2!=0){
            if(curVec.ver!=0){
                moveDirs.push_back(m_moveVec[(int)Dir::upRight]);
            }
            if (curVec.ver!=NUM_MAPROWANDCOW-1) {
                moveDirs.push_back(m_moveVec[(int)Dir::downRight]);
            }
        }
 
    }
     
}```

&#160;&#160;&#160;&#160;其实，看到这里很多童鞋可能会问，A星算法不是只能计算一个点到另外一个点之间的最短路径吗？这里我们要计算的是兔子点到一条边境的最短路径啊？  
&#160;&#160;&#160;&#160;哈哈哈，其实这里为了简单，运用了边境点的这一概念。
![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_249caa7a58fdf80.png)  

&#160;&#160;&#160;&#160;图画的有点丑，讲究看下吧。这里我为了简单换成5X5的，然后假设兔子在三角形(3,1)位置，然后先画出了4个基本方向边境上的边境点（其实就可以理解为是兔子沿着该方向到达边境最近的点），这点很好理解。

&#160;&#160;&#160;&#160;接下来因为兔子当前位置在基数行，所以另外两个方向为右上和右下。此时就要先进行判断到底是到达哪个边境最优，才能得到出边境点位置了。

![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_a248df0ecf8f008.png)  

&#160;&#160;&#160;&#160;其实很多童鞋，会疑问你是肿么算的，看看代码你就会知道了。

```c++
void GameCtrl::setTargetPos(const Dir& d){
    int size = NUM_MAPROWANDCOW-1;
    switch(d){
        case  Dir::up:
            m_targetPos = m_gameMap[0][m_rabbitPos.hor];
            break;
        case Dir::down:
            m_targetPos = m_gameMap[size][m_rabbitPos.hor];
            break;
        case Dir::left:
            m_targetPos = m_gameMap[m_rabbitPos.ver][0];
            break;
        case Dir::right:
            m_targetPos = m_gameMap[m_rabbitPos.ver][size];
            break;
        case Dir::upLeft:
            m_targetPos = m_rabbitPos.ver<m_rabbitPos.hor?m_gameMap[0][m_rabbitPos.hor-m_rabbitPos.ver]:m_gameMap[m_rabbitPos.ver-m_rabbitPos.hor][0];
            break;
        case Dir::downLeft:
            m_targetPos = (size-m_rabbitPos.ver)<m_rabbitPos.hor?m_gameMap[size][m_rabbitPos.hor+m_rabbitPos.ver-size]:m_gameMap[m_rabbitPos.ver+m_rabbitPos.hor][0];
            break;
        case Dir::upRight:
            m_targetPos = m_rabbitPos.ver<(size-m_rabbitPos.hor)?m_gameMap[0][m_rabbitPos.ver+m_rabbitPos.hor]:m_gameMap[m_rabbitPos.ver+m_rabbitPos.hor-size][size];
            break;
        case Dir::downRight:
            m_targetPos = (size-m_rabbitPos.ver)<(size-m_rabbitPos.hor)?m_gameMap[size][m_rabbitPos.hor+size-m_rabbitPos.ver]:m_gameMap[m_rabbitPos.ver+size-m_rabbitPos.hor][size];
            break;
        default:
            break;
    }
}```

&#160;&#160;&#160;&#160;先比较沿着该方向的两个基本方向到达各自边境的距离比较，确定到达那条边境，再计算边境点的位置。  
&#160;&#160;&#160;&#160;剩下的两个方向，有兴趣的童鞋也可以算算。其实这部分主要是利用到了一个边境点替换边境进行”偷懒“，这里其实还可以每次在遍历的时候遍历边境上的每一个点和目标点之间的最短路径，麻烦是麻烦了点，但是实际上是得到的结果会更准确。  
&#160;&#160;&#160;&#160;好了剩下的就是A*部分的代码了，这部分其实如果理解代码了思想就很好写出来了，直接上代码。

```c++
void GameCtrl::AStart(){
    //清空上次的数据
    m_openPos.clear();
    m_closePos.clear();
     
    //将兔子的位置放进open
    m_openPos.push_back(m_gameMap[m_rabbitPos.ver][m_rabbitPos.hor]);
     
    //初始化当前的花费
    int curCost = 0;
     
    while (!m_openPos.empty()) {
        //找出open中最小的S
        Pos* minPos = NULL;
        int minFval = VAL_MAX;
        int minIndex =  0;
        for (int i =0 ; i<m_openPos.size(); i++) {
            if (minFval>=m_openPos.at(i)->fVal) {
                minFval = m_openPos.at(i)->fVal;
                minPos = m_openPos.at(i);
                minIndex  = i;
            }
        }
        //open中剔除
        m_openPos.erase(m_openPos.begin()+minIndex);
 
        //加入close
        m_closePos.push_back(minPos);
         
        if (NULL!=minPos){
        //如果找到目标就退出
          if(m_targetPos==minPos) {
                break;
             }
            curCost++;
            vector<Vec*> moveDirs;
            this->getCurDirs(moveDirs,minPos->vec);
            for (auto it: moveDirs) {
                this->updateDirsFVal(m_gameMap[minPos->vec.ver+it->ver][minPos->vec.hor+it->hor],curCost);
             }
            }
        }
}
 
void GameCtrl::updateDirsFVal(Pos* newPos,int curCost){
//如果该点不是障碍
  if(!newPos->isStone){
    //如果该店不在close中
      if (std::find(m_closePos.begin(),m_closePos.end(), newPos)==m_closePos.end()) {
        //如果该点也不再open中
              if (std::find(m_openPos.begin(),m_openPos.end(), newPos)==m_openPos.end()) {
                newPos->cost=curCost;
                //计算H姑凉值
                newPos->hEstimate = this->funcHEstimate(newPos,m_targetPos);
                newPos->fVal = newPos->cost+newPos->hEstimate;
                //放进open
                m_openPos.push_back(newPos);
              }
            //如果在则刷新
              else{
                  int tfVal  = curCost+this->funcHEstimate(newPos,m_targetPos);
                  if (newPos->fVal>tfVal) {
                    newPos->cost= curCost;
                    //计算H姑凉值
                    newPos->hEstimate = this->funcHEstimate(newPos,m_targetPos);
                    newPos->fVal = newPos->cost+newPos->hEstimate;
                     
                  }
            }
          }
      }
}
 
int GameCtrl::funcHEstimate(Pos* p1,Pos* p2){
    return abs(p1->vec.ver-p2->vec.ver)+abs(p1->vec.hor-p2->vec.hor);
}
 
void GameCtrl::resetMapFVal(){
    for (int i = 0; i<NUM_MAPROWANDCOW; i++) {
        for (int j = 0; j<NUM_MAPROWANDCOW; j++) {
            m_gameMap[j]->fVal = VAL_MAX;
        }
    }
 
}```
&#160;&#160;&#160;&#160;最后关于被围住后的最大通路算法，其实很好理解。
```c++
Dir GameCtrl::maxRoadMove(){
    //定义规格
    int size = NUM_MAPROWANDCOW-1;
    //记录最长距离
    int maxL =  0;
    //记录返回的方向
    Dir maxDir = Dir::none;
    vector<Vec*> moveDirs;
    this->getCurDirs(moveDirs,m_rabbitPos);
    for (auto it:moveDirs) {
        int curL = 0;
        Vec curVec = m_rabbitPos+(*it);
        //如果当前不是石头并且不是边界
        while (!m_gameMap[curVec.ver][curVec.hor]->isObt&&curVec.ver>0&&curVec.ver<size&&curVec.hor>0&&curVec.hor<size){
            curL++;
            curVec = curVec +(*it);
        }
        if (maxL<curL) {
            maxL = curL;
            maxDir = (Dir)m_moveDirs[it];
        }
    }
    return maxDir;
}```
&#160;&#160;&#160;&#160;想想如果被围住了，怎么办呢？当然是做垂死的挣扎，朝着没有障碍最多的一个方向移动呀！上面的代码就是这个意思。（注意这里因为我采用的是边界点替代边界，可能在被围住判断过程中会出现一定的误差，大牛们可以帮我看下这个BUG)。  
&#160;&#160;&#160;&#160;最后我们再修改一下HelloWorld中的相关部分，可以添加一个随机产生障碍的函数玩玩。  
```c++
void HelloWorld::setRandStones(int num){
    int curNum = 0;
    //随机产生障碍
    while (num!=curNum) {
        int ver,hor,tag;
        ver = 1+rand_0_1()*(NUM_MAPROWANDCOW-2);
        hor = 1+rand_0_1()*(NUM_MAPROWANDCOW-2);
        tag = ver*NUM_MAPROWANDCOW+hor;
        if(!m_pNodes->getChildByTag(tag)->isVisible()&&!(ver==NUM_MAPROWANDCOW/2&&hor==NUM_MAPROWANDCOW/2)){
            m_pNodes->getChildByTag(tag)->setVisible(true);
            m_ctrl->addObtacle(tag);
            curNum++;
        }
        else{
            continue;
        }
  
    }
}```
&#160;&#160;&#160;&#160;现在编译运行就可以看见，当你添加障碍的时候，兔子好像真的变得很”聪明“，一股脑的往边界跑，但是还没有胜利和失败的判断，下次我们会对游戏进行最后的完善。  
___________________
# 三、最后说几句
  * **.这次教程难度可能会有点大，因为涉及算法的讲解，其实这里我想说，如果我们作为开发者真的想要有所提高，算法和数据结构是根本的。所以希望大家能够不要光是会用引擎，调调API神马的。更要学习一下经典的一些算法，丰富自己游戏开发的基础。** 
  * **本次教程的资源传送门：<http://pan.baidu.com/s/1o6mh5ZO>,密码：70gc。**
  * **如果教程出现语法，拼字错误还请见谅~~~**
   __________
