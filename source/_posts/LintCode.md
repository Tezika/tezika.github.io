title: LintCode Resolving(持续更新中)
date: 2016-03-15 18:30:07
tags:
- 数据结构和算法
---
## 介绍  
 &#160;&#160;&#160;&#160; 为了学习和巩固数据结构和算法基础，于是便开始受苦的LintCode之旅，现在差不多刷了有50%，所以想把一些题的自己思路，以题解的形式记录下来。同时
可以在[Github中的题解库](https://github.com/Tezika/LintCode)，找到相应题的详细代码:).
## Hard,54,[转换字符串到整数](http://www.lintcode.com/zh-cn/problem/string-to-integer-ii/)    
  &#160;&#160;&#160;&#160; 这道题看似比较简单，但是其实坑很多的。比如会出现“   &#160;&#160;&#160;  -52”，“  &#160;&#160;&#160;&#160;52lintcode”这样畸形和有爱的数据。个人认为这道题主要是考虑你处理一些不寻常数据时候的能力，通过率比较低，下面是代码。  
```c++ 

  /**
     * @param str: A string
     * @return An integer
     */
     #define MAX_INT 2147483647
     #define MIN_INT -2147483648 
     long long foo(string str,int s,int e){
       long long  ret = 0;
       int posOfEnd = -1;
       for (int i = s; i <= e; ++i){
         /* code */
         if(str[i] < '0' || str[i] >'9'){
            posOfEnd = i;
            break;
         }
       }
       if(posOfEnd == -1){
         posOfEnd = e + 1;
       }
       for (int i = s; i < posOfEnd; ++i){
         ret +=((str[i] - '0') * pow(10,posOfEnd - i - 1));  
       }
       return ret;

    }
    int atoi(string str) {
        // write your code here
        int strLen = str.length();
        if (strLen == 0){
          /* code */
          return 0;
        }
        //处理一开始的空格
        int posOfNotSpace = 0;
        if(str[0] == ' '){
           for (int i = 0; i < strLen; ++i){
                 if(str[i] != ' '){
                    posOfNotSpace  = i;
                    break;
                 }
           }
        }
        str = str.substr(posOfNotSpace,strLen);
        strLen = str.length();
         if(str[0] == '-' || str[0] == '+'){
          auto res = foo(str,1,strLen - 1);
          if (str[0] == '-'){
            res = -res;
            if(res < MIN_INT){
             return MIN_INT;
            }else{
              return res;
            }
          }else{
           if(res > MAX_INT){
             return MAX_INT;
           }else{
             return res;
           }
          }
        }else if(str[0] >= '0' && str[0] <= '9'){
           auto res = foo(str,0,strLen - 1);
           if(res > MAX_INT){
             return MAX_INT;
           }else{
             return res;
           }
        }
        else{
          return 0;
        }

    }
```


