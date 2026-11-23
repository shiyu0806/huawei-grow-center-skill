# huawei-grow-center-skill

完成华为云开发者成长中心任务的 skill：按每日、每周、每月周期做任务、领积分。

2026-11-23 实测：9 项任务全部完成，当日 +540 分，总分 850 升到 1390。

## 内容

- `SKILL.md`：完整做法，包括登录态保存与复用、签到和任务清单接口、9 项任务各自的做法、鸿蒙实践、展览馆发布作品、Star Grape、核对和汇报方法。
- `skill.yaml`：skill 描述信息。

## 安装

把这个仓库放到你的 skill 目录，例如 Muse 的 `~/workspace/custom-skills/huawei-grow-center/`，或者按你所用 agent 的 skill 存放规则放置：

```bash
git clone https://github.com/shiyu0806/huawei-grow-center-skill.git
cp -r huawei-grow-center-skill ~/workspace/custom-skills/huawei-grow-center/
```

## 使用

1. 在华为云开发者成长中心登录自己的账号，绑定 GitCode，GitHub 相关任务需要再绑定 GitHub。
2. 让 agent 读 `SKILL.md`，先看成长中心的任务列表，再按周期逐项执行。
3. 每次执行完核对积分明细账本（页面顶部总分会滞后，以账本为准）。
4. 任务列表会变，有新任务就尝试执行，执行完把新做法更新回 skill。

## 定时执行建议

- 每日任务：每天做一次。
- 每周任务：每周一做一次。
- 每月任务：每月 1 号做一次。
- 全部按北京时间。每次执行前先读任务列表。

## 注意

- 全程不要下单、不要付费、不要创建付费云资源。
- 登录过期、需要本人授权或验证的步骤，让用户自己完成，不要猜账号密码。
- 本项目与华为云官方无关，接口和页面随时可能变化，以实际页面为准。
- 参考文章：https://bbs.huaweicloud.com/blogs/503275

## License

MIT
