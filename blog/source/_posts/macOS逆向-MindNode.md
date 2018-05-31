title: macOS逆向(MindNode)
id: 1527694139538
author: 王磊磊
date: 2018-05-30 23:29:03
tags:
---
---


## 目标程序:MindNode(试用版本)
- 目标结果1:	去除欢迎界面
- 目标结果2:	去除30个节点限制

## 最终效果



![去除30个节点限制Document.gif](http://upload-images.jianshu.io/upload_images/1855222-b49b3c05fe0ae5a1.gif?imageMogr2/auto-orient/strip)

## 工具 

-  class-dump ( 逆向工程的入门级工具，导出一个App的某些信息,导出头文件)	
-  Hopper Disassembler v4	( 反编译工具，根据可执行文件反编译出汇编码)
-  gdb ( 调试器:找到想要改变的地址处的16进制代码)
- Hex Fiend (16进制文件编辑器，要用这个修改原来的16进制文件。改变想要改变的地址处的16进制代码)


## 分析思路

1. 使用Hooper反编译[MindNode]()
2. 使用gdb加载[MindNode]() 
	- 查找16进制(x/x 地址) 代码
3. 使用Hex Friend编辑[MindNode]()
   - 查找16进制(x/x 地址) 代码
4. 重新给[MindeNode]()签名 
  - codesign -f -s 证书名 /Applications/MindNode.app/Contents/MacOS/MindNode

## 完成逆向
---
---

# 详细过程


### hooper 搜索appdelegate入口

![屏幕快照 2017-04-04 上午12.36.43.png](http://upload-images.jianshu.io/upload_images/1855222-bd76379af637584b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-04-04 上午12.33.39.png](http://upload-images.jianshu.io/upload_images/1855222-485f2e02f0572a98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 下图就是30个节点弹出框伪代码
![8E57FEC8-7247-491E-914C-0416B50EDE33.png](http://upload-images.jianshu.io/upload_images/1855222-10d07a464a2f6c3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![0216CD6E-796D-44DD-8117-D362F450DE7F.png](http://upload-images.jianshu.io/upload_images/1855222-91dd7aa66011e71a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 应该就是调用了这个代码


![B84F4AAE-950D-47CA-921E-12AA6CB1BFBF.png](http://upload-images.jianshu.io/upload_images/1855222-0b1bdc71d1cf8ce8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 记错汇编指令瞎改(原来可以30个节点,现在刚打开就弹出!!尴尬了)

![157172B3-D98A-4C78-81D3-4A4FBF48A0FD.png](http://upload-images.jianshu.io/upload_images/1855222-fe3a87682366a21c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 恢复修改过的代码,继续分析
- 监控节点数量的伪代码1
```
char -[MNCDocumentKVOController _mindMapObjectsEqualMaxCountForDemoVersion](void * self, void * _cmd) {
    rbx = [[self mindMapObjects] retain];
    r14 = [rbx count];
    r15 = *0x100331ad8;
    [rbx release];
    if (r14 < r15) {
            rax = 0x0;
    }
    else {
            [MNXDemoManager showCanvasObjectsExceededSheet];
            rax = 0x1;
    }
    rax = rax & 0xff;
    return rax;
}
```
- 监控节点数量的伪代码2

```
char -[MNCDocumentKVOController _mindMapObjectsExceedMaxCountForDemoVersion](void * self, void * _cmd) {
    rbx = [[self mindMapObjects] retain];
    r14 = [rbx count];
    r15 = *0x100331ad8;
    [rbx release];
    if (r14 > r15) {
            [MNXDemoManager showCanvasObjectsExceededSheet];
            rax = 0x1;
    }
    else {
            rax = 0x0;
    }
    rax = rax & 0xff;
    return rax;
}
```
+ 对于汇编代码1

![7DD597B0-47E1-4BE4-B79F-0DE1CDE8AF9A.png](http://upload-images.jianshu.io/upload_images/1855222-1343adc4cd60cc89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ 对于汇编代码2

![4B29325B-7CC4-4764-BC73-ED30F36E6416.png](http://upload-images.jianshu.io/upload_images/1855222-7512123cf7cd866c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 搞定这两个很明显就可以去掉限制了
``` 有点儿激动了,希望今天可以完成任务,因为4.4号sgi截止日期的最后一天```

### 把上面两个判断中的r15修改为比30大的数字就可以了.我设置了0xff!okay 终于去除了30个节点的限制(数一数下面的节点数肯定超过30了)
![C52C9181-4979-4FF4-BD6C-01B135206B7B.png](http://upload-images.jianshu.io/upload_images/1855222-693ef2ce01e4adaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)