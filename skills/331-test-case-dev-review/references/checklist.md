# 测试脚本开发评审检查清单

## 1. API 测试脚本评审

| 检查项 | 要求 |
|--------|------|
| 接口覆盖 | 设计文档中的API用例均有对应 `.spec.ts` 脚本 |
| 断言质量 | 状态码 + 业务code + 关键数据字段断言 |
| 数据驱动 | 参数化数据使用 fixture 或 JSON 文件 |
| 权限测试 | 每个角色均有 Token 认证测试 |
| 异常覆盖 | 400/401/403/404/500 场景有脚本 |
| 脚本命名 | `{module}.spec.ts` 格式统一 |

## 2. E2E 测试脚本评审

| 检查项 | 要求 |
|--------|------|
| 页面覆盖 | 设计文档中的E2E用例均有对应脚本 |
| Page Object | 共享组件封装在 `shared/page-objects/` |
| 元素定位 | 基于字段一致性原则，优先使用 getByRole/getByPlaceholder/locator([name])（无需 data-testid） |
| 跨终端 | 多终端用例使用 `MultiTerminalTest` |
| 独立性 | `beforeEach` 准备数据，无测试间依赖 |
| 目录结构 | 单终端在 `{项目名}/`，跨终端在 `cross-case/` |

## 3. JMeter 脚本评审

| 检查项 | 要求 |
|--------|------|
| 场景覆盖 | 基准/负载/压力/稳定性均有 `.jmx` 文件 |
| 参数化 | 服务器地址、端口使用 `${__P()}` |
| CSV数据 | 在 `load/data/` 目录，格式正确（UTF-8, 首行标题） |
| 断言 | 状态码断言 + 业务code断言 + 响应时间断言 |
| 线程组 | 线程数、ramp-up、持续时间与设计文档一致 |
| JSON提取 | Token 提取使用 JSON PostProcessor |
| 监听器 | Aggregate Report + Backend Listener |

## 4. 安全测试脚本评审

| 检查项 | 要求 |
|--------|------|
| ZAP扫描 | `zap-scan.sh` 目标URL、排除路径、输出格式正确 |
| Trivy扫描 | `trivy-scan.sh` 覆盖后端/前端/镜像 |
| Playwright安全 | SQL注入、XSS、越权、认证绕过脚本完整 |
| OWASP覆盖 | Top 10 类别均有对应测试 |
| 报告输出 | JSON + HTML 双格式 |

## 5. bin/ 执行脚本评审

| 检查项 | 要求 |
|--------|------|
| 脚本完整 | setup/run-api/run-e2e/run-load/run-security/run-all 齐全 |
| 环境变量 | API_BASE_URL、WEB_BASE_URL 等引用正确 |
| 报告路径 | 输出到 `test/reports/{type}/` 带 YYMMDDHHMM 后缀 |
| 可执行权限 | `chmod +x` 已设置 |
| 错误处理 | 脚本中有错误退出和提示 |

## 6. 通用质量检查

| 检查项 | 要求 |
|--------|------|
| Playwright配置 | 双项目（api + e2e）配置正确 |
| 共享工具 | auth.ts / test-data.ts / multi-context.ts 抽取复用 |
| 无硬编码 | 无硬编码URL、密码、Token |
| 代码风格 | TypeScript 严格模式，统一格式 |
