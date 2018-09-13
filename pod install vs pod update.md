来源于 https://guides.cocoapods.org
原文地址：https://guides.cocoapods.org/using/pod-install-vs-update.html
仅供学习,如有转摘,请注明出处.
***
## 介绍
很多开始使用CocoaPods的人认为`pod install`仅仅是首次对项目进行CocoaPods设置时使用，而后续使用`pod update`就行了，然后并非如此。

本指南的目的在于解释如何适时的使用`pod install`与`pod update`。

##### TL;DR:
- 无论是对已存在`Podfile`文件的（项目）运行过`pod install`，还是仅仅对已经使用CocoaPods的项目进行添加或者移除pods操作。（都是）执行`pod install`在项目中去安装新的pods。

- 只有在你想把pods更新到更新的版本时才需要执行`pod update`。

### pod install
首次对项目设置pods，或者每次对（项目中的）`podfile`进行添加、更新、移除pod等操作执行该命令
- 每次运行`pod install`，（它）都会根据`Podfile.lock`中的每个pods进行下载和安装。`Podfile.lock`文件跟踪了每个已安装的pods版本并进行 锁定。
- 运行`pod install`时，它只会解决`Podfile.lock`中尚未列出的pods 依赖。
    -  对于`Podfile.lock`中列出的pods，无需尝试检查是否存在新的版本，直接对指定的pods版本进行下载。

    - 对于`Podfile.lock`中尚未列出的pods，先搜索与`Podfile`中描述的相匹配的pods版本（例如 指定版本 1.2 的 ‘MyPod’）。

### pod outdated
执行`pod outdated`，CocoaPods会列出所有比`Podfile.lock`中已存在pods的新版本。这意味着，如果对这些（存在新版本的）pods执行`pod update PODNAME` 命令，它们都会被更新 - 前提是这些新版本仍然与`Podfile`中的限制相匹配。
**说明：如果Podfile中的版本设置方式为 ～> x.y 则会更新到  【x.最大版本号】，如果版本设置为 = x.y，则不会更新**

#### pod update
当执行`pod update PODNAME`，CocoaPods会尝试寻找该pod的一个更新版本，不再考虑`Podfile.lock`中列出的版本。它会尽可能将pod更新到最新版本【前提是这些新版本仍然与`Podfile`中的限制相匹配】。

如果执行`pod update`时未指定pod name，CocoaPods将会`Podfile`中列出的每个pod更新到最新版本。

## 使用目的
使用`pod update PODNAME`，只会更新指定的pod【检查是否存在新版本并相应的更新】。与`pod install`不同，`pod update` 不会尝试更新已经安装的的pods 版本。

当在`Podfile`中增加一个pod时，应该执行`pod install`而非`pod update` - 安装此pod，而且避免在此过程中更新已存在的pod。

只有想更新指定pod版本（或者所有pods）时才需要使用`pod update`。

## 提交你的 Podfile.lock
提醒一下，即使[`Pods`文件夹不允许提交到共享仓库](https://guides.cocoapods.org/using/using-cocoapods.html#should-i-check-the-pods-directory-into-source-control)，**也应该一直提交&推送`Podfile.lock`文件。**

*否则，它会打破上述`pod install`能够锁定已安装pods版本解释的所有逻辑。*

## 场景举例
这有个场景来说明一个项目生命周期中各种可能的遇到的使用情况。

### Stage1: user1 创建项目
*user1*创建一个项目，并且准备使用`A`,`B`,`C`（三个pods）。使用这些pods创建`Podfile`，并且执行`pod install`。此种情况（该项目）将会安装`A`,`B`,`C`（三个pods），版本号都定为`1.0.0`。

`Podfile.lock` 将会保持追踪并且记录`A`,`B`,`C`所有安装的版本都是`1.0.0`
>*由于这是首次执行`pod install`，而且`Pods.xcodeproj`项目尚未存在，所以该命令会创建`Pods.xcodeproj`以及`.xcworkspace`，这是该命令附带的效果，而非主要结果。*

### Stage 2: User1 增加一个新的pod
过会，*user1*准备在项目中的`Podfile` 中增加`D`pod 。

**因此，需要执行`pod install`，**即使首次执行`pod install` 后，`B`pod的维护又发布了`1.1.0`版本，项目仍然会保持使用`1.0.0`版本 — 因为*user1*仅仅是想增加`D`pod，而不希望冒险去更新`B`pod。 

>*这就是某些人使用错误的地方，因为他们在这使用了`pod update` — 可能认为“我想用新的pods更新 *project* ”？— 所以使用了`pod install` 去安装项目中的新pods*

### Stage 3: User2 加入项目
从未参加该项目的*user2*加入了该小组，克隆好仓库并执行了`pod install`。

`Podfile.lock`中的内容可以保证*user2*得到与*user1*正确且相同的pods。

即使是`C`pod维护的最新版本是`1.2.0`，*user2*仍将使用`C`pod的`1.0.0`版本。因为那是在`Podfile.lock`被注册的版本。即，`C`pod被`Podfile.lock` 锁定为`1.0.0`
### Stage 4: 检查pod的新版本
稍过些时候，*user1*准备查看pods是否有更新可用。执行`pod outdated`后，提示pod `B` 有`1.1.0`新版本，pod `C`有`1.2.0`新版本。

*user1*决定更新pod `B`，而pod `C`保持不变；于是，**执行`pod update B`**，将pod `B` 从 `1.0.0` 更新至 `1.1.0`（也会同时相应的更新`Podfile.lock`），**但是**pod `C`仍保持为`1.0.0` 版本（*不会*更新至`1.2.0`）

## 在Podfile中指定明确的版本仍不够
待续...

***
写于18年08月22号
