# 华为云开发者成长中心任务

## 任务是什么

- 打开华为云开发者成长中心（https://grow.developer.huaweicloud.com/），按任务周期完成当日、当周、当月的任务并领取积分。

- 每次执行前先读一遍成长中心的任务列表，任务列表会变，有新任务就尝试执行，执行完把新发现写回这个 skill。

- 开始前填好自己的账号：华为云开发者账号、已绑定的 GitCode 账号、已绑定的 GitHub 账号（做 GitHub 相关任务时才需要）。

## 登录态怎么保存和复用

- 华为云的会话 cookie 全部是 session 类型，没有 expires，关掉浏览器理论上就失效，普通 user-data-dir 持久化对它不生效。

- 用 Playwright 的 `storageState` 存取登录态，保存时按 domain、path、name 三个字段做并集合并，不能整体替换。

- 华为云是一域一套 session，developer、bbs、auth、gallery 各有自己的 cookie，直接整体覆盖会把别的域的登录态冲掉。

- 不能只在 Node 里直连接口，cookie 种在 developer 域、由浏览器带着去访问 devdata 域，Node 直连时没有这套会话，所有接口调用都在页面上下文里用 `fetch`，并带 `credentials: 'include'`。

- 所有 POST 请求都带一个 `csrf` 请求头，这个值不在 cookie 里也不在 HTML 里，是页面 JS 跑起来之后才算出来的，监听页面的真实请求，把请求头里的 `csrf`、`hd-csrf`、`x-csrf-token` 抄下来。

- 页面加载完不一定马上发带 csrf 的请求，拿不到就 reload 一次等 6 秒，还没有就点一下页面上的积分明细，强制它发一次请求。

- 如果打开成长中心被跳到登录页，说明登录过期，停下来汇报，让用户自己完成华为云登录和验证，不要猜账号密码，也不要用别的账号登录。

## 接口速查

- 签到接口在 `https://devdata.huaweicloud.com/rest/developer/fwdu/rest/developer/user/hdcommunitysoservice/v1/growth`，路径里 `rest/developer` 出现两次，照抄，不要改。

- 先 GET `/growth/sign` 查 `sign_flag`，0 是没签，1 是签过了，再决定要不要 POST。

- POST 签到要带 body `{"account_name": "账号名"}`，账号名从 `GET .../hdcommunityservice/v1/member/get-personal-info` 的 `memName` 拿，注意这里服务名是 `hdcommunityservice`，少了 `so`。

- 重复签到返回 HTTP 200 加业务错误码 `HD.67510032` 或 `HD.67520011`，意思是今天已经签过，不要用 HTTP 状态判断成功，POST 之后再 GET 一次 `sign_flag` 确认真的变成 1。

- 任务清单接口是 `POST .../growth/mission/query-mission-list-by-condition`，body `{"page_no": 1, "page_size": 8, "category_key": "DAILY_MISSIONS"}`，分页循环到返回空列表为止。

- 任务列表里每条看 `mission_name`、`bonus`、`is_finish`，再看 `cycle_type`：`DAILY` 每日，`WEEKLY` 每周，`MONTHLY` 每月，`LIMIT` 一辈子一次，页面上不显示周期，只看页面会把每月任务当成每日任务白做。

- 浏览器里做任务时优先用用户本机 Chrome（登录态保存在那里），云浏览器可能打不开 github.com，也可能报 `github.client_id 未配置`。

## 任务清单和做法

- 以下清单是 2026-11-23 实测的版本，每次执行前以页面上的最新列表为准。

- 每日任务共 4 项：每日签到 10 分，打开成长中心页面就自动完成，没有签到按钮，页面会弹提示；体验 AI DevSpace 30 分，进 AI DevSpace 体验版，容器正常运行并新建 AI Shell 会话实际用一下终端；每日浏览一个鸿蒙案例 10 分，在案例详情页停留超过 30 秒；Star 一个鸿蒙案例代码仓 30 分，在 GitCode 上给案例代码仓点 Star。

- 每周任务共 3 项：Star 一个 Agent 开发实践中心代码仓 50 分；给 Agent 代码仓提 Issue 60 分，在 Agent 实践中心的代码仓提交 Issue；每周实践一次鸿蒙案例 100 分。

- 每月任务共 1 项：在开发者作品展览馆中发布作品 200 分。

- 一次性任务共 1 项：Star 开源项目 Grape 代码仓 50 分，仓库在 GitHub（github.com/grape-dev/grape），必须用 GitHub 身份 Star，GitCode 绑定不够。

## 每周实践鸿蒙案例的做法

- 进作业中心体验版，在鸿蒙开发实践中心 WebUI 新建项目，填入案例公仓地址，系统自动拉取代码，再在 CodeArts CLI 里用 hvigorw 编译打包，生成未签名的 HAP 包。

- 2026-11-23 实测用的是朗读助手案例，代码仓 `https://gitcode.com/harmony-practice-center/HarmonyOS-APP-voice-assistant.git`，编译 BUILD SUCCESSFUL，HAP 约 801KB，项目登记后状态为 active。

- 这项积分按任务周期统一发放，不是做完马上到账，做完先记下项目登记状态，稍后去积分明细账本复查。

- 不要安装到云手机，不要创建任何付费云资源，HAP 未签名是正常的，签名需要已实名认证的华为开发者联盟账号。

## 展览馆发布作品的做法

- 展览馆入口在 https://gallery.developer.huaweicloud.com/gallery/submit-guide，成长中心的去完成按钮有时点不跳转，换展览馆直链进。

- 按 `huawei-cloud-publish-work-to-gallery` Skill 的 Step 1 到 Step 8 走：前置检查、跳过运行项目、命名、生成 16:9 封面（1280x720）、选择可投稿活动、生成详情包、门禁复核、提交发布并申领积分。

- 2026-11-23 实测选的是通用型可投稿活动“ICT大赛作品展示活动”，提交返回 201 成功，reward=success，value=200。

- 发布流程里检查 Git 仓库一步需要 GitCode 凭证，凭证缺失时会弹出 GitCode OAuth 授权链接和二维码，这一步只能用户本人在浏览器里完成授权，不能代做，授权完再继续。

## Star Grape 的做法

- 先确认 GitHub 已绑定，绑定流程里 AIAgentPracticeCenter 只请求只读个人资料权限，授权后重定向回 hands-on-agent.developer.huaweicloud.com。

- 再去 github.com/grape-dev/grape 点 Star，确认按钮变成已加星标，再去积分明细账本核对 50 分到账。

- 如果 GitHub 绑定页报 `github.client_id 未配置` 或 github.com 打不开，先不要硬做，汇报给用户，等站点恢复或换浏览器再试。

## 核对和汇报

- 不要相信返回值和页面顶部的总分，页面顶部和任务清单显示会滞后，以积分明细账本为准。

- 每做完一项去积分明细账本核对一次，记下每项的到账时间和账本余额，全部做完再汇总汇报。

- 2026-11-23 实测结果：9 项全部完成，今日 +540 分，总分从 850 升到 1390，积分当年有效，次年 1 月 1 日 0 点清零。

- 全程不要下单、不要付费、不要改动与任务无关的设置，需要用户本人授权、验证或确认的步骤停下来汇报，等用户做完再继续。

## 定时执行

- 每日任务每天做一次，每周任务每周一做一次，每月任务每月 1 号做一次，全部按北京时间。

- 每次执行前先读任务列表，有新任务就尝试执行，执行完把新任务的做法和新发现更新到这个 skill 里。

## Available connectors

- Use `muse.connector_search` to find other connectors and `muse.skill_search` to find related skills.

- If nothing here fits, look on the web for a maintained API or MCP server and set it up with [Custom Connectors and MCPs](/opt/muse/skills/custom-connector-setup/SKILL.md), or build a custom connector.

## Websites

- [华为云开发者成长中心](https://grow.developer.huaweicloud.com/) for 任务列表和积分明细，浏览器任务里先读任务列表再逐项做。

- [开发者作品展览馆](https://gallery.developer.huaweicloud.com/gallery/submit-guide) for 发布作品的入口和提交指引。

- [华为云开发者社区文章](https://bbs.huaweicloud.com/blogs/503275) for 签到自动化和登录态复用的抓包思路，接口路径和字段照抄。
