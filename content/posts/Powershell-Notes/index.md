---
title: Powershell Notes
date: 2020-08-04
categories:
  - windows
markup: mmark
math: true
tags:
    - powershell
---

查看文件（类似cat的功能）

```powershell
type $file
```

创建文件：

```powershell
New-Item [path] 
```

使用记事本打开文件

```powershell
Start-Process notepad $file
```

打开文件夹

```powershell
ii .
```
或者
```powershell
Invoke-Item .
```