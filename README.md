# Relay-Station · 通用指令中转站

本仓库是**通用指令中转站**：既托管在线控制台网页，也用 Issues 承载各类指令队列。
当前服务于 RemoteDL 远程下载，但定位**不限于下载指令**，后续其它中转需求也复用本仓库。
因此这里只放网页与队列，不放任何令牌、源码或产物。

- 在线控制台：<https://f1uke76.github.io/Relay-Station/>
- 指令队列：本仓库 Issues（公开、匿名可读；创建与修改需要主令牌）
- 进度回传：App 把进度写进对应 Issue body 的隐藏注释 `<!--RDL:{...}-->`

配套私有库 `f1uke76/RemoteDL` 存放 Android 源码、APK 产物与 `pat.md` 令牌登记。
网页与 App 都不内置令牌；首次使用请打开私有库 `pat.md` 手动复制填写。
