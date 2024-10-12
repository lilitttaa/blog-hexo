---
title: Unreal Kismet
category:
  - Game
---

## 获取选中节点 Copy 的内容

先通过Asset拿到 UBlueprint
![image-5.png](/images/Pub_Note_UnrealKismet/image-5.png)
![image.png](/images/Pub_Note_UnrealKismet/image.png)
然后通过 UBlueprint 获取 UEdGraph
![image-1.png](/images/Pub_Note_UnrealKismet/image-1.png)
UEdGraph 中包含所有 Nodes：
![image-2.png](/images/Pub_Note_UnrealKismet/image-2.png)
可以通过 Nodes 构建 SelectionSet：
![image-3.png](/images/Pub_Note_UnrealKismet/image-3.png)
然后就可以基于这个 SelectionSet 来 Copy 了：
![image-4.png](/images/Pub_Note_UnrealKismet/image-4.png)
