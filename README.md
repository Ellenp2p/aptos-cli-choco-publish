# aptos-cli-choco-publish

本仓库用于自动将 [Aptos CLI](https://github.com/aptos-labs/aptos-core) 打包并发布到 [Chocolatey](https://community.chocolatey.org/packages/aptos)。

---

## 两个 CI 工作流，到底用哪个？

仓库里有两个 GitHub Actions 工作流，它们是**配合使用**的：一个负责监测新版本，另一个负责实际发布。

### 1. `check-update.yaml` —— 自动版本检测（自己跑，不用管）

| 属性 | 说明 |
|------|------|
| **触发方式** | 每天北京时间 10:00（UTC 02:00）自动运行；也可在 **Actions → Check Aptos CLI update → Run workflow** 手动触发 |
| **做什么** | 去 `aptos-labs/aptos-core` 查最新稳定版 Aptos CLI，再检查 Chocolatey 上有没有这个版本。如果**没有**，就自动触发 `cli-publish.yaml` 去发布。 |
| **需要手动运行吗？** | **不需要**——它每天自动跑。只有当你不想等到明天、想立刻检查一次时，才需要手动触发。 |

### 2. `cli-publish.yaml` —— Chocolatey 发布（真正干活的）

| 属性 | 说明 |
|------|------|
| **触发方式** | 由 `check-update.yaml` 检测到新版本后**自动调用**；也可在 **Actions → Checkout Aptos CLI Version → Run workflow** 手动触发（需要自己填写版本号，例如 `v5.1.0`） |
| **做什么** | 拉取指定版本的 `aptos-labs/aptos-core`，下载 Windows 二进制文件，计算校验和，打包成 `.nupkg`，然后推送到 Chocolatey。 |
| **需要手动运行吗？** | **一般不需要**。只有在想立刻发布某个特定版本、或者自动流程漏掉了某个版本时，才需要手动触发。 |

---

## 整体流程图

```
每天定时（北京时间 10:00）
        │
        ▼
check-update.yaml
  ├── Chocolatey 上已有最新版？
  │     是 → 输出"无需更新"，流程结束。
  │
  └──   否 → 自动触发 cli-publish.yaml，传入新版本号
                        │
                        ▼
              cli-publish.yaml
              打包并推送 Chocolatey 包
```

**正常情况下**：两个工作流都不需要手动操作，每天自动完成检测和发布。

**手动发布**：如果需要立刻发布某个指定版本，直接在 Actions 页面运行 **`cli-publish.yaml`** 并填入版本号即可。

---

## 所需密钥

| 密钥名称 | 说明 |
|----------|------|
| `CHOCO_API_KEY` | 你的 Chocolatey API Key，供 `cli-publish.yaml` 推送包时使用。在 **Settings → Secrets and variables → Actions** 中设置。 |