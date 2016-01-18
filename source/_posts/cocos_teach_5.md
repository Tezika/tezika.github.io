title: Cocos小游戏《Rabbit Escape》教程(五) 完成游戏
date: 2015-02-27 21:44:41
tags:
- 开发教程
- cocos2d-x
---
# 一、前言
 &#160;&#160;&#160;&#160;上一次,我们已经解决了游戏中的核心问题如何实现兔子的寻路移动。但是我们还没有添加游戏的成功失败判断和积分的管理，这次教程我们来完善它。
 __________
# 二、GameCtrl类中添加游戏的成功和失败判定，积分管理。
&#160;&#160;&#160;&#160;关于游戏的成功和失败判定，首先我们需要在头文件中添加相应的成员函数。  
```c++
public:
    //判断成功
    bool judgeSuccess();
    //判断失败
    bool judgeFail();```

&#160;&#160;&#160;&#160;关于游戏成功，玩过神经猫的童鞋都应该知道，只要将猫围住就算赢了，同比我们的游戏，只要将兔子围住就可以了。
```c++
bool GameCtrl::judgeSuccess(){
    //获取当前可遍历的方向
    vector<Vec*> moveDirs;
    this->getCurDirs(moveDirs,m_rabbitPos);
    for (auto it:moveDirs) {
        if (!m_gameMap[m_rabbitPos.ver+it->ver][m_rabbitPos.hor+it->hor]->isObt){
            return false;
        }
    }
    return true;
}```
&#160;&#160;&#160;&#160;实现起来的话，遍历当前兔子可以移动到的位置是不是都是障碍就行了，如果都是说明成功了，否则则没有成功。

&#160;&#160;&#160;&#160;关于游戏失败，只要当兔子此时的位置只要到达任意一条边界边上，则就说明兔子成功的逃脱了，游戏也失败了。
```c++
bool GameCtrl::judgeFail(){
    if (m_rabbitPos.hor==0||m_rabbitPos.ver==0||m_rabbitPos.hor==NUM_MAPROWANDCOW-1||m_rabbitPos.ver==NUM_MAPROWANDCOW-1) {
        return  true;
    }
    return  false;
}```
&#160;&#160;&#160;&#160;ok,我们已经添加了完了成功和失败的判定，现在我们来添加积分的管理。分析一下，在这个游戏中积分是什么，其实就是兔子移动的次数(step)。如果你能在兔子移动越少的情况下将兔子围住，当然就说明你越牛X。  

在头文件中添加响应的成员变量和接口函数。    
```c++
//当前移动步数
    int m_step;
 
public:
    //得到当前移动步数
    int  getStep(){return m_step;};```
&#160;&#160;&#160;&#160;什么时候step增加呢？肯定是兔子移动的时候啊，我们修改下rabbitMove函数。
```c++
   int GameCtrl::rabbitMove() {
    Dir d = this->getRabbitMoveDir();
    if (Dir::none!=d) {
        //增加移动次数
        m_step++;
        m_rabbitPos=(*m_moveVec[(int)d])+m_rabbitPos;
        return m_rabbitPos.ver*NUM_MAPROWANDCOW+m_rabbitPos.hor;
    }
    else{
        return VAL_MAX;
    }```
     
___________________

# 三、HelloWorld中添加游戏成功和失败的部分

  &#160;&#160;&#160;&#160;首先修改之前，我们需要重新梳理一下游戏的逻辑流程。在添加了成功和失败判定部分后，我们游戏的逻辑流程应该是变成这样的:  
  
 * 判断是否当前状态已经失败，如果是，则xxx
 * 添加障碍
 * 判断是否当前状态已经成功，如果是，则xxx
 * 兔子移动  
  **（游戏的逻辑流程是非常重要的，如果错了，那么整个游戏的运行方式将会发生很大的变化。）**
&#160;&#160;&#160;&#160;有了上面的基础我们来修改一下触摸响应函数，这里假设成功和失败后先让它们显示MessageBox然后重新加载场景。
```c++
bool HelloWorld::onTouchBeganCallBack(Touch *t, Event* event){
    for (auto it =m_pNodes_obstacle.begin();it!=m_pNodes_obstacle.end() ; it++) {
        if((*it)->getBoundingBox().containsPoint(t->getLocation())&&(!(*it)->isVisible())){
             
            //判断是否失败
            if (m_ctrl->judgeFail()) {
                MessageBox("没办法，AI就是任性", " =.=,兔子跑了!!");
                Director::getInstance()->replaceScene(HelloWorld::createScene());
            }
 
            //add obt
            auto pObt = (*it);
            pObt->setVisible(true);
            m_ctrl->addObtacle(pObt->getTag());
             
            //判断是否胜利
            if (m_ctrl->judgeSuccess()) {
                MessageBox("Success", "orz,兔子被围!!");
                Director::getInstance()->replaceScene(HelloWorld::createScene());
            }
 
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
&#160;&#160;&#160;&#160;现在再编译运行一下，就会发现，当兔子被我们围住或者成功逃脱后就会有了成功和失败的判定。:)
_______________________
# 四、HelloWorld中添加积分管理的部分
&#160;&#160;&#160;&#160;关于积分管理，这里我会用到一个Cocos2d-x中的用户数据管理UserDefault来管理玩家的最好成绩。  
&#160;&#160;&#160;&#160;首先在头文件中添加两个string key。     

```c++
const string STR_STEP = "Step:"; //step的key
const string STR_BEST = "Best:"; //bestStep的key```

&#160;&#160;&#160;&#160;然后在AppDelegate.cpp中初始化用户数据中最好成绩的值。  

```c++
//set user data
    if (0==UserDefault::getInstance()->getIntegerForKey(STR_BEST.c_str())) {
        UserDefault::getInstance()->setIntegerForKey(STR_BEST.c_str(),VAL_MAX);
    }```
&#160;&#160;&#160;&#160;还需要修改一下触摸响应函数判定成功后的部分，使其能够修改最好成绩，并且在MessageBox上面显示出来。  
```c++
            //判断是否胜利
            if (m_ctrl->judgeSuccess()) {
                if (UserDefault::getInstance()->getIntegerForKey(STR_BEST.c_str())>m_ctrl->getStep()) {
                    UserDefault::getInstance()->setIntegerForKey(STR_BEST.c_str(), m_ctrl->getStep());
                }
                 
                //设置messageBox上面显示的信息
                char str_info[60];
                sprintf(str_info, "当前成绩为: %d,最好成绩为: %d",m_ctrl->getStep(),UserDefault::getInstance()->getIntegerForKey(STR_BEST.c_str()));
                MessageBox(str_info, "orz,兔子被围!!");
                Director::getInstance()->replaceScene(HelloWorld::createScene());
            }```
&#160;&#160;&#160;&#160;ok，接下来我们会让两个分数显示到UI上面。  
![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_c4ba0c7af9e05fb.png?ra=0.19033564743585885)  
&#160;&#160;&#160;&#160;这里再次会涉及到Cocos2d-x和CocosStudio两者之间的联系，需要使用UI::Text组件将其联系起来，添加响应的成员变量和成员函数。
```c++
//Step Text
    Text* m_pText_curStep;
     
    //BestStep Text
    Text* m_pText_bestStep;
 
    //设置Text
    void setText(Text* pTxt,int num);```
&#160;&#160;&#160;&#160;讲一个实用技巧吧，这里我们可以观察下CocosStudio相对应控件属性的类型值。
![](http://cdn.cocimg.com/bbs/attachment/Fid_41/41_300356_62ddc418c6d1020.png?ra=0.21180030005052686)  
&#160;&#160;&#160;&#160;可以对应到UI命名空间中同名控件，来实现两者的联系。在init方法中获取Text。
```c++
    //获取UITEXT
    m_pText_curStep = m_pNodes->getChildByName<Text*>("Text_curStep");
    m_pText_bestStep =  m_pNodes->getChildByName<Text*>("Text_bestStep");```
&#160;&#160;&#160;&#160;我封装了一个setText的方法来设置他们的文本信息。
```c++
void HelloWorld::setText(Text* pTxt,int num){
    std::stringstream stream;
    stream<<num;
    string str;
    stream>>str;
    if(m_pText_curStep==pTxt){
        pTxt->setString(STR_STEP+str);
    }
    else{
        //如果不是第一次设置，则设置成对应值
        if(num!=VAL_MAX){
            pTxt->setString(STR_BEST+str);
        }
        else{
            pTxt->setString(STR_BEST);
        }
    }
     
}```
&#160;&#160;&#160;&#160;C++的sstream可以实现int和string的转换（这里也可以使用Cocos2d-x中的数据结构Value实现，童鞋们可以思考一下）  
&#160;&#160;&#160;&#160;我们还需要在适当的时候调用这个方法，场景进入(onEnter)以及兔子移动的时候。
```c++
void HelloWorld::onEnter(){
    Layer::onEnter();
    //生成12个障碍
    this->setRandStones(12);
    //设置UI上面Text
    this->setText(m_pText_curStep,m_ctrl->getStep());
    this->setText(m_pText_bestStep,UserDefault::getInstance()->getIntegerForKey(STR_BEST.c_str()));
}
 
 
   //rabbit move
            int moveTag = m_ctrl->rabbitMove();
                if(m_pNodes->getChildByTag(moveTag)){
                     m_pNode_rabbit->runAction(MoveTo::create(0.1, m_pNodes->getChildByTag(moveTag)->getPosition()));
                    //设置Text
                     this->setText(m_pText_curStep,m_ctrl->getStep());
            }```
&#160;&#160;&#160;&#160;最后，是不是觉得每次障碍每次好像“假随机”一样总是生成到相同的位置，总是12个石头让人很不爽。  

&#160;&#160;&#160;&#160;没事，我们来给它在app类的初始化方法中添加设置随机种子的函数，让它变成个“真随机"。     
```c++
    //set rand
    srand(time(NULL));```

&#160;&#160;&#160;&#160;然后，需改一下生成随机障碍的部分，让它每次生成不同数目的障碍。
```c++
const int NUM_INITSTONE = 8; //至少8个障碍
const int NUM_DEATALSTONE = 4; //变化的障碍数
 
void HelloWorld::onEnter(){
    Layer::onEnter();
    //生成随机的障碍
    this->setRandStones(NUM_INITSTONE+CCRANDOM_0_1()*NUM_DEATALSTONE);
    //设置UI上面Text
    this->setText(m_pText_curStep,m_ctrl->getStep());
    this->setText(m_pText_bestStep,UserDefault::getInstance()->getIntegerForKey(STR_BEST.c_str()));
  
}```
&#160;&#160;&#160;&#160;Duang~~~,我们的《Rabbit Escape》就算完成了，赶快去抓兔子吧。

&#160;&#160;&#160;&#160;iphone真机测试DEMO演示视频:
<iframe height=498 width=510 src="http://player.youku.com/embed/XMTI5NTI2Nzg5Mg==" frameborder=0 allowfullscreen></iframe>
__________
# 五、最后说几句
  * **本教程算是完结了，虽然做的Demo比较简陋，但是希望童鞋们不要仅限于我带你们做出来的Demo，应该想一下给它添加你们自己的元素,比如背景音乐的添加，兔子的动画，游戏平衡的调整，寻路算法的优化等等.....希望每个看过这个教程的童鞋都能做出一个属于自己的神经兔。** 
  * **以后我还会把在学习cocos2d-x过程中一些有价值的Demo做出教程分享出来，如果大家有对教程什么不懂的地方以及我讲的什么不对的地方可以给我留言，或者新浪微博@Tezika给我私信，我会在第一时间内回复。**
  * **本次教程资源：<http://pan.baidu.com/s/1hqvhpU8>，密码: gqn8 (资源文件包括项目文件夹中的Classes以及Resources文件夹，CocosStudio资源文件夹Rabbit).**
   __________
