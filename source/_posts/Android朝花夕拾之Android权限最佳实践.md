title: Android朝花夕拾之Android权限最佳实践
tags:
  - Android
  - 朝花夕拾
  - Android进阶
  - Android高级
categories:
  - Android朝花夕拾
date: 2016-05-20 20:25:00
---
## 前言

大家好，我是光源。

从 Android6.0 开始Android的权限模式有了一番更改，从安装时一股脑列给用户，到运行时动态申请权限。对于 Android开发者而言，这是一个重要的变更。

讨论这个问题之前，我们得先检查一下项目的 target version。如果是 23及以上，则必须得适配新权限模式；如果是 23之下，则还是统一在安装时全部申请权限——即便如此，使用Android 6.0及以上系统的用户还是可以在设置中去关闭你的某些权限。当权限被关闭时，不会导致你的应用直接崩溃，但是会导致你获取到的返回值为null 或者 0，这个是需要注意的地方。

以下是 Android 官网给出的最佳实践，草草翻译，有不妥之处还请斧正。

## 正文

app很容易用海量的权限请求淹没用户。如果用户发现 app 影响使用或者担忧 app 滥用用户个人信息，他们可能不再使用或者完全删除这个app。以下最佳实践能帮助你避免这种糟糕的用户体验。
<!--more-->
## 优先使用 Intent

在很多情况下，你都可以选择两种方式来为你的app完成任务。你可以让你的app去请求权限来自己完成操作，或者你也可以让 app 利用 intent 让其他 app 进行这项工作。

举个例子假如你的 app 需要用设备中的相机来拍照，你的 app 可以请求 `CAMERA` 权限 以直接使用相机组件，然后你的 app 将使用相机组件接口去控制相机并拍照。这个方法使你的 app 有完整的摄影进程控制权，让你将相机UI纳入app中【译者注：即可以完全自定义相机UI】。

然而，如果你不需要这么完整的控制，你可以使用  `ACTION_IMAGE_CAPTURE` intent 去请求图片。当你发送这个intent，系统让用户选择一个相机 app（如果还没有默认的相机 app）。 用户用选中的相机 app 拍照后，它将照片反回给你的 app 的  `onActivityResult()` 方法。

同样得，如果你需要打电话、读写用户的通讯录等等，你可以通过创建合适的 intent 来实现或者请求权限后直接访问合适的组件。二者各有利弊。

如果你使用权限：

- 你的 app 在执行该操作时对用户体验有完整的掌控。但是这么宽泛的控制增加了任务的复杂度，因为你需要自行设计合适的UI。
- 不管在运行时还是刚安装时，用户倾向于只进行一次权限操作，此后，你的 app 就可以进行操作无需用户额外授权。但是，如果用户没有允许权限或者后来取消了，你的 app 将一直无法进行该操作。

如果你使用 intent：

- 你不需要为该操作设计UI。处理 intent 的 app 提供了UI。然而，这意味着你无法掌控这部分用户体验。用户可能跟你一无所知的 app 打交道。
- 如果用户没有处理该操作的默认应用，系统会让用户选择一个。如果用户没有指定一个默认处理，每次进行该操作都会弹出额外的选择对话框。

## 只申请必须的权限

你每次申请权限都在迫使用户做决定，所以你应该减少发出这些请求的次数。如果用户使用的是 Android 6.0 及之后的系统，每次用户使用某些 app 中需要权限的特性，app 就必须因申请权限而打断用户。如果用户使用 6.0之前的系统，在安装 app 时用户必须得允许每一个权限。如果权限清单太长或者看起来不合适，用户可能会决定停止安装你的 app。因为以上种种，你最好减少申请的权限数量。

## 不要压垮用户

如果用户使用 Android6.0及之后的系统，在运行 app时需要授权。如果你把一大堆权限申请一次性抛给用户，你会把他们压垮进而删除你的应用。因而你应该只在必要时才去请求权限。

在某些情况下，对你的 app 而言一个或多个权限是完全必要的，在 app 启动时就去请求这些权限是有意义的。比如你做了一个摄影 app，它需要使用相机。当用户第一次启动该 app 时，他们自然不会惊讶请求相机的权限。但是如果这个app还有通过用户的通讯录分享图片的功能的话，你就不应该在第一次启动时获取 `READ_CONTACTS` 权限。而应该等到用户使用”分享“功能时再请求权限。

## 解释你为何需要权限

当你调用[requestPermissions()](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions(android.app.Activity, java.lang.String[], int)) 表明你的 app 需要什么权限时系统会显示权限对话框，但是没有说为什么。在某些情况下，用户会感到困惑。在调用[requestPermissions()](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions(android.app.Activity, java.lang.String[], int)) 前向用户解释你的 app 为何需要这些权限是个好主意。

比如，一个拍照 app 可能会需要定位服务以便给图片打上地理位置标签。一个普通用户可能无法理解一张图片可以包含位置信息，然后会困惑为什么拍照 app 会需要知道他的位置。所以在这种情况下，app 在调用[requestPermissions()](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions(android.app.Activity, java.lang.String[], int)) 前告诉用户这个特性是个好主意。

告知用户的一个不错的方法是把这些权限请求合并进 app 引导中。app 引导可以依次介绍 app 的特性，在做这些的同时，可以解释需要哪些权限。例如，拍照 app的引导可以示范”通过通讯录分享照片“的功能，告知用户他们需要授权 app 查看用户的通讯录。然后该 app 可以调用[requestPermissions()](https://developer.android.com/reference/android/support/v4/app/ActivityCompat.html#requestPermissions(android.app.Activity, java.lang.String[], int)) 请求读取通讯录。当然，不是所有用户都会跟从引导，所以你依然需要检查确认并且在 app 的日常使用中请求权限。

## 测试两种权限模式

从 Android 6.0 开始，用户在应用运行时允许或者拒绝权限而不是在应用安装时，这导致你需要测试各种条件下app 的表现。

在 Android 6.0之前，你可以合理假定你的 app 一直可运行，它拥有所有在 app manifest中声明的权限。在新的权限模式下，你不能这么认为了。

以下提示将帮助你验证 Android 6.0及以上系统的权限相关代码问题。

- 验证 app 当前权限以及相关代码路径。

- 测试用户通过权限保护服务和数据。

- 测试多种权限分别被允许、拒绝的组合情况。比如，相机 app 可能在 manifest 文件中声明了`CAMERA` 、 `READ_CONTACTS` 和 `ACCESS_FINE_LOCATION` 权限。你应该测试每个权限打开和关闭的情况来确保 app 可以优雅地处理所有权限配置情况。记住，从 Android 6.0开始用户可以打开或关闭任意一个app的权限，即便是 targets API 在 22及以下的。

- 使用 adb 工具通过命令行管理权限：

  - 以组的形式列出权限和状态：

    ````shell
    $ adb shell pm list permissions -d -g
    ````

  - 允许或拒绝一个或多个权限：

    ````shell
    $ adb shell pm [grant|revoke] <permission-name> 
    ````

- 为使用权限的服务分析 app

## 后记

以上是 Android 官网给出的最佳实践。个人觉得这篇文章主要从用户体验角度来阐述权限的最佳实践，还缺乏代码层面的实践，因此扩展一下。

## 权限相关代码最佳实践

**第零步，在manifest 文件中声明所需的权限。**

**第一步，检测是否拥有权限。**

|   API 23 or above   |             all version             |
| :-----------------: | :---------------------------------: |
| checkSelfPermission | ContextCompat.checkSelfPermission() |

被授权函数返回`PERMISSION_GRANTED`，否则返回`PERMISSION_DENIED` 。

**第二步，检测是否需要显示申请权限对话框。**如果第一步返回没有权限的话，则我们需要去申请权限，但是申请权限之前，需要显示查看用户是否已经禁止了申请权限对话框，如果禁止的话，我们用第三步的代码将没有任何效果。

| API 23 or above                      | all version                              |
| ------------------------------------ | ---------------------------------------- |
| shouldShowRequestPermissionRationale | ActivityCompat.shouldShowRequestPermissionRationale  / FragmentCompat.shouldShowRequestPermissionRationale |

返回为 `true` 或 `false` ，在 6.0以下恒定为 `false` ，这个不用多解释了吧，在 6.0以下还没有系统默认对话框存在，肯定不弹了。 

**第三步，显示提示框去申请权限。**

| API 23 or above    | all version                              |
| ------------------ | ---------------------------------------- |
| requestPermissions | ActivityCompat.requestPermissions/ FragmentCompat.requestPermissions() |

发出请求后，会在 onRequestPermissionsResult 方法中获得回调结果。

以目前 Android系统的占有率分布，最低建议支持4.0，所以我们可以直接使用兼容库中的三个方法以兼容所有版本。下面是一个简单的需求，向SD卡中写入数据：

````java
	/**
     * 向SD卡插入数据
     */
    private void insertDataToSDCard() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED) {
            //拥有权限
            //TODO 向SD卡写数据
        } else {
            //没有权限,判断是否会弹权限申请对话框
            boolean shouldShow = ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
            if (shouldShow) {
                //申请权限
                ActivityCompat.requestPermissions(this, new String[] {Manifest.permission.WRITE_EXTERNAL_STORAGE}, REQUEST_CODE_REQUEST_PERMISSION);
            } else {
                //被禁止显示弹窗
                //TODO 显示对话框告知用户必须打开权限
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == REQUEST_CODE_REQUEST_PERMISSION) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                //permission granted
                //TODO 向SD卡写数据
            } else {
                //permission denied
                //TODO 显示对话框告知用户必须打开权限
            }
        }
    }
````

可见，通过兼容库去做权限模式兼容还是挺简单明了的。考虑到扩展性，完全可以把权限判断逻辑写成一个通用的工具类。

最后，推荐一个第三方库[hotchemi’s PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)，这个也是参考资料中看到的一个不错的第三方库，有兴趣可以看看。

## 参考资料

1. [Permissions Best Practices](https://developer.android.com/training/permissions/best-practices.html)
2. [Android M 新的运行时权限开发者需要知道的一切](http://jijiaxin89.com/2015/08/30/Android-s-Runtime-Permission/)