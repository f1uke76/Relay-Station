# Relay-Station 仓库规范（Agent 必读）

维护者：WorkBuddy Agent（代 f1uke76）
适用对象：任何修改本仓库的 AI Agent 或自动化脚本。
**修改任何文件前，必须完整阅读本文件。** 本文件优先于 README 与页面内注释。

## 一、本中转库的界面逻辑

- 本仓库是公开仓库，GitHub Pages 从 `main` 分支根目录 `/` 发布，站点根为 `https://f1uke76.github.io/Relay-Station/`。
- 根 `index.html` 只做入口导航：视觉上严格保持「每组名称一行 + 网址一行」，格式为
  `名称：` 换行 `完整网址`。**禁止**加表格、卡片、按钮组、图标、私有库介绍或任何其它可见内容；
  给 Agent 的说明只能放在 HTML 注释或本规范文档里。
- 每个控制台是一个独立子目录，目录里放自己的 `index.html`（以及可选的 CSS / JS），
  网址为 `https://f1uke76.github.io/Relay-Station/<子目录名>/`。各页面资源用相对路径，互不依赖。
- 目前有两个控制台：`RemoteDL-Queue/`（远程下载指令）与 `wake-console/`（远程开机策略，
  写 `wake-console/state.txt` 供路由器读取，**不走 Issues 队列**）。
- App（Android，源码在私有库）的「打开中转网页」与导出链接都指向
  `RemoteDL-Queue/` 的完整地址。若改动该路径，必须同步改 App 常量 `Prefs.WEB_URL`、
  网页常量 `WEB_URL`，重新编译并推送 APK，否则按钮会指向 404。

## 二、中转库与私有库的关系

- 公开库 `f1uke76/Relay-Station`（本库）：托管所有控制台网页，并用 Issues 承载指令队列。
  匿名可读；创建 / 修改 Issue 需要 fine-grained PAT 的 `Issues: Read and write`。
- 私有库 `f1uke76/RemoteDL`：存放 Android 源码、APK 产物、控制台源码副本，以及
  `pat.md`（明文登记主令牌）。匿名访问一律 **404**，必须先用 f1uke76 账号登录 GitHub 才能打开；
  因此「查看 pat.md」按钮走 `https://github.com/login?return_to=<pat.md>`：已登录会直接跳回文件，
  未登录先停在登录页，登录后自动回到 `pat.md`（避免未登录直接看到 404）。
- 网页与 App 都**不内置任何令牌**。首次使用：登录 GitHub → 打开私有库 `pat.md` →
  手动复制「当前令牌」→ 粘贴到控制台或 App → 验证 → 保存。
- 保存**不是无条件覆盖**：先读 `pat.md`，若已登记**另一枚仍有效**的令牌（`GET /user` 返回 200），
  则**拒绝写入**并提示改用登记的那枚（弹窗可一键采用）；只有原登记已失效（401/403）或登记为空时才写入。
  用户要主动换新令牌，必须先「删除所有」清空登记再保存。禁止退回「后写覆盖先写」。
- `pat.md` 末尾只有一条 `<!--RDL-PAT:{...}-->` 登记标记，多端保存不会累加；
  同一枚令牌重复保存只写本机、不重复提交 commit。
  某端点「删除」会清空登记；其它已配置的端启动时会识别清空状态并同步删除本机令牌。
  **清空登记不等于撤销 GitHub 上的令牌** —— 真要作废必须去 GitHub 撤销，并在各端点删除。
- 所有控制台**共享同一个 Issues 队列**。远程下载 App 读取全部 open Issues，只执行首行形如
  `ADD https://...` 的指令。其它控制台必须避开该格式，或改用独立前缀 / 标签，否则会被误当成下载任务。

## 三、给 Agent：后续添加新的中转链接

当用户要求「在本中转站新增一个 XX 控制台 / 网站」时，严格按以下顺序执行：

1. 在 `main` 分支新建子目录，目录名用小写英文或连字符，不得与现有目录重名，不得含空格或中文。
2. 把该控制台完整的 `index.html`（及依赖资源）放进该子目录。页面内资源一律用相对路径。
3. 在根 `index.html` 中新增**一对**内容，格式与现有两行完全一致：
   一行 `XX 中转站：`，换行后一行 `<a href="完整网址">完整网址</a>`。
   除新增这一对外，不要改动根页其它内容，不要把它改成表格或富文本。
4. 同步更新本仓库 `README.md` 的链接清单（同样的「名称：换行 网址」格式），保持极简。
5. 若新控制台也用 Issues 传指令，指令首行必须使用与 `ADD https://` 不同的前缀，
   并在本规范「界面逻辑」补一行说明，避免与远程下载 App 串队列。
6. 同步更新私有库 `f1uke76/RemoteDL` 的 `console/` 源码副本与其 README 中的页面清单。
   **仅当该控制台有 App 端时执行**；纯网页控制台（如 `wake-console/`）跳过本条，也不涉及重编译 APK。
7. 部署用 GitHub Contents API（`PUT` 必须带当前文件 `sha`），不要用 `git push`（本机代理不稳）。
   改完用带随机 query 的请求复核 Pages 已刷新，并检查 `pages/builds/latest` 为 `built`。
8. 完成后在回复里给出新增页面的完整网址，并说明是否需要同步改 App。

## 四、PAT 填写指引（网页与 App 必须一致）

- 「一键获取令牌」预填链接只带 `name` / `description` / `target_name` / `expires_in=none` /
  `issues=write` / `contents=write`，**不预填 Repository access**；用户必须在创建页自己确认仓库范围，
  漏勾是 403 的常见原因。
- 两端都要说明逐项填法与效果：Resource owner 选 `f1uke76`；Expiration 选 `No expiration`（选有效期到期后两端全线 401）；
  Repository access 用 `All repositories` 或只勾 `RemoteDL` + `Relay-Station`；
  `Issues` / `Contents` 都要 `Read and write`，`Metadata` 为强制 `Read-only`。
- 效果对应关系：`Issues` 写 → 网页创建 / 取消 / 删除 + App 回写进度；`Contents` 写 → 读写 `pat.md`、
  其它端同步；只勾一个库 → 另一个库的操作 403；已生成的令牌可直接编辑补权限，令牌字符串不变。
- 网页指引在 `RemoteDL-Queue/index.html` 的 `patGuide` 折叠区，App 指引在 `MainActivity.PAT_GUIDE` 弹窗里。
  **改任何一端都要同步另一端**，并同步私有库 `console/` 源码副本、重新编译 APK；
  根导航页与公开库 README 保持极简，不放指引。
- 指引入口规则：网页「填写指引」按钮是**可展开也可收起**的切换（展开后文字变「收起指引」，
  点 `<summary>` 也能折叠且按钮文字同步）；App 的「PAT 填写指引（逐项效果）」**只在未配置时显示**，
  已保存后隐藏。两端都**不再提供「私有代码库」跳转按钮**，只保留 `pat.md` 入口。
- 「删除」必须先弹选择框，分两条路径：**只删本机**不写 `pat.md`（其它端与登记保留，
  本地时间戳写当前时间防止被旧登记反填）；**删除所有**清本机并清空 `pat.md` 登记（其它端下次启动同步删除）。
  禁止再做成「一点就全删」。

## 四之半、控制台版式（务必保持）

- 页面骨架：`.wrap > h1 / .layout > .card-add / .card-list / .card-dev / .card-pat`，四张卡片都在 `.layout` 内；
  页面**没有副标题、没有开场说明**，第一屏直接是操作卡片。
- 手机（< 900px）：`.layout` 为纵向 flex，视觉顺序固定为
  **添加下载链接 → 下载请求 → 设备监控 → 连接 GitHub**（靠 `order` 控制，不放宽）。
- 电脑（≥ 900px）：`.layout` 变网格，`grid-template-areas: "list add" "list pat" "list dev"`，
  即**左列独占下载管理，右列自上而下「添加下载」「连接 GitHub」「设备监控」**；`.wrap` 放宽到 1080px。
- 「添加下载链接」默认折叠：`.card-head`（标题 + 右侧 `展开/收起`）点一下切换 `.fold-body` 的 `hide`，
  右侧文字与 `aria-expanded` 同步；键盘 Enter / 空格也能开合。**禁止默认展开**，也禁止删掉表单只留按钮。
- 新增控制台卡片时必须先决定它在这两套版式里的位置，不能直接插在 `.wrap` 下破坏分列。
- **页面只放操作元素**：按钮、输入框、状态行、必填提示。双库职责、令牌复制流程、安全须知这类说明文字
  一律写在 `AGENTS.md` / 私有库 README 里，不要堆到网页上（已按要求删除「双库职责与令牌复制方式」等段落）。
- 空的状态行（`.msg` / `.tokstat`）必须不占位：样式里用 `:empty { display: none; }`，避免卡片底部留白。

## 四之三分、App 任务管理与检查更新（务必保持）

- App 的「下载任务」列表与控制台同一套数据：读 `state=all&per_page=100`，显示序号（按列表顺序重排）、
  文件名、状态、认领设备与进度；设备心跳 issue（title `DEVICE`）不进入列表。
- 动作语义必须与控制台一致（统一走 `TaskControl`）：
  暂停 = open + `paused` + `RDL-PAUSE`（保留分片）；开始 = open + `queued` 且清空认领；
  取消 = open + `queued` + 进度归零 + 清空认领（解除认领、重新开放，**不关闭 issue**）；
  删除记录 = closed + `deleted`。禁止在 App 里做成「取消即关闭」。
- App **只管理自己的任务**：`dev` 为空（可自由认领）或等于本机 id 时才显示操作按钮；
  已被其它设备认领的条目只展示状态与认领方，不给按钮，避免两台手机互相打断。
- 操作的对象若是本机当前任务，必须先写 `Sync.pauseRequested` / `Sync.cancelRequested`，
  让下载线程立刻停，再 PATCH issue，避免只改状态不停传输。
- 「检查更新」读私有库 `f1uke76/RemoteDL` 的 `releases/latest`，仓库私有，**必须带令牌**；
  版本比较按数字段逐段比（忽略 `v` 前缀），禁止用字符串比较。有新版只弹窗说明并跳浏览器下载，
  **不做应用内静默安装**（避免引入安装权限与 FileProvider 风险）。

## 四之四、远程开机控制台（wake-console，务必保持）

- 定位：用户在网页上设「路由器联网后要不要自动唤醒电脑」，路由器（小米路由器 mini / Padavan）
  联网后读取 `wake-console/state.txt` 决定是否发 Wake-on-LAN 魔术包。
- **不走 Issues 队列**：状态只落在 `state.txt`，因此不需要任何指令前缀，也不会被远程下载 App
  误判成 `ADD https://` 任务。禁止把唤醒指令塞进 Issues。
- 状态文件是纯文本 `key=value`，一行一键、值内**不含空格**、`#` 开头为注释，路由器直接 `.` 或
  `source` 读取。当前键：`enabled`（1/0 自动唤醒）、`mode`（all/win）、`window`（`all` 或
  `HH:MM-HH:MM`，允许跨零点）、`cooldown`（分钟）、`mac`（可空，空则用路由器本机 MAC）、
  `req`（Unix 秒，一次性唤醒请求，路由器比对后执行一次）、`req_at`（写入时间）、`updated`。
  新增键必须保持「无空格 + 可被 sh 直接 source」两条硬约束。
- 令牌权限只需 `Relay-Station` 的 **Contents: Read and write**，不要向用户索要 Issues 权限；
  令牌存浏览器 localStorage（键 `wol.pat`），**禁止**写进 `state.txt`、页面或仓库任何文件。
- MAC 默认留空：目标网卡地址优先由路由器本地配置持有，公开库只在用户主动填写时下发覆盖值。
- GitHub Pages 对静态文件有约 10 分钟缓存：任何读取方（含路由器、含人工核对）必须带
  `?t=<epoch>` 破缓存；控制台写入后要提示「约 1 分钟（重新构建）后生效」。
- 网页只放操作元素（开关、时段、冷却、保存、立即唤醒、令牌），说明文字放本文件；
  版式沿用四之三的骨架：手机顺序 策略 → 立即唤醒 → 路由器 → 连接 GitHub，电脑端左列独占策略卡。
- 禁止在 `wake-console/` 页面里直接发 WOL 包或调用路由器接口：控制台只写文件，执行侧永远在路由器。

## 五、禁止事项

- 禁止在本公开仓库存放任何令牌、`pat.md`、APK、源码或带签名参数的私密链接。
- 禁止把私有库内容整体复制进公开库；私有库切勿转为公开（`pat.md` 明文且留在 git 历史里）。
- 禁止修改 `RemoteDL-Queue/` 内的控制台逻辑，除非用户明确要求（改它通常要同步改 App 并重编译）。
- 禁止删除或改名现有子目录，除非用户明确要求（会让线上地址与 App 按钮变成 404）。
- 禁止在根导航页或 README 里使用表格、卡片或大段介绍；用户明确要求保持极简。
- 令牌区按钮统一等宽网格（3 个三列、4 个两列两行），不要退回 flex/LinearLayout 让单个按钮拉满整行。
- App 的按钮网格必须用 `removeAllViews()` 后按状态重新 `addView`：`GridLayout` 里 `GONE` 的按钮仍占格子，
  只切 visibility 会留出空位（曾出现「第一行第一格空着」）。
- 禁止只改网页或只改 App 的 PAT 填写指引；两端文案与私有库 `console/` 源码副本必须同步更新（见第四节）。
- 禁止把令牌、MAC 之外的可识别信息写进 `wake-console/state.txt`；禁止让路由器侧以任何方式写回本文件
  （路由器只读，写入只来自网页）。

## 六、版本

- 当前定版：**0.9.1**（`versionCode 21`）。除非用户明确要求，**不要提升版本号**。
- 发版时才改 `versionName` / `versionCode` / `Net.UA` 三处，并只保留当前版本 APK 在 `apk/`。
- 仓库保持纯净：只放当前版本需要的文件，历史产物（旧 APK、临时脚本）不入库。
