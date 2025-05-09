---
title: UnityProjectSettings设置优化
date: 2024-09-12 11:05:43
tags: 性能优化
---

# Unity Project Settings设置优化

## 1. 降低或禁用Accelerometer Frequency

IOS设备下，Unity 每秒对移动端的加速度计进行几次池操作。如果在应用程序中不会使用，请将其禁用，或者降低其频率，从而提升性能。

![img](UnityProjectSettings设置优化\1725277707452-ac4ddead-db4f-402b-ae51-e5ac126726fd.png)

## 2. **禁用不必要的** **Player** **或Quality** **设置**

在 **Player** 设置中，对不支持的平台禁用 **Auto Graphics API**，以便防止生成过多着色器变体。如果应用程序不支持，对较旧的 CPU 禁用 **Target Architectures**。

在 **Quality** 设置中，禁用不需要的质量级别。

## 3. **选择正确的帧率**

移动端项目必须在帧率和电池续航时间以及热节流之间获得平衡。不需要将设备限值推向 60 fps，可以折衷以 30 fps 运行。Unity 默认移动端为 30 fps。

您也可以通过 **Application.targetFrameRate** 在运行时动态调整帧率。例如，您甚至可以将缓慢或相对静止的场景降至 30 fps 以下，而将玩游戏时的 fps 设置保留为较高值。

## 4. Vsync垂直同步

移动平台不渲染半帧。即使在编辑器中禁用 Vsync (Project Settings > Quality)， Vsync 在硬件级别也处于启用状态。如果 GPU 无法足够快地刷新，将保持当前帧，从而有效降低每秒帧数。

**在移动平台上使用Application.targetFrameRate 控制帧率，移动平台的最大帧率就是屏幕的刷新率，移动平台上的默认帧率通常为每秒 30 帧。**

https://docs.unity3d.com/ScriptReference/Application-targetFrameRate.html

移动平台下强制开启垂直同步可能有几个原因：

- 移动设备追求低功耗和高效能。强制垂直同步的设计可以减少对复杂的帧缓冲管理和同步机制的需求，降低硬件成本和设计复杂度。
- 避免过高帧率渲染带来的高功耗问题，通过强制垂直同步，让GPU的工作节奏和屏幕刷新率协调一致。 

## 5. 启用**Prebake Collision Meshes**

可以通过PlayerSetting选项中勾选Prebake Collision Meshes选项来在构建应用时预先Bake出碰撞网格。

这项设置可以减少加载/初始化的时间, 虽然会增加一些构建时间和包体积。

![img](UnityProjectSettings设置优化\1725278077488-b236524e-5fc8-4ad2-a96d-13ce933fc651.png)

## 6. 设置Vertex Compression

![img](UnityProjectSettings设置优化\1725281737661-3bcb1e1e-a320-45d8-8cd4-1db4411bb981.png)

Vertex Compression：将顶点channel的数据格式format设置为16bit，**因此可以节约运行时的内存使用（float->half)。**

**顶点压缩有助于节省内存和带宽。**

影响Vertex Compression的因素：

1. 是否适合进行dynamic batching，开启了dynamic batching，则300个顶点以下的mesh不会被执行顶点压缩。
2. 在Model Importer中是否开启了Read/Write Enabled，Read/Write Enabled这个选项会使Vertex Compression失效。PS：(Read/Write enabled标志导致Texture会在内存中存在两份资源：一份在GPU，一份在GPU寻址空间【因为在大部分平台，从GPU内存读取非常缓慢。从GPU内存读取纹理资源到临时缓冲区非常不划算】)
3. 在Model Importer中是否开启了Mesh Compression，开启了Mesh Compression，则会override掉Vertex Compression以及Optimize Mesh Data的设置 。**Mesh Compression会将mesh在硬盘上的存储空间进行压缩，但是不会在runtime时节省内存。**

**Optimize Mesh Data：剔除不需要的channel，即剔除额外的数据，与压缩无关。**

**Mesh Compression：用压缩算法，将mesh数据进行压缩，结果是会减少占用硬盘的空间，但是在runtime的时候会被解压为原始精度的数据。**

[Unity的Mesh压缩：为什么我的内存没有变化？ - 慕容小匹夫 - 博客园 (cnblogs.com)](https://www.cnblogs.com/murongxiaopifu/p/10447076.html)

[Vertex Compression 顶点压缩 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/448647434)

![img](UnityProjectSettings设置优化\1725415914357-eb56b1bb-0a6e-4105-ae0c-40118c9a3699.png)

![img](UnityProjectSettings设置优化\1725416605285-3cfd5642-7e88-43bb-a544-3d7fe7667268.png)

## 7. 多线程渲染

单线程渲染的流程为，执行完逻辑的Update代码后，才执行Render的部分。

Render的部分里会执行渲染相关的指令调用 ，主线程需要调用图形API来更新渲染状态，设置渲染状态、调用DrawCall。

图形API调用都是与驱动层交互的，驱动层维护者所有的渲染状态，当渲染状态发生改变时可能会造成卡顿从而影响后续的图形API调用，从而拉长这一帧的时间。导致逻辑造成卡顿。

所以将渲染的部分抽出，形成独立的渲染线程，来减少主线程卡顿。

![img](UnityProjectSettings设置优化\1725351435738-abbcb998-b971-4231-b467-53dcaa0f8cd2.png)

**一般都建议保持开启Project Settings的Multithreaded Rendering选项(多线程渲染)。**

注意事项：

- 开启多线程渲染时，CPU等待GPU完成工作的耗时会被统计到Gfx.WaitForPresent函数中，而关闭多线程渲染时这一部分耗时则被主要统计到Graphics.PresentAndSync中。
- 在项目开发和测试阶段可以考虑暂时性地关闭多线程渲染并打包测试，从而更直观地反映出渲染模块存在的性能瓶颈。
- 对于正常开启了多线程渲染的项目，Gfx.WaitForPresent的耗时走向也有相当的参考意义。**测试中局部的GPU压力越大，CPU等待GPU完成工作的时间也就越长，Gfx.WaitForPresent的耗时也就越高。所以，当Gfx.WaitForPresent存在数十甚至上百毫秒地持续耗时时，说明对应场景的GPU压力较大。**
- **GPU压力过大也会使得渲染模块CPU端的主函数耗时（Camera.Render和RenderPipelineManager.DoRenderLoop_Internal）整体相应上升。**
