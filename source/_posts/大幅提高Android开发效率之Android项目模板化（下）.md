title: 大幅提高Android开发效率之Android项目模板化（下）
tags:
  - Android
  - Live Template
categories:
  - Android
date: 2016-11-16 11:02:00
---
大家好，我是光源。

在[《大幅提高Android开发效率之Android项目模板化（上）》](http://www.jianshu.com/p/e8ac0c284601)中我们了解了如何用 Android Studio Template 大幅减少写业务代码前的工作量，同时也稍微提了下用 Live Template 减少写业务代码过程中的“样板式代码”。

可能有朋友会问，我们这么紧张这点效率真的必要么？

这个问题我先不回答，我们先来看看一个场景：写一个单例。

单例模式应该是开发过程中最常见的设计模式之一了。写单例前总得先纠结一下吧，单例模式这么多种实现方式该用哪种好呢？选定了实现方式后，老老实实写一堆代码，然后你会突然发现除了类名不一样外，其他代码都是一模一样，这时你心里会不会隐隐约约觉得这是可以优化的？

我不知道你完成单例模式的实现需要多长时间，**但我告诉你，我们可以用一秒来搞定，你会不会觉得老老实实写代码的自己很傻？**

这就是 Live Template 的第一个作用，**提高编码效率**。

再接着上面的话题，又有人吐槽另一个细节了，一般有点年头的程序员对于单例模式都有自己的一个默认实现方式，不需要花时间去纠结吧？

这话不假，像我就默认使用内部静态类的方式实现单例。但是你有没有想过，项目开发是一个团队协作的过程，就算个人不纠结，但多人开发情况下，要么每个人用自己的一套实现方式造成代码风格不统一，要么同事还是得事先通气、询问用哪种好。（编码规范文档也不会事无巨细把这类问题也事先规定好的）

再思考一下，很多有多方案且不属于编码规范的场景，我们又要怎么去统一呢？

答案还是用 Live Template。例如，Android 开发中，我们应该使 `Message.obtain()` 来获取一个 Message 实例，而不是去 `new` 一个，那么我们完全可以定一个 `getMsg` 的 Live Template，大家只要是用 `getMsg` 的，可以严格保证统一性——这就是 Live Template 的另一个作用了，**保持项目中非编码性质的代码风格统一**。（编码性质的代码风格统一果断用编码规范文档来实现了）

大家看到这里一定已经对 Live Template 有了一定的重视与好奇了吧，别着急，下面上正餐。
<!--more-->
## 一、Live Template 简介
何为 Live template ？直译是“实时模板”，它的机制简单地说就是提前定义好一些通用的代码片段在编写代码时插入编辑器，使用方法类似代码补全，默认快捷方式是 Tab 键（不同平台不同版本可能有差异）。本文将根据 Android Studio 1.5+ 版本介绍 Live template 相关内容。

进入 `Settings/Preferences -> Editor -> Live Templates`，可以看到当前已经有的的模板组。

![图一](http://zhenghuiy-blog.qiniudn.com/lt1.png)

可以看到，默认情况下已经有一些模板存在。

打开分组的某个具体模板，可以看到一个模板包含了缩写、描述、内容三块。

![图二](http://zhenghuiy-blog.qiniudn.com/lt2.png)

在编写代码的过程中输入模板缩写，按下 Tab 键，对应的代码片段就插入到当前位置。

以上就是 Live Template 的简单使用，基本与普通代码补全没啥差别，特别简单实用。

当然，默认的模板虽然不少，但我们不会止步于使用默认的模板，下面说一说如何自定义模板。

## 二、自定义 template

### 第一步 创建新模板
进入 `Settings/Preferences` -> `Editor` -> `Live Templates`，选择某个分组，点击右上角的 “+” 号，选择 Live Template。

### 第二步 选择模板适用范围 
点击下方的 “Define”,选择适用范围。如适用于 class 文件就选 java ，适用于 xml 文件就 xml。

选定了范围后，在范围之外的文件则无法使用该模板。例如 Android XML 分组的“lh”模板选定了 xml，则在 class 文件中是无法使用的。

### 第三步 填写内容
如图二所示，一个模板包括模板缩写、描述、内容三块。

- 模板缩写一方面是模板的名称，另一方面是 Live Template 的触发标识，命名上建议尽量简洁，同时不要与普通类名、方法名等相同。

- 描述是模板的详细描述，在提示中会显示给用户。

- 内容是模板的具体定义，由普通代码和变量组成。

模板内容除了写死的内容外，还可以声明和使用变量。变量形式为 $<variable_name>$ 。通过点击下图中的“Edit variables”按钮打开变量定义窗口我们就可以开始创建当前模板的变量。

![图三](http://zhenghuiy-blog.qiniudn.com/lt3.png)

- Name 变量名，对应 variable_name
- Expression 变量的表现
- Default value  默认值
- Skip if defined 是否跳过编辑

编辑器已经事先定义好 Expression 的一些方法， 部分常用方法如下：

| 方法 | 描述 |
| ---- | ---- |
| annotated("annotation qname") | 注解 |
| arrayVariable() | 集合变量 |
| className(sClassName) | 当前类名 |
| classNameComplete() | 当前完整类名 |
| complete() | 代码补全 |
| componentTypeOf (<array variable or array type>) | 集合内的元素类型 |
| date(sDate) | 系统当前日期 |
| expectedType() | 所需结果的类型，这个在类型转换中特别适用 |
| fileName(sFileName) | 带扩展名的文件名 |
| iterableComponentType(<ArrayOrIterable>) | 可遍历的集合或数组类型 |
| lineNumber() | 行号 |
| methodName() | 方法名 |
| methodReturnType() | 方法返回类型 |
| suggestIndexName() | 建议的索引名（i/j 等） |
| suggestVariableName() | 建议的变量名 |
| time(sSystemTime) | 系统时间 |
| typeOfVariable(VAR) | 变量类型 |
| user() | 当前用户名 |

（详见 [https://www.jetbrains.com/help/idea/2016.1/live-template-variables.html#predefined_functions](https://www.jetbrains.com/help/idea/2016.1/live-template-variables.html#predefined_functions)  ）

## 三、举个例子
众所周知，单例是程序开发中最常用的模式之一，虽然有多种实现方案，但不管选择哪种都避免不了重复写一些“样板代码”，而这种场景就可以用live template 来解决了。接下来我们用创建单例模式的模板来具体说明如何自定义模板。

1. 进入 `Settings/Preferences` -> `Editor` -> `Live Templates`，选择某个分组，点击右上角的“+” 号，选择 `Live Template`。

2. 选定适用范围为 `java`。

3. 填上缩写为 singleTon，描述为单例模式。然后把以下代码拷贝到模板内容中：

```
private static class SingleTonHolder {
    private static final $class$ INSTANCE = new $class$();
}

public static $class$ getInstance() {
    return SingleTonHolder.INSTANCE;
}

private $class$() {
    
}

```

4. 这里定义了 class 变量，需要点击 edit variables 创建该变量，变量名为 class，Expression 选择 className() 。点击 OK 保存。

5. 创建某个class后，输入 `singleTon` ，按下 tab 键就可以看到单例模板。

## 四、Live Template 的导出与导入
第一种方法是，创建完模板后模板数据会以XML的形式存储在 `android studio config\templates` 目录下，window下为  `C:\Users\\<user>\\.AndroidStudiox.x\config\templates` （user为你的计算机用户名）, mac下为
`~/Library/Preferences/AndroidStudiox.x/templates` ，将其中的文件拷贝出即可导出；同理可知，导入时将已定义好的xml文件放到相同目录下，即可完成导入。

第二种方案是，通过 `File` -> `Export Settings` 和 `Import Settings`，只选中 `Live Templates`。

## 五、项目实践

对于具体的项目而言，自定义 Live Template 可以分成两部分，一部分是相对通用的模板，作为默认模板的补充，比如上文中的单例模式模板；另一部分是跟项目逻辑深度相关的模板，比如使用项目中自定义的`LogUtil`而不是普通的`Log`来打 log 。

上文中已经给出自定义通用模板的例子，这里再给出项目相关模板的示例。

例如，项目内部有 toast 工具类来显示一个 toast，定义一个名为“xxtoast”的模板(避免与普通toast冲突，加上前缀)，内容是
```
    ToastUtil.makeShortToast($strRes$);
```

最后，将这些自定义模板导出放到 gitlab 上，其他同事只需要将这些文件 clone 到对应目录下，即可直接使用。

### 小工具
有一个叫做 [exynap](http://exynap.com/) 的 Anroid Studio 插件可以很方便搜索、创建、使用代码片段（与 Live Template 不共享），感兴趣的朋友可以看看。

## 六，参考资料
https://www.jetbrains.com/help/idea/2016.2/live-templates.html

![](http://zhenghuiy-blog.qiniudn.com/qrcode_for_gh_8d80345d7ff8_430.jpg)
