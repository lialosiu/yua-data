title: 弃用bower，以后的项目准备改用jspm！
date: 2016/06/12 09:29
---

嘛，刚刚在项目中跑了下`bower update`，然后就发现`jointjs`的`bower.json`文件里面，依赖居然空了……

再看看`package.json`，依赖还在

也就是说`jointjs`的维护者尼玛的把`bower.json`写错了……\_(:3」∠)\_

简直丧心病狂

于是感觉顺便也是时候放弃bower了，毕竟这货最近不知为何拉包都是`git checkout`，巨慢...

于是瞄了下jspm，目测用着还行，于是以后的新项目准备投奔jspm去咯~
