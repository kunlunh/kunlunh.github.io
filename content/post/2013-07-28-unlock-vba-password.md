---
author: HKL
date: "2013-07-28T07:41:00Z"
slug: unlock-vba-password
tags:
- Windows
- Office
title: Unlock a VBA password protected Excel file
---


## Unlock a VBA password protected Excel file

##（怎样解除受VBA密码保护的Excel文件）



    

> Ever felt the need to open a VBA protected excel file… maybe one of your old files that contained an excellent routine! How do you come out of that pain?


##   当你想要打开一个VBA加密的文件。这篇文章可能会帮助到你。。


<!--more-->


`Important: This article is for educational purposes. Try this method for opening ONLY your own files, as I did too!`

### （只能用于测试自己的文件！！）

 

以下是正文内容。这些英文比较简单。只翻译一些主要步骤。其他就自己看吧。


So how does Excel store the file contents – cell data including formulas and formats, conditional formatting, VBA code, etc. etc. Lets investigate.

Create a new Excel file MyTest.xlsm and enter some dummy test data in the first sheet. （先新建一个xlsm文件。）

Add some formulas and conditional formatting (if you want to really understand the details).）（自己输入一些内容）


![alt text](http://photo.fanpou.com/f/5/)    

Let us now see how excel stores this data in the file.

Open the file in notepad or a hex editor.（用记事本打开该文件，推荐用notepad++）

Did you notice the first 2 characters? “PK”. （前两字“PK”）

So Excel compresses its file contents. Now we know why there is not much difference if you compress an Office 2007 file.

![alt text](http://photo.fanpou.com/f/6/)        

Lets look into the compressed contents. Rename the file extension from .xlsm to .zip。（改文件后缀为zip）

![alt text](http://photo.fanpou.com/f/7/)     

Open the MyFile.zip file. （打开改后缀后的文件）

Wow! its an extensive structure with xml files to store the workbook, worksheets, calculations, sharedstrings, etc.

![alt text](http://photo.fanpou.com/f/8/)     

This is how the XML of the Sheet1 looks

![alt text](http://photo.fanpou.com/f/9/)     

Lets explore more. Lets go back to our original file and add some VBA code to it.

![alt text](http://photo.fanpou.com/f/a/)     

Add a password and protect the VBA code.（添加个VBA密码）

![alt text](http://photo.fanpou.com/f/b/)    

Save the file and redo the same steps as earlier to open the xml file structure. 

We now have another XML file called vbaProject.bin.（之前的xml文件都变成了个vbaProject.bin）

This is the file that I need to recover.

Lets investigate further. 

Open this file "vbaProject.bin" in a Hex Editor (there are lots of free ones out there… the one I use is Hex Editor Neo 

![alt text](http://photo.fanpou.com/f/c/)   

（用 [Hex Editor Neo](http://www.hhdsoftware.com/free-hex-editor)打开文件 "vbaProject.bin"）