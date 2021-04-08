---
author: HKL
categories:
- 默认分类
cid: 23
date: "2016-05-04T14:42:00Z"
slug: create-random-in-c-sharp
status: publish
tags:
- Coding
- Windows
title: C# Random 生成不重复随机数
updated: 2019/01/29 16:40:17
---


## 方法一：以系统时间作为随机因子

```csharp
Random ro = new Random(10);
long tick = DateTime.Now.Ticks;
Random ran = new Random((int)(tick & 0xffffffffL) | (int) (tick >> 32));
```

这样可以保证99%不是一样。
之后，我们就可以使用这个Random类的对象来产生随机数，这时候要用到Random.Next()方法。这个方法使用相当灵活，你甚至可以指定产生的随机数的上下限。

不指定上下限的使用如下：
```csharp
int iResult;
iResult=ro.Next();
```

<!--more-->

下面的代码指定返回小于100的随机数：
```csharp
int iResult;
int iUp=100;
iResult=ro.Next(iUp);
```

而下面这段代码则指定返回值必须在50-100的范围之内：
```csharp
int iResult;
int iUp=100;
int iDown=50;
iResult=ro.Next(iDown,iUp);
```

除了`Random.Next()`方法之外，Random类还提供了`Random.NextDouble()`方法产生一个范围在0.0-1.0之间的随机的双精度浮点数：
```csharp
double dResult;
dResult=ro.NextDouble();
```

## 方法二：通过Hash表
```csharp
Hashtable hashtable = new Hashtable();
Random rm = new Random();
int RmNum = 10;
for (int i = 0; hashtable.Count < RmNum; i++)
{
	int nValue = rm.Next(100);
	if (!hashtable.ContainsValue(nValue) && nValue != 0)
	{
		hashtable.Add(nValue, nValue);
		Console.WriteLine(nValue.ToString());
	}
}
```