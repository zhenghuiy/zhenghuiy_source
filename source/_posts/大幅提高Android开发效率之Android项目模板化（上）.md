title: 大幅提高Android开发效率之Android项目模板化（上）
tags:
  - Android
  - template
categories:
  - Android
date: 2016-11-08 13:06:00
---
## 一、两个场景的思考
大家好，我是光源。

首先思考一个最普通的场景：创建一个 `Activity`。你需要做的是：

1、 创建 `Activity` 类文件；
2、 创建对应的 `Layout` 布局文件；
3、 在 `AndroidManifest.xml` 注册；

当然现在 `MVP` 模式(`mvp` 模式有多种实现方式，这里选择一种普遍的)基本成为标配，所以你还需要接着：

4、 对应的 `View interface`;
5、 对应的 `Presenter interface`；
6、对应的 `View` 实现；
7、对应的 `Presenter` 实现；
8、对应的 `Model` 类；

当你气喘吁吁地做好以上工作后，你又想起这个页面你需要用 `RecyclerView` 实现，所以你继续：
9、在布局中加入 `RecyclerView`;
10、设置 `LayoutManager`；
11、创建 `RecyclerView Adapter`；
······
终于，你可以开始写具体的业务代码了，而此刻距刚开始已经过去了 N 首歌的时间。
 
开始写代码后，我们再思考一下这样的场景：显示一个 `Dialog` 。
这时一般有两种情况，一种是你需要从零开始构造一个对话框、设置样式、编写逻辑等等，这与上面一个场景一样，一大堆的样板式代码；另一种情况是部门内已经对 `Dialog` 的设置样式、构造等代码做了封装，比较常见的方式会像这样：
```
DialogUtil.show(...);
```
那么你直接调用就行。但这同样存在“无脑写代码”的问题，以及对于部门内的新人来说，可能不一定会遵循这种使用封装好的代码的约定。

看到这里，假如你有一点点感同身受的话，那么恭喜你，本文将教你如何解决这些问题。如果没有共鸣也没关系，看完本文，你也会有一定的收获——实际上对于所有还不知道 Android 模板相关内容的开发者而言，看完本文都能大幅提升项目开发效率，这也是本文标题的由来。
<!-- more -->
## 二、Android Studio Template
针对第一个场景，我们可以通过自定义 Android Studio Template 来解决。

### 1. 简介
在我们创建项目时，在最后一步可以选择一种预设的 `Activity` 作为项目的第一个 `Activity`，如下图

![图一](http://huihuitest.u.qiniudn.com/at1.png)

这就是 Android Studio Template 的初次登场。Android Studio Template 依靠 FreeMarker 引擎，将事先定义好的模板文件生成我们所需的 class 文件、layout 文件等等，可以极大减少样板式代码的编写。

打开 Android Studio 的安装目录，进入到模板所在的文件夹。关于模板位置，Windows 的路径在 `/plugins/android/lib/templates/`，Mac 下是  `Android Studio.app/Contents/plugins/android/lib/templates/`。可以看到，默认已经实现了一些模板。

![图二](http://huihuitest.u.qiniudn.com/at2.png)

### 2. 模板使用流程与模板结构介绍
我们以相对简单典型的 `EmptyActivity`为例进行讲解。

![图三](http://huihuitest.u.qiniudn.com/at3.png)

从上图我们可以看到 EmptyActivity 模板有以下文件：
- globals.xml.ftl
- recipe.xml.ftl
- template.xml
- template_blank_activity.png
- root/src/app_package/SimpleActivity.java.ftl

在这里我并不准备直接给出各个文件的介绍，我知道那样并不容易理解，毕竟我是在写博客而不是写文档。我们先看看模板的使用流程。

第一步，选择 File(或者在某个包上右键) -> New，可以看到菜单最下方出现了一列模板分类，我们选择 Activity 分类下的 EmptyActivity，点击。

![图四](http://huihuitest.u.qiniudn.com/at4.png)

第二步，将出现的弹框填写完整，点击完成即可生成文件。

![图五](http://huihuitest.u.qiniudn.com/at5.png)

这时我们打开模板文件夹中的 `template.xml` 文件，文件内容如下：

![](http://zhenghuiy-blog.qiniudn.com/empty_template.png)

看到这里，再联系上图中的元素，聪明的你一定已经知道 `template.xml`的作用了。`template.xml`是整个模板的定义，从模板名称到模板分类再到模板参数都在这里定义，功能有点像 Android 项目中的 `AndroidManifest.xml`。下面详细介绍它的主要内容。
1.  `template` 标签，整个 `xml` 的根节点，其中 `name`属性为模板名称，`description`属性为模板的描述。
2. `category`标签，定义了模板所属的分类，即图四中的分类列表中的分类，分类名一样的模板会被归纳到同一目录下。
3. `parameter` 标签，定义了模板输入弹窗中的输入参数，每个 `parameter` 为一行。其中：
  - `id` 属性为参数唯一标识，我们可以在代码中通过 `id`来使用该参数。
  - `name` 属性为参数名称，显示在输入控件的前面或后面。
  - `type`属性为参数类型，根据该属性和`constraints`属性的值综合比较后参数会被渲染成不同的输入形式，比如 `string` 类型会显示输入框，而 `boolean`类型会显示一个选择框。
  - `constraints`属性为输入约束，常见的有`class`，代表类名；`layout`代表布局名；`package` 代表包路径； `unique`则是不能与现有的重复；`nonemptye`表示不能为空。
  - `suggest` 和 `default`标签，前者是建议名称，后者是默认名称，前者优先级高于后者。
  -  `help`属性是参数输入提示，当该参数获取焦点后，对应的帮助信息会显示在对话框上。
4. `thumbs`标签定义了该目标的缩略图，这也就是`template_blank_activity.png`文件的作用了。
5. `globals`标签指定了 global 文件，具体用途下文会具体说明。
6. `execute`标签，跟字面上的意思一样，执行 `recipe.xml.ftl`文件的内容，将模板文件生成具体的可用文件。

了解了 `template.xml` 文件后，我们再看看 `globals.xml.fl`文件：
```
<?xml version="1.0"?>
<globals>
    <global id="hasNoActionBar" type="boolean" value="false" />
    <global id="parentActivityClass" value="" />
    <global id="simpleLayoutName" value="${layoutName}" />
    <global id="excludeMenu" type="boolean" value="true" />
    <global id="generateActivityTitle" type="boolean" value="false" />
    <#include "../common/common_globals.xml.ftl" />
</globals>
```
整个文件很简单，用 `global` 标签定义了一系列的全局参数（这里的全局是指可以在其他模板文件中使用它们）供具体的模板文件使用，`id`为唯一标识，`type`为类型，`value`为参数值。

至于`#include`标签的作用，相信大家能猜到一点，与 `xml` 文件中的 `include`作用是一致的，包含其他文件内容到本文件中。包含的这个文件具体内容我就不贴出来了，也是一堆定义好的参数，感兴趣的可以打开对应的文件看看。

总结一下，`globals.xml.fl`文件的作用就是定义或引入一些变量（参数）供后续模板文件使用。

我们讲完了三个文件的作用后，相信大家对 Android Studio Template 已经有一定了解，在讲解 `recipe.xml.ftl` 文件和 `SimpleActivity.java.ftl` 文件之前，我们先了解一下 Android Studio Template 的工作原理。看下图：

![图六](http://huihuitest.u.qiniudn.com/at6.png)
> 图片来自：http://www.slideshare.net/murphonic/custom-android-code-templates-15537501

如图所示，整个模板是以 `FreeMarker` 引擎为基础工作。其中，`globals.xml.ftl`和模板的输入参数弹窗（ `template.xml` 中定义的 
`parameter` 标签）相当于声明参数和定下参数值，再执行 `recipe.xml.fl` 中定义好的命令，将 `SimpleActivity.java.ftl` 转化为 `SimpleActivity.java`。转化过程中自然免不了使用定义好的参数。

说到这里，我们知道了 `recipe.xml.ftl` 文件的作用就是定义模板转化的命令。看看具体文件内容：
```
<?xml version="1.0"?>
<recipe>
    <#include "../common/recipe_manifest.xml.ftl" />

<#if generateLayout>
    <#include "../common/recipe_simple.xml.ftl" />
    <open file="${escapeXmlAttribute(resOut)}/layout/${layoutName}.xml" />
</#if>

    <instantiate from="root/src/app_package/SimpleActivity.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${activityClass}.java" />

    <open file="${escapeXmlAttribute(srcOut)}/${activityClass}.java" />
</recipe>
```
首先，`<#include>`标签那行表示包含了 `recipe_manifest.xml.ftl` 文件的内容，打开对应文件后会发现里面是 `<merge>` 标签，作用是将定义的 `manifest.xml.ftl` 文件转化为 `manifest.xml`后与项目中的 `AndroidManifest.xml` 文件合并（可以参考安卓开发中 xml 文件中的 `merge` 标签）。

也就是通过 `merge` 标签，我们可以自定义一个同名文件，然后与项目中的文件合并。这里是完成了 `Activity` 在 `AndroidManifest.xml` 文件中注册的工作。

然后下面的 `<#if>`标签代表的含义相信大家可以猜测出来。形如`<#if condition> </#if>`，是 freeMarker 模板中的`if`语句，前后成对出现，`condition`常见的有 `boolean`、`var == "content"`(判断变量的值)、`var??`（判断是否为空）等等。

有 `if` 当然少不了 `else`，形如`<#if condition> <#else> </#if>`。

`instantiate` 标签是 `recipe.xml.ftl` 文件的核心标签，它的作用是将 `from` 属性的 ftl 文件转化为 `to` 属性的文件。例如文件中就是将 `SimpleActivity.java.ftl` 文件转化为对应包下的 `${activityClass}.java` 文件。（这里的 `${activityClass}`是模板变量的表示方法，activityClass 变量由上文中的 `template.xml`定义，最后会被它的值填充）

最后，`open` 文件会打开对应的文件。

接下来上“主菜”，打开 `SimpleActivity.java.ftl` 看看：
```
package ${packageName};

import ${superClassFqcn};
import android.os.Bundle;

public class ${activityClass} extends ${superClass} {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
<#if generateLayout>
        setContentView(R.layout.${layoutName});
</#if>
    }
}
```
看起来就是一个普通的 Java class 文件，区别是多了一些 FreeMarker 标记。在 `recipe.xml.ftl` 文件定义的转化过程中，这些标记会被来自 `globals.xml.ftl` 和 `template.xml` 的参数填充为具体的值。

### 3. 举个例子
接下来我们该一展身手动手实践了。

对于 `MVP` 模式我的第一感受是，爽则爽矣，然而创建一个界面需要做的事情太多了！那么这里就以 `MVP`模式为例子将它模板化，将它降低的工作效率找回来。

如下图，用 [Google MVP samples](https://github.com/googlesamples/android-architecture) 的 `MVP` 模式实现写了一个简单的例子。这里已经将具体的业务代码剥离，留下的是纯粹的“必须存在的代码”。现在我们将它模板化。

![图七](http://huihuitest.u.qiniudn.com/at7.png)

说到写一个模板，常规的做法是根据上面说到的模板文件从 `globals.xml.ftl` 开始一个一个编写，但这种做法个人觉得并不好。特别是对于新手而言，很可能会处于一种打开文件一片茫然不知怎么下手的状态。

我这边提倡的做法是，**将模板生成过程逆向后就是写模板的过程**。接下来我们将一步一步实现这个模板。
#### 步骤一、找到并整理模板所需的文件
我们要制作一个模板，首先得知道我们要用模板去做何种工作。在项目中找到这类工作最典型的一个实现，将它做成模板。
我们先将业务代码、不必要的代码剥离，找到真正所需的几个文件（这一步上述的例子中已经整理好）。我们需要的是：
- MainActivity
- MainFragment
- MainPresenter
- MainContract
- activity_main
- fragment_main
- AndroidManifest.xml (`Activity` 需要注册)

这些文件。

#### 步骤二、整理出变量（参数）
这里以代码量最多的 `MainFragment` 为例：
```
package name.huihui.androidstudiotemplatetest.main;

import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import name.huihui.androidstudiotemplatetest.R;

public class MainFragment extends Fragment implements MainContract.View {
    private MainContract.Presenter mPresenter;

    public static MainFragment newInstance() {
        MainFragment fragment = new MainFragment();
        return fragment;
    }

    @Override
    public void onResume() {
        super.onResume();
        mPresenter.start();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View root = inflater.inflate(R.layout.fragment_main, container, false);
        setHasOptionsMenu(true);

        return root;
    }

    @Override
    public void setPresenter(@NonNull MainContract.Presenter presenter) {
        mPresenter = presenter;
    }

}
```
1. 从上往下看，第一行语句就是指定包路径。从例子中的项目结构看 `MainActivity` 是放置于 `main` 包下的，同理，我们以后创建其他 `Activity` 也会将它放到其他包下。所以第一个必须的变量就是包名，一般取名为 `packageName`。
2. 接下来一些 `import` 语句没啥可说，看这一句 `public class MainFragment extends Fragment implements MainContract.View` 显然 `fragment` 名称和 `contract` 契约类名称是需要由使用者填写的，分别定义变量为 `fragmentName`和`contractName`。
3. 定义好 `fragmentName`和`contractName` 后，再往下看到 `MainFragment` 和 `MainContract` 自然而然用前者去替换就好。
4. 下面看到需要变更的就是 `fragment_main` 布局文件名了，定义为 `contentLayoutName`。
5. 最后一步，用上面定义好的变量名去替换需要变更的代码，就是模板文件了，将模板文件命名为 `Fragment.java.ftl`。

> 小提示：还记得么，FreeMarker中变量的表示方法为 ${name}

修改完的 `Fragment.java.ftl` 内容如下
```
package ${packageName};

import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import name.huihui.androidstudiotemplatetest.R;


public class ${fragmentName} extends Fragment implements ${contractName}.View {
    private ${contractName}.Presenter mPresenter;

    public static ${fragmentName} newInstance() {
        ${fragmentName} fragment = new ${fragmentName}();
        return fragment;
    }

    @Override
    public void onResume() {
        super.onResume();
        mPresenter.start();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View root = inflater.inflate(R.layout.${layoutName}, container, false);
        setHasOptionsMenu(true);

        return root;
    }

    @Override
    public void setPresenter(@NonNull ${contractName}.Presenter presenter) {
        mPresenter = presenter;
    }

}

```
其他文件同理。最终我们整理出定义的变量为：
- packageName，包名
- fragmentName，`Fragment` 名
- contractName，`MVP`契约类名（Google这种实现方式的类，不了解含义也无伤大雅）
- contentLayoutName，`Fragment` 的布局文件名
- presenterName，`Presenter`名
- activityName，`Activity`名
- activityLayoutName，`Activity`的布局文件名

整理出的文件为：
- Fragment.java.ftl
- Activity.java.ftl
- activity.xml.ftl
- Contract.java.ftl
- fragment.xml.ftl
- Presenter.java.ftl
- AndroidManifest.xml.ftl

#### 步骤三、将整理出的模板文件放置到正确的路径下
模板文件夹下有个 `root` 文件夹，我们仿造 `EmptyActivity` 模板：
1. 将 `.java.ftl`文件放到 `root/src/app_package/`路径下
2. 将 `AndroidManifest.xml.ftl`放到 `root/`路径下
3. 将 `.xml.ftl`文件放到 `root/res/`路径下

#### 步骤四、写出 template.xml 
有了所需的变量，我们就可以对照着上文中的 `EmptyActivity` 模板写出我们需要的`template.xml` 文件了。

在`EmptyActivity` 模板同目录下创建一个与 `EmptyActivity` 模板文件目录基本一致的新模板文件（拷贝黏贴大法好），命名为`MvpActivity`。

最终内容如下：
```
<?xml version="1.0"?>
<template
    format="5"
    revision="5"
    name="MVP Activity"
    minApi="7"
    minBuildApi="14"
    description="Creates a new mvp activity">

    <category value="Test" />
    <formfactor value="Mobile" />

    <parameter
        id="activityName"
        name="Activity Name"
        type="string"
        constraints="class|unique|nonempty"
        suggest="${layoutToActivity(layoutName)}"
        default="MainActivity"
        help="The name of the activity class to create" />

    <parameter
        id="fragmentName"
        name="Fragment Name"
        type="string"
        constraints="class|unique|nonempty"
        default="MainFragment"
        help="The name of the fragment class to create" />

    <parameter
        id="contractName"
        name="Contract Name"
        type="string"
        constraints="class|unique|nonempty"
        default="MainContract"
        help="MVP 契约类" />

    <parameter
        id="presenterName"
        name="Presenter Name"
        type="string"
        constraints="class|unique|nonempty"
        default="MainContract"
        help="Presenter" />

    <parameter
        id="contentLayoutName"
        name="Fragment Layout Name"
        type="string"
        constraints="layout|unique|nonempty"
        default="fragment_main"
        help="The name of the layout to create for the fragment" />

    <parameter
        id="activityLayoutName"
        name="Activity Layout Name"
        type="string"
        constraints="layout|unique|nonempty"
        suggest="${activityToLayout(activityName)}"
        default="activity_main"
        help="The name of the layout to create for the activity" />
    
    <parameter
        id="packageName"
        name="Package name"
        type="string"
        constraints="package"
        default="com.mycompany.myapp" />

    <!-- 128x128 thumbnails relative to template.xml -->
    <thumbs>
        <!-- default thumbnail is required -->
        <thumb>template_blank_activity.png</thumb>
    </thumbs>

    <globals file="globals.xml.ftl" />
    <execute file="recipe.xml.ftl" />

</template>

```
模板文件就不一一解释，有几点需要说明一下：
1. 为了更好地组织模板，建议将 `category`的值改为自己的项目名称，这样假如有针对其他项目创建的模板不会导致混乱。
2. `thumbs` 标签里的图片可以换成对应页面的截图。

#### 步骤五、`globals.xml.ftl` 和 `recipe.xml.fl`
`globals.xml.fl` 没啥好说的，保持原样即可（当然你也可以选择将不用的参数删除），来看看 `recipe.xml.fl` 文件。

为了方便大家理解，这里我们不去 `include`，自己写 `AndroidManifest.xml` 的 `merge`，从零开始构建。最终内容如下：
```
<?xml version="1.0"?>
<recipe>
    <merge from="root/AndroidManifest.xml.ftl"
    	to="${escapeXmlAttribute(manifestOut)}/AndroidManifest.xml" />

    <instantiate from="root/src/app_package/Activity.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${activityName}.java" />

    <instantiate from="root/src/app_package/Fragment.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${fragmentName}.java" />

    <instantiate from="root/src/app_package/Contract.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${contractName}.java" />

    <instantiate from="root/src/app_package/Presenter.java.ftl"
                   to="${escapeXmlAttribute(srcOut)}/${presenterName}.java" />

	<instantiate from="root/res/fragment.xml.ftl"
		to="${escapeXmlAttribute(resOut)}/layout/${contentLayoutName}" />

	<instantiate from="root/res/activity.xml.ftl"
		to="${escapeXmlAttribute(resOut)}/layout/${activityLayoutName}" />

    <open file="${escapeXmlAttribute(manifestOut)}/AndroidManifest.xml" />

    <open file="${escapeXmlAttribute(srcOut)}/${activityName}.java" />

    <open file="${escapeXmlAttribute(srcOut)}/${fragmentName}.java" />

    <open file="${escapeXmlAttribute(srcOut)}/${contractName}.java" />

    <open file="${escapeXmlAttribute(srcOut)}/${presenterName}.java" />

    <open file="${escapeXmlAttribute(resOut)}/layout/${contentLayoutName}" />

    <open file="${escapeXmlAttribute(resOut)}/layout/${activityLayoutName}" />
</recipe>

```
创建一个注册有 `Activity` 的 `AndroidManifest.xml`文件并与原项目中的合并、将模板文件转化为我们需要的文件并在最后打开它们。

#### 步骤六、测试模板
重启 Android Studio，让模板生效。

假设现在有一个需求是创建一个 `DetailActivity`，按照实例工程中 `MainActivity` 的结构，我们需要创建多个文件，现在我们用 `MVPActivity` 模板来做这些事。

创建一个 `detail` 包路径，选中它右键选中模板，如下图：

![图八](http://huihuitest.u.qiniudn.com/at8.png)

再按下图填写好所需参数，即可生成所需文件。

![图九](http://huihuitest.u.qiniudn.com/at9.png)

**测试项目和测试模板的代码已放到 github 上: [AndroidStudioTemplateTest](https://github.com/zhenghuiy/AndroidStudioTemplateTest)**
## 三、Android Studio Template 项目实践与不足
啰啰嗦嗦写了一堆终于把 Android Studio Template 简单介绍一下了，可能看文章时会觉得有点复杂有点麻烦，但是真正上手写出自己的模板后你就会觉得，原来这么简单！

### 1. 怎么做
说了这么多，那到底我们要怎么将它运用到项目中呢？很简单，**将项目中最常见、最显得重复的类模板化**。

举个例子，本文开头的第一个场景：每次创建一个 `Activity` 我们都需要各种创建文件等等，我们就可以利用模板来解决这些工作。虽然 Android Studio 默认已经提供了 `EmptyActivity`模板，但实际开发中绝大多数项目的 `Activity` 都是需要继承自定义的 `BaseActivity`，然后做点其他小逻辑，那这时我们就可以自己定义一个 `XXEmptyActivity`。

再举个例子，上文提到的 `MVP` 模式在创建一个 `Activity` 时需要创建大量里面有固定代码的文件，完全可以由模板代劳。点点鼠标轻松解决 `MVP` 带来的麻烦，岂不快哉。

为什么我们需要自定义模板？因为我们需要的是“接地气”、与项目风格一致的模板，这才是关键。

### 2. 注意事项
有一点需要特别提醒：Android Studio Template 的功能只是利用模板将那些“固定的代码”由无脑手写改为自动生成，它不会减少代码也不会优化代码。

因此使用时我们要先分清当前场景是代码写的不够好、代码没有封装优化还是真正需要模板化。

具体怎么区分？保持一个原则，**只将不得不做的工作交给模板**。

什么是不得不做的事？在写代码前先创建对应的文件是不得不做的事、在 `AndroidManifest.xml` 注册 `Activity`是不得不做的事、`Activity` 里需要在 `onCreate` 方法里去 `setContentView`也是不得不做的事，等等。

而像“在 Activity 里显示 `ToolBar`”这种需求完全可以封装出一个 `BaseToolBarActivity` 出来，纯粹是代码层面的封装优化了。

### 3. 不足
我们简单地将日常写代码的过程划分为以下几个阶段：
1. 创建文件。
2. 写样前置代码（就是上文提到的“不得不做的工作”）
3. 写业务代码

显然，利用 Android Studio Template 我们可以解决第一和第二阶段的大部分工作，而且你的项目架构越是结构优良越能从众得益（好的架构总是结构清晰）。

但是不足之处体现在 Android Studio Template 的粒度是文件级别，在文件创建完毕后也就是到了第三阶段，它就无能为力了。而恰恰是在我们写代码的过程中，也极其容易产生一些“样板式”代码。

比如 xml 文件中我们总是需要为每个控件“不厌其烦”地写 
```
android:width="match_parent"
android:height="wrap_content"
```
之类的代码。

它们当然是必要的，但是有没有什么办法可以让我们免于把时间花费在敲这些代码上呢？

答案是使用 Live Template。

具体介绍将会在《大幅提高Android开发效率之Android项目模板化（下）》中给出，同时本文的下半部分也会对 Android 项目模板化做一个总结。**感兴趣的朋友请关注我的微信公众号得到第一时间的通知。**

![](http://zhenghuiy-blog.qiniudn.com/qrcode_for_gh_8d80345d7ff8_430.jpg)

## 四、参考资料
1. [Android Studio自定义模板 写页面竟然可以如此轻松](http://blog.csdn.net/lmj623565791/article/details/51635533)
2. [https://developer.android.com/studio/projects/templates.html](https://developer.android.com/studio/projects/templates.html)