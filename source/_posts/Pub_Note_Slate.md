---
title: Unreal Engine Slate
category:
 - Game
sortValue: 05
---

## Resources

- [Slate 概述](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/slate-overview-for-unreal-engine?application_version=4.27)
- [Slate 架构](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/understanding-the-slate-ui-architecture-in-unreal-engine?application_version=4.27)
- [Slate 控件示例](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/slate-widget-examples?application_version=4.27)
- [Unreal Slate UI 的使用](https://cloud.tencent.com/developer/article/2347199)

## 介绍

Slate 是 UE 的 UI 框架，UE4 的 编辑器都是建立在 Slate 整个框架下，包括 UE4 用于 Runtime 游戏的 UMG 也是基于 Slate 系统的。

## 继承体系

![alt text](image-16.png)

Leaf Widget:
![alt text](image-15.png)

Compound Widget:
![alt text](image-17.png)

Panel:
![alt text](image-18.png)

## 创建

使用 SNew 或者 SAssignNew 创建 Widget
![alt text](image-8.png)

用 SLATE_ATTRIBUTE 定义属性，属性可以是值或者函数：
![alt text](image-9.png)
用 SLATE_ARGUMENT 定义参数，参数只能是值：
![alt text](image-10.png)
用 SLATE_NAMED_SLOT 添加 SLOT，也就是放置子控件的位置：
![alt text](image-20.png)
用 SLATE_EVENT 定义事件：
![alt text](image-21.png)

这些属性的初始化需要这么写：
![alt text](image-11.png)
![alt text](image-12.png)
定义部分这么写：
![alt text](image-13.png)
![alt text](image-14.png)

为什么要这么做？？？这样不是定义和构造都要写两遍吗？这里的 trade-off 是什么？

- 猜测是用来支持这样的链式编程：
  ![alt text](image-19.png)

## Attribute 和 Argument

这俩有啥区别，其实看宏定义就知道了：
![alt text](image-34.png)

- Argument 就是一个原生的值，没啥特别的

![alt text](image-35.png)

- 而 Attribute 要复杂一些，本质上是 TAttribute，可以 bind 一个函数，这样每次获取值的时候都会调用这个函数：
  ![alt text](image-36.png)
  没有 bind 的时候就是直接返回一个值：
  ![alt text](image-32.png)

## 添加子控件

把 NamedSlot Header 的内容填充到 HorizontalBox 新增的 Slot 中：
![alt text](image-24.png)
![alt text](image-28.png)
把下面的内容赋给 NamedSlot Header：
![alt text](image-25.png)
![alt text](image-29.png)

这种+号的写法是通过 SLATE_SUPPORTS_SLOT 实现的，通常定义在 Panel 类型的容器里边：
![alt text](image-27.png)
![alt text](image-26.png)

静态声明：
![alt text](image-23.png)
动态添加：
![alt text](image-22.png)

## Style

定义：
![alt text](image-30.png)
使用：
![alt text](image-31.png)

## 绘制

![alt text](image-33.png)

## 布局计算

![alt text](image-37.png)

## Shaped Text

图集大小：
![alt text](image-1.png)

文字数据从哪来的?

- 调用 FreeType 库来获取字形数据：
  ![alt text](image-2.png)

- 为什么图集需要更新?
  ![alt text](image.png)

- 为什么加入一个字体所有的字体图集都要添加？
  ![alt text](image-3.png)

为什么整个图集需要更新?

调用栈：

```cpp
FSlateTextureAtlas::MarkTextureDirty() TextureAtlas.cpp:108
FSlateTextureAtlas::AddTexture(unsigned int, unsigned int, const TArray<…> &) TextureAtlas.cpp:90
FSlateFontCache::AddNewEntry(FCharacterRenderData, unsigned char &, unsigned short &, unsigned short &, unsigned short &, unsigned short &) FontCache.cpp:909
FSlateFontCache::AddNewEntry(const FShapedGlyphEntry &, const FFontOutlineSettings &, FShapedGlyphFontAtlasData &) FontCache.cpp:824
FSlateFontCache::GetShapedGlyphFontAtlasData(const FShapedGlyphEntry &, const FFontOutlineSettings &) FontCache.cpp:1036
FSlateElementBatcher::BuildShapedTextSequence<…>(const FSlateElementBatcher::FShapedTextBuildContext &) ElementBatcher.cpp:2484
FSlateElementBatcher::AddShapedTextElement<…>(const FSlateDrawElement &) ElementBatcher.cpp:1270
FSlateElementBatcher::AddElementsInternal(const TArray<…> &, const FVector2D &) ElementBatcher.cpp:343
FSlateElementBatcher::AddElements(FSlateWindowElementList &) ElementBatcher.cpp:274
FSlateRHIRenderer::DrawWindows_Private(FSlateDrawBuffer &) SlateRHIRenderer.cpp:1254
FSlateApplication::PrivateDrawWindows(TSharedPtr<…>) SlateApplication.cpp:1291
FSlateApplication::DrawWindows() SlateApplication.cpp:1010
FSlateApplication::TickAndDrawWidgets(float) SlateApplication.cpp:1573
FSlateApplication::Tick(ESlateTickType) SlateApplication.cpp:1424
FEngineLoop::Tick() LaunchEngineLoop.cpp:4986
[Inlined] EngineTick() Launch.cpp:60
GuardedMain(const wchar_t *) Launch.cpp:179
LaunchWindowsStartup(HINSTANCE__ *, HINSTANCE__ *, char *, int, const wchar_t *) LaunchWindows.cpp:268
WinMain(HINSTANCE__ *, HINSTANCE__ *, char *, int) LaunchWindows.cpp:326
[Inlined] invoke_main() 0x00007ff615888046
__scrt_common_main_seh() 0x00007ff615888025
<unknown> 0x00007ffce7417374
<unknown> 0x00007ffce767cc91
```

## Slot

Slot 已经包含了 Padding 的部分:
![alt text](image-4.png)

![alt text](image-5.png)

## FExtender

在 Menu 中插入内容
![alt text](image-6.png)
![alt text](image-7.png)
