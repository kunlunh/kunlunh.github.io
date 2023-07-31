---
author: HKL
categories:
- 默认分类
cid: 26
date: "2016-06-07T15:54:00Z"
slug: winform-key-program
status: publish
tags:
- Coding
- Windows
- Microsoft
title: C# 如何给Winform的button等控件添加快捷键
updated: 2019/01/29 17:31:27
---


第一种：Alt + *(按钮快捷键)    
    
在大家给button、label、menuStrip等控件设置Text属性时在名字后边加&键名就可以了,
比如`button1.text= "确定(&O)"`。就会有快捷键了，这时候按Alt+O就可以执行按钮单击事件。    
    
    
第二种：Ctrl+*及其他组合键    
    
在WinForm中设置要使用组合键的窗体的KeyPreview(向窗体注册键盘事件)属性为True;    
    
然后使用窗体的KeyDown事件(在首次按下某个键时发生).    
    
实例代码： 


<!--more-->


   
```csharp
private void ***_KeyDown(object sender, KeyEventArgs e)    
{    
    
if (e.KeyCode == Keys.F && e.Control)    
{    
    button1.PerformClick(); //执行单击button1的动作    
}    
    
}    
```
 
注：
1、\***代表窗体名称，大家可以看一下 ”Keys”的枚举参数，以实现自己需要    
    
2、还有一个问题，当使用Ctrl + *快捷键时，对于焦点在可写的控件（如TextBox）上时，可能会将* 键值同时输入，则需要加另一句话将Handled设置为true，以取消 KeyPress 事件。    
    
即：    
```csharp
private void ***_KeyDown(object sender, KeyEventArgs e)    
{    
    
if (e.KeyCode == Keys.F && e.Control)    
{    
    
e.Handled = true; //将Handled设置为true，指示已经处理过KeyPress事件    
button1.PerformClick();     
    
}    
    
}    
```  
    
第三种：    
    
还是以button为例。给form添加一个contextMenuStrip1，将其邦定到button上，假设为button1。给contextMenuStrip1添加一个item，然后为它设置快捷键（就是你想加在button上的快捷键），并且将它的Visible属性设为false。这样，button1的快捷键设置成功。    

例如：窗口FormTestLink（的keydown事件）的回车快捷键添加
```csharp
private void FormTestLink_KeyDown(object sender, KeyEventArgs e)    
{    
    if (e.KeyCode == Keys.Enter)    
    {    
        btnTest_Click(sender, e);    
    }    
}        
```


来自：http://www.cnblogs.com/benben7466/archive/2009/07/06/1517993.html    
