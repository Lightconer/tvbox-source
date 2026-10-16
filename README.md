# 单仓源（tvbox-source）

一个可**自动更新、稳定自维护**的单仓订阅源，供 **TVBox / 影视仓** 等播放器作为"源"使用。

本仓库通过 GitHub Actions 定时监听上游，自动拉取、合并、去重并回写 `tvbox.json`，全程无需手动干预。

---

## 目录结构

```
tvbox-source/
├── tvbox.json                  # 源配置文件（播放器消费的成品）
├── upstream.txt                # 上游源地址列表（你要维护的唯一入口）
├── scripts/
│   └── update.py               # 自动更新脚本（拉取→合并→去重→写回）
└── .github/
    └── workflows/
        └── auto-update.yml     # GitHub Actions 自动更新工作流
```

## 怎么用

### 1. 配置上游源

编辑 `upstream.txt`，每行填入一个你要聚合的成熟源地址（TVBox 兼容配置 JSON 的直链）：

```
# 示例
https://raw.githubusercontent.com/Example/box/main/tvbox.json
https://example.com/api.php/provide/config
```

保存并推送到 GitHub 后，工作流会自动执行一次合并。

### 2. 获取订阅地址

仓库里的 `tvbox.json` 就是你的成品源，直接给 TVBox / 影视仓 填写订阅地址即可：

```
https://raw.githubusercontent.com/Lightconer/tvbox-source/main/tvbox.json
```

## 自动更新机制

| 触发方式 | 说明 |
| --- | --- |
| 定时任务 | 每 6 小时自动轮询一次上游（`0 */6 * * *`，UTC） |
| 手动触发 | 在仓库 Actions 页面点击 `Run workflow` |
| 推送触发 | 修改 `upstream.txt` 或 `scripts/update.py` 时自动执行 |

工作流每次都会：
1. 逐个拉取 `upstream.txt` 里的上游源；
2. 合并 `sites / lives / rules / flags / parses` 各段；
3. 按规范化 key 去重，保留先出现的条目；
4. 写入 `tvbox.json` 并附上 `last_updated` 与统计信息；
5. 若内容无变化则静默跳过提交（不产生无意义的历史）。

## 高级用法（可选）

在 GitHub Actions 的 `Run workflow` 中，或本地运行脚本时，可启用健康检查：

```bash
# 合并后对每个站点做 HEAD/GET 探测，剔除确认失活的站点（较慢）
python scripts/update.py --health-check

# 只查看合并结果，不写文件
python scripts/update.py --dry-run

# 自定义请求超时
python scripts/update.py --timeout 20
```

> 提示：健康检查会显著拖慢执行时间，默认工作流不开启；建议在源头上游不稳定时再手动启用。

## 本地开发

```bash
python scripts/update.py --dry-run
```

## 免责声明

本仓库仅提供源的聚合与自动更新机制，不生产、不存储任何媒体内容。上游地址由使用者自行配置，请遵守所在地法律法规及上游源的服务条款。
