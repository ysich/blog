---
title: ManagerOfManager
date: 2023-01-17 14:14:51
categories: 框架篇
tags: Manager 框架
---

# 框架搭建随笔

## 版本号

使⽤版本号命名的⽂件格式为: Framework_vX.Y.Z

- X 是主版本号，⽤于不向前兼容的更新。
- Y 是中间版本，⽤于可向前兼容的功能性更新。
- Z 是⼩版本号，⽤于功能完善和 bug 修复的更新

⼀般都是从 v0.1.1 这个版本开始发布的，但是这个版本呢叫做 mvp 版本，也就是最⼩可验证版本。后续发布版本都需要向前兼容

## Obsolete 标签

使用Obsolete标签标记方法已弃用，使用其他方法。

添加弃用标签后会报警报提醒

```c#
[Obsolete("方法已过时，请使用xx方法")]
public static void OpenInFolder(string folderPath)
{
	Application.OpenURL("file://" + folderPath);
}
```

## Partial关键字

当类后续可能增长的时候使用，各个部分类需要用相同的访问权限。而且每个部分类都需要加`partial`关键字。

## 方法结构重复 解决方案

当一个方法的的参数类型不同时，如果有共同父类可以将类型设置为父类，子类可以进行使用

```c#
public static int GetRandomValueFrom(int[] values)
{
    return values[Random.Range(0, values.Length)];
}
public static float GetRandomValueFrom(float[] values)
{
    return values[Random.Range(0, values.Length)];
}
public static string GetRandomValueFrom(string[] values)
{
    return values[Random.Range(0, values.Length)];
}
```

可以将类型设置为object

```c#
public static object GetRandomValueFrom(object[] values)
{
    return values[Random.Range(0, values.Length)];
}
```

### 泛型实现结构复用

使用泛型实现方法复用

```c#
public static T GetRandomValueFrom<T>(params T[] values)
{
    return values[Random.Range(0, values.Length)];
}
```

### params关键字

修饰形参**必须为一维数组**，并且方法声明只能有一个`params`，如果不是一维数组编译器将报错。

使用 `params` 参数调用方法时，可以传入：

- 数组元素类型的参数的逗号分隔列表。

- 指定类型的参数的数组。

- 无参数。 如果未发送任何参数，则 `params` 列表的长度为零。

``` c#
//数组元素类型的参数的逗号分隔列表
public void Get()
{
	GetRandomValueFrom<int>(1, 2, 3);
}
//无参数
public void Get()
{
	GetRandomValueFrom<int>();
}
```

## 消息机制

### Unity内置消息机制

方法调用使用字符串，可能用到反射，尽量不用。

```c#
this.SendMessageUpward("MethedName");
```

### 通过委托通知

A注册特定方法，B声明委托。当B想调用A的方法时，通过委托通知。

### 消息机制提供的功能

- 注册事件
- 注销事件
- 发送事件

```c#
MsgDispatcher.Register("消息名",(obj)=>{ /* 处理消息 */ });
MsgDispatcher.Send("消息名","消息内容");
MsgDispatcher.UnRegister("消息名");
```

## new Class的优化

当一个class作为存储数据时，为了减少new的次数可以做一个对象池进行存储

```c#
private class MsgRecord
{
    //私有构造函数后，class无法被new
    private MsgRecord() { }
    //对象池
    static Stack<MsgRecord> mMsgRecordPool = new Stack<MsgRecord>();
    //创建对象
    public static MsgRecord Allocate(string msgName, Action<object> onMsgReceived)
    {
        MsgRecord msgRecord;
        if (mMsgRecordPool.Count > 0)
        {
            msgRecord = mMsgRecordPool.Pop();

        }
        else
        {
            msgRecord = new MsgRecord { };
        }
        msgRecord.Name = msgName;
        msgRecord.OnMsgReceived = onMsgReceived;
        return msgRecord;
    }
    //移除对象
    public void Recycle()
    {
        Name = null;
        OnMsgReceived = null;
        mMsgRecordPool.Push(this);
    }
    public string Name;
    public Action<object> OnMsgReceived;
}
```

## 框架的定义

`框架`：提供⼀个架构（⽂件结构、约定等等），你必须遵守它，只要你遵守，那剩下的就 全部处理通⽤需求了。 

**好架构=好规则**

## 库的定义

库，插到既有 架构上，补充特定功能。

## Unity常用架构

### 1.EmptyGo

 在 Hierarchy 上创建⼀个空的 GameObject,然后挂上所有与 GameObject ⽆关的逻辑控制的脚 本。使⽤GameObject.Find() 访问对象数据。 

缺点:逻辑代码散落在各处,不适合⼤型项⽬。 

###  2.Simple GameManager

所有与 GameObject ⽆关的逻辑都放在⼀个单例中。 

缺点:单⼀⽂件过于庞⼤ 

###  3.Manager Of Managers

将不同的功能单独管理。

如下:

- MainManager: 作为⼊⼝管理器。
- EventManager: 消息管理。 
- GUIManager: 图形视图管理。 
- AudioManager: ⾳效管理。
- PoolManager: GameObject管理（减少动态开辟内存消耗,减少GC)。  存储各类型的spawnPool，spawnpool存储各prefabPool。删除和添加时如果不需要立即操作，可以分步进行添加删除。(最基本拥有) 
- LevelManager: 关卡管理。 (最基本拥有)
- GameManager: 游戏管理。
- SaveManager: 配置&存储管理。(最基本拥有)
- `Easy save2`插件使用二进制操作 ，对数据加密
- MenuManager 菜单管理。 


### 4.将 View 和 Model 之间增加⼀个媒介层

UI和逻辑分离

 MVCS:

​	StrangeIOC 插件。

​	`IBinder.Bind<Key>().To<Value>();`

​	`IBinder.Bind<Key>().To<Value>().ToName(name);`当key和value都相同时，根据name区分

​	通过event和listener来触发按键的操作

​	机制依赖于C#的Reflection(反射)，效率慢，模式、思想和理念可借鉴

MVVM:

​	uFrame 插件

### 5.ECS (Entity Component Based System) 

 Unity 是基于 ECS,⽐较适合 GamePlay 模块使⽤ 

## Manager Of Managers架构模式

### LevelManager

Unity里的`LoadScene方法`只能传递scene名字或index。改名或者变换顺序时变得非常麻烦。通过配置表配置，读取level时按顺序读取配置表即可。

### PoolManager

两个经典操作，Spawn、Despawn

#### Spawn

在创建新资源时，对象池有则直接调用，没有则需要初始化。

#### DeSpawn

当池子容量达到指定上限时，将第一个第二个按照队列顺序进行销毁，先进先出。

#### 对象池优化

一个PoolManager下有若干个SpawnPool来管理一类的物体。比如NPC一个Pool，物品一个Pool。一个SpawnPool有若干个PrefabPool，一个PrefabPool只能存储一个Prefab，可以进行单个Prefab的加载和卸载。

对于每一个PrefabPool可以管理两个List，一个是ActivetedList，一个是DeactivateList，并管理所有Prefab的加载和卸载过程。

在删除时对数量要严格控制，一帧内不能同时删除太多物体，否则会触发GC。需要PoolManager管理时能够缓释，一帧只删除少量对象。

### SaveManager

`Easy Save2`二进制进行Load和Save，与Unity很好的结合，Unity类型基本都能Serialized。比Json的一些方案快。

### MainManager

入口管理器，如资源加载流程、第三方SDK启动流程、热更新检测，都是在入口处完成的。

在开发阶段不同流程会有不同的log或调试需要进行阶段划分进行屏蔽。

职责：

- 管理多个入口
- 负责游戏的启动流程。

####  阶段划分

1. 开发阶段:不断编码->验证结果->编码->验证结果->....
2. 出包/真机阶段：跑完整流程，QA测试
3. 发布阶段：上线

对应的枚举

```c#
public enum EnvironmentMode{
    Developing,
    QA,
    Release
}
```

根据枚举执行抽象方法

```c#
public abstract class MainManager : MonoBehaviour
{
    public EnvironmentMode mode;

    private static EnvironmentMode mSharedMode;
    private static bool mModeSetted = false;

    private void Start()
    {
        //不同场景不同mode时只会有一个唯一mode
        if (!mModeSetted)
        {
            mSharedMode = mode;
            mModeSetted = true;
        }
        switch (mSharedMode)
        {
            case EnvironmentMode.Developing:
                LaunchInDevelopingMode();
                break;
            case EnvironmentMode.Test:
                LaunchInTestMode();
                break;
            case EnvironmentMode.Production:
                LaunchInProductionMode();
                break;
        }
    }
    protected abstract void LaunchInDevelopingMode();
    protected abstract void LaunchInTestMode();
    protected abstract void LaunchInProductionMode();

}
```

### GuiManager

#### 加载卸载

通过字典管理，加载和卸载通过存储Panel名字为key，GameObject为Value

```c#
 private static Dictionary<string, GameObject> mPanelDict = new Dictionary<string, GameObject>();
        public static void UnLoadPanel(string PanelName)
        {
            if (mPanelDict.ContainsKey(PanelName))
            {
                var gObj = mPanelDict[PanelName];
                Destroy(gObj);
            }
        }
        public static GameObject LoadPanel(string PanelName, UILayer uILayer)
        {
            var PanelPrefab = Resources.Load<GameObject>(PanelName);
            var PanelObj = Instantiate(PanelPrefab, UIRoot.transform);
            PanelObj.name = PanelName;
            mPanelDict.Add(PanelName, PanelObj);

            switch (uILayer) 
            {
                case UILayer.Bg:
                    PanelObj.transform.SetParent(UIRoot.transform.Find("Bg"));
                    break;
                case UILayer.Common:
                    PanelObj.transform.SetParent(UIRoot.transform.Find("Common"));
                    break;
                case UILayer.Top:
                    PanelObj.transform.SetParent(UIRoot.transform.Find("Top"));
                    break;
            }
            return PanelObj;
        }
```



#### 层级管理

通过枚举分级,在Scene中创建相对应的GameObject来管理对应Panel的层级关系

```c#
public enum UILayer
{
    Bg,
    Common,
    Top
}
```

![1649575865089](ManagerOfManager/1649575865089.png)

在加载时加载到对应层级的GameObject中

```c#
public static GameObject LoadPanel(string panelName, UILayer uILayer)
{
    var canvasObj = GameObject.Find("Canvas");

    var PanelPrefab = Resources.Load<GameObject>(panelName);
    var PanelObj = Instantiate(PanelPrefab, canvasObj.transform);
    
    switch (uILayer)
    {
        case UILayer.Bg: PanelObj.transform.SetParent(canvasObj.transform.Find("Bg"));
            break;
        case UILayer.Common:
 PanelObj.transform.SetParent(canvasObj.transform.Find("Common"));
            break;
        case UILayer.Top:     PanelObj.transform.SetParent(canvasObj.transform.Find("Top"));
            break;
    }
    return PanelObj;
}
```

#### UIRoot

将前面所描述的canvas结构，制作为Prefab，名字为UIRoot，管理所有的UIPanel。

在GUIManager中存储一份，在加载Panel时使用。

![1649577942704](ManagerOfManager/1649577942704.png)

```c#
private static GameObject mPrivateUIRoot;
public static GameObject UIRoot
{
    get
    {
        //懒加载
        if (mPrivateUIRoot == null)
        {
            mPrivateUIRoot = GameObject.Instantiate(Resources.Load<GameObject>("UIRoot"));
        }
        mPrivateUIRoot.name = "UIRoot";
        return mPrivateUIRoot;
    }
}
```

### AudioManager

#### 播放音效

```c#
/// <summary>音效 </summary>
public void PlaySound(string soundName)
{
    CheckAudioListener();

    var doorClip = Resources.Load<AudioClip>(soundName);
    var audioSource = gameObject.AddComponent<AudioSource>();

    audioSource.clip = doorClip;
    audioSource.Play();
}
```

播放背景音

```c#
/// <summary>背景音 </summary>

public void PlayMusic(string soundName, bool IsLoop = true)
{
    CheckAudioListener();
    if (mMusicSource == null)
    {
        mMusicSource = gameObject.AddComponent<AudioSource>();
    }
    var doorClip = Resources.Load<AudioClip>(soundName);
    var audioSource = gameObject.AddComponent<AudioSource>();

    audioSource.loop = IsLoop;
    audioSource.clip = doorClip;
    audioSource.Play();
}
```

### PoolManager

#### 对象池解决的问题

1. 减少new时候寻址造成的消耗，该消耗的原因是内存碎片。
2. 减少Object.Instantiate时内部进行序列化和反序列化而造成的CPU消耗。

#### 简易对象池

获取的操作一般为Allocate(分配)，放入为Recycle(回收)。也有叫Spawn和Despawn

##### 定义池接口

```c#
public interface IPool<T>
{
    T Allocate();
    bool Recycle(T obj);
}
```

在Allocate和Recycle时不在意对象顺序，只需要位置连续，使用stack容器来存储对象。

##### 使用一个简单工厂接口来创建对象。

```c#
public interface IObjectFactory<T>
{
    T Create();
}
```

##### 对象池

```c#
public abstract class Pool<T> : IPool<T>
{
    protected Stack<T> mCachedStack = new Stack<T>();
    protected IObjectFactory<T> mFactory;
    protected int MaxNum = 5;
    public int curCount
    {
        get { return mCachedStack.Count; }
    }
    public virtual T Allocate()
    {
        if (mCachedStack.Count == 0)
        {
            return mFactory.Create();
        }
        return mCachedStack.Pop();
    }
    public abstract bool Recycle(T obj);
    //public virtual bool Recycle(T obj)
    //{
    //    if (mCachedStack.Count >= MaxNum)
    //    {
    //        return false;
    //    }
    //    mCachedStack.Push(obj);
    //    return true;
    //}
}
```

##### 基于工厂接口的简易工厂

```c#
/// <summary>
/// 基于工厂接口的简易工厂
/// </summary>
/// <typeparam name="T"></typeparam>
public class CustomObjectFactroy<T> : IObjectFactory<T>
{
    /// <summary>
    /// 使用委托设置类内部信息
    /// </summary>
    private Func<T> mFactoryMethod;
    public CustomObjectFactroy(Func<T> factoryMethod)
    {
        mFactoryMethod = factoryMethod;
    }
    public T Create()
    {
        return mFactoryMethod();
    }
}
```

##### 包含创建方法的简易对象池

```c#
/// <summary>
/// 简易对象池
/// </summary>
/// <typeparam name="T"></typeparam>
public class SimpleObjectPool<T> : Pool<T>
{
    //重置方法
    Action<T> mResetMethod;
    public SimpleObjectPool(Func<T> factoryMethod, Action<T> ResetMethod = null, int initCount = 0)
    {
        mFactory = new CustomObjectFactroy<T>(factoryMethod);
        mResetMethod = ResetMethod;
        for (int i = 0; i < initCount; i++)
        {
            mCachedStack.Push(mFactory.Create());
        }
    }
    public override bool Recycle(T obj)
    {
        if (mResetMethod != null)
        {
            mResetMethod(obj);
        }
        mCachedStack.Push(obj);
        return true;
    }
}
```

#### 复盘

设计一个比较复杂的模块结构时,可以先设计一个接口(规范)。描述内容有什么

![1649780552285](ManagerOfManager/1649780552285.png)

在mFractroy.Create和Recycle(回收)时，更偏向于自定义的操作 

### LevelManager

```c#
public class LevelManager : MonoBehaviour
{
    //可换成配置表
    private static List<string> mLevelNames;
    public static int Index { get; set; }
    public static void Init(List<string> levelNames)
    {
        mLevelNames = levelNames;
        Index = 0;
    }
    public static void LoadCurrent()
    {
        SceneManager.LoadScene(mLevelNames[Index]);
    }
    public static void LoadNext()
    {
        Index++;
        if (Index >= mLevelNames.Count)
        {
            Index = 0;
        }
        SceneManager.LoadScene(mLevelNames[Index]);
    }
}
```

## 单元测试

Probject内->右键->Create->Testing

创建相对应的c#文件(test模板)
在Test Runner 界面内可以看到所有的测试脚本和对应的方法

类似于自己编写测试用例进行测试

`Assert.AreEqual`叫做断言，在开发大项目时非常有用的工具。测试通不通过取决于断言通不通过。