当想撤销中间某次提交时，强烈建议使用`revert`命令，而不是`reset`。

`git reset –hard commit_id` 虽然可以回退远程库，但是其要求`pull`最新代码的每个人的本地分支都要进行版本回退。这样就增加工作量了！

正确的步骤：

```
    git revert commit_id
//如果commit_id是merge节点的话,-m是指定具体哪个提交点
        git revert commit_id -m 1
//接着就是解决冲突
        git add -A
        git commit -m ".."
        git revert commit_id -m 2
//接着就是解决冲突
        git add -A
        git commit -m ".."
        git push
```

其中`git revert commit_id -m` 数字是针对，`merge`提交点的操作。

如果是普通的提交点，不需要这么麻烦。

推荐阅读：

[Android 如何实现气泡选择动画](http://mp.weixin.qq.com/s?__biz=MzIxMTg5NjQyMA==&mid=2247485806&idx=1&sn=bcc4f10a32e730fc350cb6dfee2b1123&chksm=974f1865a038917329406dac91a42b96670a0139fa6912a4fe87554b6c02ae3e207ce6802e96&scene=21#wechat_redirect)

[自定义LayoutManager：实现弧形以及滑动放大效果RecyclerView](http://mp.weixin.qq.com/s?__biz=MzIxMTg5NjQyMA==&mid=2247485799&idx=1&sn=8bed431b825c700cea0ac105dd495543&chksm=974f186ca038917a2d7d11db34f968b39303ebb22aa6d0720d3a85f3bda5979cdf8555ef445d&scene=21#wechat_redirect)

- [**Flutter 插件开发详细讲解**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247484007&idx=1&sn=cabd496e5a512815cecd4083219e2988&chksm=fd1e4c79ca69c56f9220e5d96866f499e1a06937e1097272b27685e06fea47251b05a698b62f&scene=21#wechat_redirect)
- [**Flutter 1.9 升级体验及填坑全攻略**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247484131&idx=1&sn=f875ec0db3c83dd237b04f281640b6f3&chksm=fd1e4cfdca69c5eb8e110c1a1d5dd81cfc3f6a18824a408cb5efe2d36fc17e4789be555a56b1&scene=21#wechat_redirect)
- [**Flutter for Web 详细介绍**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247484021&idx=1&sn=f7e4690a85963d14201d529474937729&chksm=fd1e4c6bca69c57dcc77dd6c2bebf50f1e9c71ca9d84b27eb4f88975cad00821ad13132024af&scene=21#wechat_redirect)
- **Flutter [路由详解](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247483895&idx=1&sn=32bfdfae090efebf4bbb97d2a08e30a8&chksm=fd1e4fe9ca69c6ffe64e944451fc77111436762f33807aa73acb7cf34cada5a2535b075c42ed&scene=21#wechat_redirect)**
- [**Flutter布局篇（1）--水平和垂直布局详解**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247483843&idx=1&sn=50aef503d5438391cbb33ed98b59f0d7&chksm=fd1e4fddca69c6cb1fef324ab046c995402b851a6ee2c609a436474fc7099d4276600bfb675b&scene=21#wechat_redirect)
- [**Flutter 代码模板，解放双手，提高开发效率必备**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247483804&idx=1&sn=3df7287a2a8d176b2d25394a7dfe2448&chksm=fd1e4f82ca69c6948241bc7e51da65a6320dec5dd32e49d2d6b8da9540890bcdadf2daf77796&scene=21#wechat_redirect)
- [**Flutter 遇到的那些坑全面总结**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247483768&idx=2&sn=f221a6b9007245cad902743b38ed5af0&chksm=fd1e4f66ca69c670d7952aa6f9bc68814e5395857b7e506d156ab484f5c98ef9dda40d3761e2&scene=21#wechat_redirect)
- [**Flutter 配置安装全面总结**](http://mp.weixin.qq.com/s?__biz=MzU3NTcwODAxNw==&mid=2247483768&idx=1&sn=08f0896a4550c7868451f70d8fba091d&chksm=fd1e4f66ca69c670a4e8a29795af007b845f8c9f5527b576da06bb526e2948d2541f0e35e840&scene=21#wechat_redirect)



[使用Flutter一年后，这是我得到的经验](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649843940&idx=1&sn=5ab1ee33f3e4bcec553aa89346dad158&chksm=83bf61bfb4c8e8a903b93023444c325ced76a0ecb2f6da29f5dea7a7010ee46affcb9e44cd92&scene=21#wechat_redirect)

[全方位观测：Flutter与React Native的跨平台王位之争](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649845504&idx=1&sn=473ea77d79b709a41f8ca631247d7ad3&chksm=83bf665bb4c8ef4d268f917b2973962b5ce34df184e5a23473f90e3e9b72812cae94d87a5982&scene=21#wechat_redirect)

[不会Dart，还搞什么Flutter?](http://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649845170&idx=2&sn=c441ea2e1be8478a279ddbf04dcee9da&chksm=83bf64e9b4c8edff0c8973045b898d41809463149deb2bbfb52055952556703d59bb1cad05b8&scene=21#wechat_redirect)

2019[年大前端技术趋势分析](https://mp.weixin.qq.com/s?__biz=MzAxMTg2MjA2OA==&mid=2649845610&idx=2&sn=19f719fb131539acec322849fbb7f608&chksm=83bf6631b4c8ef27e54fa6a1959d631f2333daca4a919a6fcac4cc59e0e4d3c613f769d12bb3&mpshare=1&scene=1&srcid=&sharer_sharetime=1567678960907&sharer_shareid=8a77cdde774533d196d0a78a501a16e6&key=5870bcb69e771e0959c17dfeed14eefb340d58d47bbb0465f330aafcc2d70e74be2d25bf65923793a3d8580e1bc7512c93644badacdf44588a8eeba3a53b57ce199a8d6c7a239b2e3d402380e58a7d85&ascene=1&uin=OTQyMTg1MzE1&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=fN%2BK%2BL2mkYp0OtqN6luQlHGYIwAXTWRRYPe3VBgl%2BF7yv0T2%2Fl7iu5ygXkOB%2FXGo)

[从0开始实现一款类似微信、B站的图片浏览组件](http://mp.weixin.qq.com/s?__biz=MzIxMTg5NjQyMA==&mid=2247485775&idx=2&sn=cc0129255336e6e4f52333beacbb0443&chksm=974f1844a03891529bbb2cfe4be2c5b668803eff3c2e996d85a1fbf0ac0baadd4de804275ffc&scene=21#wechat_redirect)

