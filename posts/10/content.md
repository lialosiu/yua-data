title: 修复 Windows 10 中损坏的 Appx 应用
date: 2015-08-28 16:14:43
---

前段时间手贱把 Surface Pro 3 的 系统升级到了 Windows Indsider Fast Ring 通道的 10525 版本，然后发现 [Chrome](https://www.google.com/intl/zh-CN/chrome/browser/desktop/index.html) 和 [Virtual Box](virtualbox.org) 都出现了兼容性问题，估计是新的内存管理机制的锅。

因为工作需要用到 Vagrant，[Virtual Box](http://virtualbox.org) 不能用简直不能忍，于是我滚回到了10240

然而没想到回滚完成后，好几个 Appx 应用都出问题了，应用名显示为 `@{microsoft.windowscommunicationsapps_17.6120.42011.0_x64__8wekyb3d8bbwe}` 类似这样的形式。

{% asset_img 000.jpg [000.jpg] %}

出问题的应用分别为 Edge、日历与邮件、应用商店、Cortana，其中 Edge 和 Cortana 只是名字没了Icon没了，商店则是变成了英文菜单，日历与邮件则是直接打开不能、更新不能、卸载不能、完全拿他没办法。

强行忍了一个多星期，实在受不了，于是昨天抽时间出来终于把这问题解决了，写下来记录下步骤 ~~顺便给blog填点东西~~

-----

首先，祭出最基本的 `wsreset` 大法，管理员运行，然而并没有什么卵用

试着跑了下系统自带的`疑难解答`，然而它提示说……

{% asset_img 001.jpg [001.jpg] %}

|дﾟ)

好咯，还是要自己动手

既然系统搞不定，那就自己来研究吧

管理员身份打开 `Powershell`，把所有包重新注册一遍

```ps
Get-AppXPackage | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```

坐等了大概一分钟，跑完了

跑完瞬间发现，Edge恢复正常了，那写坏掉的应用名也恢复正常了！

然而还是有两个坏掉的图标……

{% asset_img 002.jpg [002.jpg] %}

日历与邮件还是不行\_(:3」∠)\_

瞄了一眼 Powershell，发现有报错

{% asset_img 003.jpg [003.jpg] %}

咦，`C:\` 在C盘根目录找是什么鬼

输出一下包信息看一下先：

```ps
Get-AppxPackage microsoft.windowscommunicationsapps*
```

{% asset_img 004.jpg [004.jpg] %}

妈了个鸡，为何 InstallLocation 不见了……

```ps
cd "C:\Program Files\WindowsApps\"
ls
```

{% asset_img 005.jpg [005.jpg] %}

妈了个蛋，还真没了这个文件夹……

存在的文件是 `42011` 然而包信息里面的是 `42001`

好吧，估计这个就是问题所在了……

试试移除掉这个包咯

```ps
Get-AppxPackage *microsoft.windowscommunicationsapps* | Remove-AppxPackage
```

然而并没有用，还是读的根目录

想了想，既然只会读根目录，那我就把 42011 的 AppXManifest.xml 扔在 C: 根目录总能读到了吧？

于是把整个 42011 的所有文件复制一份扔在根目录

然后试着：

```ps
Add-AppxPackage -register "C:\AppxManifest.xml" –DisableDevelopmentMode
```

有反应！但是依然是报错信息，说在 `C:\ProgramData\Microsoft\Windows\AppRepository` 里面找不到对应的xml

那我就复制一份进去咯

```ps
xcopy 'C:\AppxManifest.xml' microsoft.windowscommunicationsapps_17.6106.42001.0_x64__8wekyb3d8bbwe.xml
```

再试试看！

```ps
Add-AppxPackage -register "C:\AppxManifest.xml" –DisableDevelopmentMode
```

nice！成功运行！

打开开始菜单看看，邮件恢复了！

为了防止出现各种奇怪的问题，先把包砍了再从商店装一遍吧

然而并不能在GUI下直接卸载，那就老方法

```ps
Get-AppxPackage *microsoft.windowscommunicationsapps* | Remove-AppxPackage
```

ok，卸掉

然后商店重新安装

{% asset_img 006.jpg [006.jpg] %}

卧槽 教练这波不亏 (´・ω・`)
