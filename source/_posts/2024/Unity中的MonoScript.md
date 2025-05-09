---
title: Unity中的MonoScript
date: 2024-09-09 17:01:17
categories: 资源管理篇
tags: 资源管理
---

# Unity中的MonoScript

MonoBehaviour的脚本，其实是一个MonoScript类型的Asset，这个类型直接继承TextAsset，作为文本文件载入。

可以通过`MonoScript.FromMonoBehaviour`返回MonoBehaviour脚本的MonoScript对象。

MonoBehaviour持有一个对MonoScript的引用。

**MonoScript包含了定位某个程序类所需的信息，含有三个字符串:程序库名称、类名称、命名空间。**

在构建工程时，Unity会收集Assets文件夹内独立的脚本文件并将它们进行编译，组成一个Mono程序库。

Unity会将Assets目录下的不同文件夹进行分开编译。默认会放在`Assembly-CSharp.dll`中，而Plugins目录下的会被放置在`Assembly-CSharp-firstpass.dll`中。

这些程序库会被包含在最终构建的Unity程序中。这些程序库都会被MonoScript所引用。与其他资源不同，所有脚本文件都会在程序第一次启动时被加载。

**因此AssetBundle内不包含脚本的可执行数据，有的则是MonoScript，进而找到对应的类。**

**在Prefab上挂载的MonoBehaviour脚本，在打包成AssetBundle后实际上存储的则是MonoScrpit的PathId，如果访问不到MonoScript则会导致Prefab上挂载的MonoBehaviour脚本变为Missing的情况。**
