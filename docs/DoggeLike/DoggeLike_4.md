# 对象与玩家之间的互动

**********

对于此项目来说，玩家与其他对象之间的具体互动方式就是飞船与圆圈相撞，游戏结束。
然后就会黑屏，表示游戏结束。

细致剖析这段话，我们可以发现，完成玩家直接的互动需要两个关键：

1. 飞船与圆圈的碰撞判定（互动方法）
2. 判定为真之后的画面变化

<br>

## 定义互动方法

*************

根据本项目的特征，首先按照前面所讲述的内容，制定一个简单的互动方法：

如果玩家对象与非玩家对象的关键点距离小到一定值后，则会返回一个布尔值，告诉程序要出现游戏结束的画面了。

实现这个判定过程首先得需要得到对象的关键点：

```java
public int[][] hit_poi(){
        return heart_poi;
    }
```

在这里，使用一个新类，专门用来生成各种各样对象之间的关键点获得方法，以及判定方法

```java
import java.util.ArrayList;

//触发新关卡
public class GobjInteract {
    public  ArrayList<int[][]> hit_poi = new ArrayList<>();
    //接收关键点数据
    public GobjInteract(CirCle cc, ship ns ){

        int[][] add_poi = cc.hit_poi();
        hit_poi.add(add_poi);

        add_poi = ns.hit_poi();
        hit_poi.add(add_poi);
    }
}
```

```xml
        使用一个泛型对象去储存这些关键点,
        这样如果各种对象增多，可以自动为其分配空间。
```
<br>

判断对象之间的干涉还是比较简单的：

```java
package Gameboot.Gobj;

import java.util.ArrayList;

//触发新关卡
public class GobjInteract {
    public  ArrayList<int[][]> hit_poi = new ArrayList<>();
    //接收关键点数据
    public GobjInteract(CirCle cc, ship ns ){

        int[][] add_poi = cc.hit_poi();
        hit_poi.add(add_poi);

        add_poi = ns.hit_poi();
        hit_poi.add(add_poi);
    }

    //判断是否干涉，干涉返回真
    public boolean Interact(){
        boolean flag = false;

        //遍历各个点
        for (int i =0;i<hit_poi.get(0).length;i++){
            //得到ship核心
            int[][] OP = hit_poi.get(1);
            //得到mon核心
            int[][] DS = hit_poi.get(0);

            //计算两点之间距离
            double S = Math.pow((DS[i][0] - OP[0][0]),2) + Math.pow((DS[i][1] - OP[0][1]),2);

            if(S <= 400){
                flag = true;
                break;
            } else if (i < hit_poi.get(1).length-1) {
                continue;
            }else {
                flag = false;
            }
        }

        return flag;
    }
}
```

<br>

## 结束关卡

****************

其实某种意义上来说，结束画面也是一种关卡。可以按照我们之前设计关卡的方式来设计它。

```java
public class EndRound extends JPanel {

    private static int band = 2;
    public void paint(Graphics graphics){
        Graphics2D gd = (Graphics2D) graphics;

        BasicStroke stroke = new BasicStroke(band);

        gd.setColor(Color.black);

        for (int i=0,y1=0,y2=0 ; i<900 ;i+=3){
            gd.drawLine(y1,y2,400,y2);
            y1+=3;
            y2+=3;
        }
        band+=2;
        System.out.println("the end");
    }
}
```

```xml
    定义个结束关卡类 EndRound 具体内容为在窗口上绘制大量的黑线。
```
<br>

<br>

## 将关卡添加到后续画面上

*********************

其实只要在 *background* 类中加上以下几句，并且替换掉之前 while(true) 中的条件就好了，

```java
        //移除关卡
        scence.remove(rd);
        //添加结束关卡
        scence.add(ed);
        //容器关系排序
        scence.revalidate();
```

难就难在 .revalidate() 这一句上，因为从前到后，不论是 JFrame ， JPane ， 还是他们的父类，一连串后最终继承的
都是一个叫做 component 的类，所以很多方法都是通用的，但不一定有效。需要去调试才知道哪一个是起作用的。

[ "使用IDEA的调试功能" ](./DoggeLike_ideaDebug "使用IDEA的调试功能" )