---
title: Unreal Kismet
tag:
  - unreal engine
category:
 - Game
sortValue: 011
---

## 获取选中节点 Copy 的内容

先通过 Asset 拿到 UBlueprint
![alt text](image-5.png)
![alt text](image.png)
然后通过 UBlueprint 获取 UEdGraph
![alt text](image-1.png)
UEdGraph 中包含所有 Nodes：
![alt text](image-2.png)
可以通过 Nodes 构建 SelectionSet：
![alt text](image-3.png)
然后就可以基于这个 SelectionSet 来 Copy 了：
![alt text](image-4.png)

## 蓝图保存导出额外数据

UBlueprint 是资产类，保存了节点图的所有信息，可以在 PreSave 或者 Serialize 里边做一些额外的操作，比如保存一些额外的数据。

![alt text](image-6.png)

在 FBlueprintEditorUtils 这个类里边提供了很多辅助类
![alt text](image-7.png)
Ed 开头的 Graph 或者 Node 是编辑器下的实体类，从这里边可以直接操作节点、引脚等
![alt text](image-8.png)
![alt text](image-9.png)

可以这样拿到蓝图生成类：
![alt text](image-10.png)
属性和变量的一些东西可以参考：
![alt text](image-11.png)
