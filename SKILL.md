---
name: easyeda-api
description: 嘉立创EDA通用公开API、Bridge运行、Gateway Protocol v1/v2协商、精确窗口/工程/文档身份、异步返回值和有界文件传输。用于启动、重连、版本兼容、扩展开发与API缺口诊断；PCB设计与类型化执行归独立PCB Skill/MCP，原理图设计归原理图增强Skill。
version: 2.5.0
---

# 嘉立创 EDA 通用 API

负责把已经确定的编辑意图准确映射到当前客户端的公开 API，并维护 Bridge、协议与能力兼容。
不承担 PCB 布局、自动寻路、连接算法或电路设计策略；不把低频 API 实现细节提前塞入设计上下文。

## 1. 连接与身份

读取 Bridge `/health`，确认 service=easyeda-bridge，再从 `/eda-windows` 取得明确窗口。
生产 PCB 任务通常可用其目标与状态工具完成发现，不必重新启动 Bridge 或重连浏览器。
目标包含 windowId、projectUuid、documentUuid 和真实 tabId，不依赖共享活动窗口或最后焦点。
同名板、测试工程和其他线程的正式工程必须分开。仅为发现目标读取其他窗口身份，不越界读取设计内容。

Bridge 只绑定127.0.0.1。默认端口49620，明确配置范围49620–49629；端口占用时识别现有进程，不盲目启动第二个服务。
跨机器访问、安全授权与账户设置由相应宿主配置负责，本 Skill 不公开本地控制端口或保存令牌。

## 2. 运行入口

本包携带独立 Bridge Server 2.0 的发布副本，依赖 Node.js >=22 和 package-lock 中固定的 ws。
在 Skill 根目录安装依赖后运行：

```powershell
npm ci --ignore-scripts
node scripts/bridge-server.mjs
```

可选环境变量 `EASYEDA_BRIDGE_PORT` 指定49620–49629中的一个端口。
已连接且版本满足要求时继续使用现有 Bridge，不重复启动。进程管理、启动器与回滚配置留在宿主安装记录。
运行源码在独立 bridge-server 包维护，发布时同步构建产物；不直接修改当前已安装 Skill 的脚本。

## 3. 按能力选择通道

常见 PCB 操作必须优先使用 PCB MCP 4.1.0 简化接口。位号、焊盘端点、图层名、批量几何、原生参数和执行结果由 MCP 封装，AI 不重新输出同等 API 脚本。常规 PCB 几何固定为 mil，公共接口不提供单位切换；使用 pcb_edit 和 pcb_read，不调用已删除的兼容工具。
MCP 是 API 的上层操作封装，AI 自主决定设计和分析时机。不要把底层协议的守卫、代次或执行凭证要求重新加回常规 MCP 参数。
只有明确的封装缺口、MCP 实现故障或底层诊断才展开本 Skill。缺失常见操作应补进 MCP，临时 API 调用不成为后续默认路径；无需故意制造失败来证明缺口。
MCP 内部选择已实现的原生调用路径。历史 Protocol v2 受保护操作仍遵守它自己的协议，不能静默伪造成功或绕过权限拒绝；它不限制 v4 显式编辑封装的正常工作流。
浏览器只补充已确认无稳定 API 的应用 UI 或真实渲染，不代替已有类型化的读取、导出与编辑能力。

Protocol v1 保留 execute/result/error 和原有 HTTP 连接入口。
Protocol v2 使用 `/rpc`、能力注册、事件状态及有界文件封装。完整协议见 [协议](references/gateway-protocol-v2.md)。
操作存在性、真实调用成功和结果完整性分开记录，升级类型声明不意味着客户端自动获得新能力。

## 4. 状态与异步执行

PCB MCP 4.1.0 在内部处理对象定位、MIL 几何、必要的身份读取、实际返回、批量分片和执行记录。普通请求不提供 guard、expected 或 executionId，不在每批后重复整板读取。
历史协议的 generation、changeEpoch、源码摘要用于它自己的并发和恢复语义；重启后的相同 epoch 不代表相同会话。事件可能缺少可靠文档身份或发生截断，不能把未知历史当成没有变化。
超时、断开和部分成功利用实际回执与必要局部读取处理，不自动重放写请求。MCP 可用 requestId 抑制同一持久请求的重复派发，通过 pcb_read 的 receipt/receipts 模式查询；refresh 仅读取原生日志。持久回执不是原子事务或完整 PCB 备份。
当前编辑 API 无法跨客户端锁住人工编辑。明确目标、只改指定字段并报告实际结果；设计判断与是否进一步分析交给 AI。
明确权限拒绝不转成其他入口重试。错误码、恢复条件见 [事件与代次](references/event-and-change-epoch.md)。

## 5. 数据与文件

API 的逻辑对象、File、Blob、undefined 和失败布尔值分别处理，缺失返回不能假定为空列表或成功。
网络调用和应用异步操作都等待真实结果，再检查目标与预期后置条件。
PCB Info 和 DSN 使用严格 UTF-8 文件封装，校验字节长度、SHA-256 和允许的最大大小。
File 在读取 ArrayBuffer 前检查 size；BOM 保留，避免文本还原后哈希不一致。
原生备份与制造文件由领域工具完成新路径写入、签名检查和无覆盖完成，不把哈希元数据当成备份。
二进制流式分块尚未启用，能力返回 false；不得宣称 STEP、ODB++ 大文件流已经可用。

PCB Info 的器件数、元件实例数、物理焊盘数和 DSN 逻辑 pin 数有不同口径。
DSN resolution 表示精度，不能直接当作坐标除数。领域解析与坐标映射由 PCB MCP 处理。
数据来源与限制见 [原生数据](references/pcb-native-data-sources.md)。

## 6. 参数和版本规范

使用与实际客户端匹配的公开类型资料，读取准确签名、枚举、坐标单位和返回类型后再调用。
方法存在但行为未验证时进行只读或可恢复最小测试，记录真实错误与后置结果。
ECO、源文档替换和高影响修改需要实际内容备份；恢复时不能用全局撤销覆盖其他编辑。
API Skill2.5.0 与 PCB MCP4.1.0 配套；本轮仅更新路由说明，不升级 Bridge 或 Gateway。既有客户端4.1.60只读记录应与本次安装验证分别报告。历史3.2.186记录仅对对应客户端适用，不能由只读成功宣称所有写操作实测通过。
SDK scaffold 迁移与协议功能分开发布；当前固定 pro-api-types0.4.25，不用升级类型号替代运行测试。
兼容记录见 [能力矩阵](references/gateway-capability-matrix.md)、[版本兼容](references/version-compatibility.md)。

## 7. 按需资料

- 协议与错误： [gateway-protocol-v2.md](references/gateway-protocol-v2.md)。
- 能力与客户端实测： [gateway-capability-matrix.md](references/gateway-capability-matrix.md)。
- 事件代次与并发边界： [event-and-change-epoch.md](references/event-and-change-epoch.md)。
- 文件封装与上限： [binary-transfer.md](references/binary-transfer.md)。
- 升级、激活及回滚： [upgrade-and-rollback.md](references/upgrade-and-rollback.md)。
- 公开类型按对象查阅： [API索引](references/_index.md)，实现与格式资料按任务再展开。

普通 PCB 任务不默认加载整个 API 类型库、旧客户端矩阵或 Bridge 实现源码。
