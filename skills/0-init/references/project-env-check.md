# 项目根目录检测与前置检查

## 问题

AI Coding 工具可能从项目任意子目录启动（如 `frontend/my-shop-admin-web/`），但 skill 中的项目文件路径（`project-info.md`、`database/`、`backend/`、`frontend/`）都是相对于项目根目录的。

## 第一步：定位项目根目录

从当前目录向上逐层查找 `project-info.md`，最多查找 3 层。

| 步骤 | 操作 |
|------|------|
| 1 | 依次检查 `./project-info.md`、`../project-info.md`、`../../project-info.md`、`../../../project-info.md` |
| 2 | 找到后，将该文件所在目录记为 `PROJECT_ROOT`（后续所有项目文件路径基于此） |
| 3 | 4 层均未找到 → 提示用户先在项目根目录执行 0-init |

**0-init 特殊规则**：未找到时使用当前工作目录作为 `PROJECT_ROOT`（新项目场景，`project-info.md` 将创建在此目录）。

## 第二步：检查项目信息完整性

读取 `PROJECT_ROOT/project-info.md`，检查 YAML 头部是否包含以下 5 个参数，值不能为空或占位符：

| 参数 | 格式要求 | 示例 |
|------|---------|------|
| `project-name` | 英文小写+数字+下划线，1-32字符，下划线仅在中间 | `nova_app` |
| `project-label` | 项目中文名称 | `Nova应用` |
| `project-desc` | 项目描述 | `Nova应用是一个基于Java的Web应用` |
| `project-mode` | 项目模式：`uniweb` 或 `saas` | `uniweb` |
| `project-server` | 开发服务器IP | `127.0.0.1` |

| 检查结果 | 操作 |
|---------|------|
| 文件存在且 5 个参数完整 | 继续当前技能 |
| 文件不存在或参数缺失 | 暂停当前技能，引导用户执行 0-init |

## 引导话术

当检查不通过时，向用户说明：

```
当前技能需要项目信息（project-info.md）才能继续执行。

请先使用 /0-init 技能完成项目信息初始化，完成后可继续当前技能。
```


