# Milolo 二开说明（milolo/main 分支）

> 本 fork = Milolo 云端 AI 数据面的底座（issue #17，lengjingxu/miloco-pi）。
> 上游：agentgateway/agentgateway（Linux Foundation，Apache-2.0）。**保持最小 diff**，方便跟上游合并。

## 我们用到什么

| 能力 | 位置 | 状态 |
|---|---|---|
| LLM Gateway（OpenAI 兼容路由 + baseUrl 自定义上游） | `crates/llm/src/custom.rs`（formats/path 可配）、`openai.rs` | 上游原生，零改动 |
| JWT 校验（issuer/audience/jwks file，mode: strict） | `crates/agentgateway/src/types/local.rs` jwtAuth | 上游原生，零改动 |
| 计量缝：`report_usage()` | `crates/llm/src/lib.rs` | Phase 2：在此挂 per-family 持久化 |
| MCP Gateway | `crates/agentgateway` mcp | 预留（roadmap C8 设备层 MCP 化） |
| A2A / Guardrails / K8s controller（Go） | controller/ 等 | 未用，暂不裁剪（保持合并面） |

## 配置契约（与 miloco-pi cloud/worker 对齐）

- JWT：issuer `milolo-cloud`、audience `milolo-ai`、RS256、jwks 文件由 miloco-pi `cloud/gateway/scripts/generate-keys.mjs` 生成（kid=milolo-2026-09）
- 上游：`llm.models[0].params.baseUrl = $AI_UPSTREAM_URL`（OpenAI 兼容）、`apiKey = $AI_UPSTREAM_KEY`
- 端口：LLM 4000

## 构建与验证

- 工具链：`rust-toolchain.toml` 钉 1.98（**brew 的 1.92 编不过**，aws-smithy 系依赖要求 ≥1.94.1；用 rustup）
- `cargo check` → `cargo build --release` → 二进制 `target/release/agentgateway`
- 集成验证：跑本仓二进制 + miloco-pi `cloud/gateway/gateway.yaml`，无 JWT 401 / 有 JWT 透传上游
- 镜像：MVP 用本地构建产物的瘦 Dockerfile；产品化切上游多阶段 Dockerfile（在部署主机构建）
