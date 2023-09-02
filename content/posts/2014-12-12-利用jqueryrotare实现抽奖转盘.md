---
title: 利用jqueryRotare实现抽奖转盘
author: admin
type: post
date: 2014-12-12T08:36:53+00:00
url: /archives/15390
categories:
 - 前端设计

---
很多公司到了年底都会做一些抽奖活动来刺激、吸引、粘住客户，比如抽奖转盘活动。

前几天用一个jqueryRotate插件实现了转盘的效果。比起那些很炫丽的flash是稍逊点，但也基本实现了需求

效果图：(也可以看线上的效果： [http://www.mbabycare.com/course/lottery](http://www.mbabycare.com/course/lottery))：

[![zhuanpanpic](http://blog.haohtml.com/wp-content/uploads/2014/12/zhuanpanpic.png)][1]

实现这个其实蛮简单的，转动的效果用的jqueryRotate插件，所以只要判断每个奖荐对应的角度，然后设置指针的转动角度就可以了。比如关键的是jqueryRotate这个插件的用法。

**jqueryRotate的资料：**

支持Internet Explorer 6.0+ 、Firefox 2.0 、Safari 3 、Opera 9 、Google Chrome，高级浏览器下使用Transform，低版本ie使用VML实现

google code地址： [http://code.google.com/p/jqueryrotate/](http://code.google.com/p/jqueryrotate/)

调用和方法：

```
$(el).rotate({
　　　　angle:0,  //起始角度
　　　　　animateTo:180,  //结束的角度
　　　　　duration:500， //转动时间
　　　　　callback:function(){}, //回调函数
　　　　　easing: $.easing.easeInOutExpo //定义运动的效果，需要引用jquery.easing.min.js的文件
　 })

$(el).rotate(45); //直接这样子调用的话就是变换角度

$(el).getRotateAngle(); //返回对象当前的角度

$(el).stopRotare(); //停止旋转动画

另外可以更方便的通过调用$(el).rotateRight()和$(el).rotateLeft()来分别向右旋转90度和向左旋转90度。
```

很简单吧，各种example可以看这里： [http://code.google.com/p/jqueryrotate/wiki/Examples](http://code.google.com/p/jqueryrotate/wiki/Examples)

下面是用jqueryRotate实现的抽奖转盘页面：

```
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>转盘</title>
<style>
    *{padding:0;margin:0}
    body{
        text-align:center
    }
    .ly-plate{
        position:relative;
        width:509px;
        height:509px;
        margin: 50px auto;
    }
    .rotate-bg{
        width:509px;
        height:509px;
        background:url(ly-plate.png);
        position:absolute;
        top:0;
        left:0
    }
    .ly-plate div.lottery-star{
        width:214px;
        height:214px;
        position:absolute;
        top:150px;
        left:147px;
        /*text-indent:-999em;
        overflow:hidden;
        background:url(rotate-static.png);
        -webkit-transform:rotate(0deg);*/
        outline:none
    }
    .ly-plate div.lottery-star #lotteryBtn{
        cursor: pointer;
        position: absolute;
        top:0;
        left:0;
        *left:-107px
    }
</style>
</head>
<body>
    <div class="ly-plate">
        <div class="rotate-bg"></div>
        <div class="lottery-star"><img src="rotate-static.png" id="lotteryBtn"></div>
    </div>
</body>
<script src="jquery-1.7.2.min.js"></script>
<script src="jQueryRotate.2.2.js"></script>

<script>
$(function(){
    var timeOut = function(){  //超时函数
        $("#lotteryBtn").rotate({
            angle:0,
            duration: 10000,
            animateTo: 2160,  //这里是设置请求超时后返回的角度，所以应该还是回到最原始的位置，2160是因为我要让它转6圈，就是360*6得来的
            callback:function(){
                alert('网络超时')
            }
        });
    };
    var rotateFunc = function(awards,angle,text){  //awards:奖项，angle:奖项对应的角度，text:提示文字
        $('#lotteryBtn').stopRotate();
        $("#lotteryBtn").rotate({
            angle:0,
            duration: 5000,
            animateTo: angle+1440, //angle是图片上各奖项对应的角度，1440是我要让指针旋转4圈。所以最后的结束的角度就是这样子^^
            callback:function(){
                alert(text)
            }
        });
    };

    $("#lotteryBtn").rotate({
       bind:
         {
            click: function(){
                var time = [0,1];
                    time = time[Math.floor(Math.random()*time.length)];
                if(time==0){
                    timeOut(); //网络超时
                }
                if(time==1){
                    var data = [1,2,3,0]; //返回的数组
                        data = data[Math.floor(Math.random()*data.length)];
                    if(data==1){
                        rotateFunc(1,157,'恭喜您抽中的一等奖')
                    }
                    if(data==2){
                        rotateFunc(2,247,'恭喜您抽中的二等奖')
                    }
                    if(data==3){
                        rotateFunc(3,22,'恭喜您抽中的三等奖')
                    }
                    if(data==0){
                        var angle = [67,112,202,292,337];
                            angle = angle[Math.floor(Math.random()*angle.length)]
                        rotateFunc(0,angle,'很遗憾，这次您未抽中奖')
                    }
                }
            }
         }

    });

})
</script>
</html>
```

这里的time跟data是要从后台获取的，但这里只是静态页面，所以我就利用了random随机数来尽量模拟抽奖的过程了。


time参数表示从后台请求是否成功，0是请求超时，1是请求成功(成功后再判断返回的值是什么样);

data就是请求返回的数据，1，2，3表示一二三等奖，0是不中奖，根据返回的数据，再去设置指针旋转的角度。

因为这个图片上的角度无法用公式计算出来，所以只能这样子计算出来后定死。

如果比较规则的话，应该可以用公式计算。

若你有更好的想法，也可以留言给我：）

[点击下载完整demo ][2]

转： [http://www.cnblogs.com/mofish/archive/2013/01/24/2875516.html](http://www.cnblogs.com/mofish/archive/2013/01/24/2875516.html)

 [1]: http://blog.haohtml.com/wp-content/uploads/2014/12/zhuanpanpic.png
 [2]: http://blog.haohtml.com/wp-content/uploads/2014/12/rotate.rar