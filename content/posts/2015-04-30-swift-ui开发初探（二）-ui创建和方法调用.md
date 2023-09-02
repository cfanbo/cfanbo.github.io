---
title: Swift UI开发初探（二） UI创建和方法调用
author: admin
type: post
date: 2015-04-29T18:20:45+00:00
url: /archives/15668
categories:
 - 程序开发
tags:
 - swift

---
[http://letsswift.com/2014/06/swift-ui-two/](http://letsswift.com/2014/06/swift-ui-two/)

```
1
2
3
4
5
```

```
//1、root控制器的创建
var rootCtrl =RootViewController()

var root:UINavigationController =UINavigationController(rootViewController: rootCtrl)
self.window!.rootViewController = root
```

```
1
2
3
4
5
```

```
//2、tab控制器的创建
var tab =UITabBarController()
tab.tabBar.barTintColor =UIColor.blackColor()
tab.viewControllers = [oneCtrl, twoCtrl, threeCtrl, fourCtrl, fiveCtrl]
self.window!.rootViewController = tab
```

```
1
2
```

```
//3、声明属性
var tableView:UITableView?
```

```
1
2
3
4
5
6
7
8
9
10
11
```

```
//4、抽出TableView的创建方法
func _initTableView(){
    //TableView的创建和设置
    self.tableView=UITableView(frame:CGRectMake(,20,CGRectGetWidth(self.view.frame),CGRectGetHeight(self.view.frame)-64))
    self.tableView!.delegate =self
    self.tableView!.dataSource =self
    self.tableView!.autoresizingMask = UIViewAutoresizing.FlexibleHeight |UIViewAutoresizing.FlexibleWidth
    self.tableView!.registerClass(UITableViewCell.self, forCellReuseIdentifier:"cell")
    self.view?.addSubview(self.tableView)
    self.tableView!.separatorColor =UIColor.cyanColor()
}
```

```
1
2
3
4
5
```

```
//dataSource 返回100个row
func tableView(tableView:UITableView!, numberOfRowsInSection section: Int) ->Int
{
    return 100
}
```

```
1
2
3
4
5
6
7
```

```
//cell的创建
func tableView(tableView:UITableView!, cellForRowAtIndexPath indexPath:NSIndexPath!) ->UITableViewCell!
{
    let cell = tableView .dequeueReusableCellWithIdentifier("cell", forIndexPath: indexPath)asUITableViewCell
    cell.textLabel.text =String(format:"%i", indexPath.row)
    return cell
}
```

UIKit


```
1
2
3
4
5
6
7
8
9
10
```

```
// UILabel
func createLabel() ->UILabel {
    var label:UILabel =UILabel(frame:CGRectMake(10,80,self.view.frame.size.width-20,50))
    label.backgroundColor =UIColor.clearColor()
    label.textAlignment =NSTextAlignment.Center
    label.textColor =UIColor.blackColor()
    label.font =UIFont.systemFontOfSize(25)
    label.text ="Hello Swift"
    return label
}
```

```
1
2
3
4
5
6
7
```

```
// UIView
func createView() ->UIView {
    var orginY =CGRectGetMaxY(self.myLabel.frame) +10
    var myView:UIView =UIView(frame:CGRectMake(10, orginY,self.view.frame.size.width-20,30))
    myView.backgroundColor =UIColor.whiteColor()
    return myView;
}
```

```
1
2
3
4
5
6
7
8
9
10
11
```

```
// UIButton
func createButton() ->UIButton {
    var orginY =CGRectGetMaxY(self.myView.frame) +10
    var button:UIButton =UIButton(frame:CGRectMake(10, orginY,self.view.frame.size.width-20,30))
    button.backgroundColor =UIColor.greenColor()
    button.setTitle("Button", forState:UIControlState.Normal)
    button.titleLabel.font =UIFont.systemFontOfSize(12)
    button.addTarget(self, action:"tappedButton:", forControlEvents:UIControlEvents.TouchUpInside)
    button.tag =100
    return button
}
```

```
1
2
3
4
5
6
7
8
```

```
// UIImageView
func createImageView() ->UIImageView {
    var orginY =CGRectGetMaxY(self.myButton.frame) +10
    var imageView:UIImageView =UIImageView(frame:CGRectMake((self.view.frame.size.width-100)/2, orginY,100,50))
    var image:UIImage =UIImage(named:"user")
    imageView.image = image
    return imageView
}
```

```
1
2
3
4
```

```
// Button target
func tappedButton(sender:UIButton!) {
    println(sender.tag)
}
```

push 控制器的方法


```
1
2
3
4
```

```
var listCtrl:UIViewController =UIViewController()
listCtrl.title ="View Controller"
listCtrl.view.backgroundColor =UIColor.redColor()
self.navigationController.pushViewController(listCtrl, animated:true)
```

pop


```
1
```

```
self.navigationController.popViewControllerAnimated(true)
```