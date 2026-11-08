# Relay-Station · 通用指令中转站

本仓库是**通用指令中转站**：一个 Pages 站点下托管多个控制台页面，并用 Issues 承载各类指令队列。
当前服务于 RemoteDL 远程下载，但定位**不限于下载指令**，后续其它中转需求也复用本仓库。

## 页面目录

| 路径 | 网址 | 说明 |
|---|---|---|
| `index.html` | <https://f1uke76.github.io/Relay-Station/> | 导航页，列出所有控制台 |
| `RemoteDL-Queue/index.html` | <https://f1uke76.github.io/Relay-Station/RemoteDL-Queue/> | 远程下载控制台（原名，专用于远程下载） |

## 添加新控制台

在 `main` 分支新建一个子目录并放入 `index.html`，网址即为
`https://f1uke76.github.io/Relay-Station/<子目录名>/`；然后在根导航页加一条入口即可。
每个页面可以有自己完整的 HTML / CSS / JS。

> **共用队列要注意**：所有页面共享同一个仓库的 Issues。远程下载 App 会读取全部 open Issues，
> 只执行首行形如 `ADD https://...` 的指令，其它控制台请避开这个格式，或改用独立前缀 / 标签。

- 队列 API：<https://api.github.com/repos/f1uke76/Relay-Station/issues?state=open&per_page=100>（公开匿名可读）
- 进度回传：写进对应 Issue body 的隐藏注释 `<!--RDL:{...}-->`

配套私有库 `f1uke76/RemoteDL` 存放 Android 源码、APK 产物与 `pat.md` 令牌登记。
网页与 App 都不内置令牌；首次使用请打开私有库 `pat.md` 手动复制填写。
