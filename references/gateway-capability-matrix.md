# 能力矩阵

每次窗口重新注册或 generation 变化后重新读取能力。方法存在性仅记为 method-presence，真实调用结果单独保留。

| 能力 | 当前实现 | 使用条件 |
| --- | --- | --- |
| Protocol v1 | 保留 | 旧 execute 兼容，不附加新写入保证 |
| Protocol v2 | Bridge2/Gateway1.1实现 | 两端握手均声明2 |
| 精确目标、generation、epoch | 实现 | 全目标核对，epoch为窗口保守失效 |
| PCB 图元事件 | partial | BETA事件可能遗漏，不能代替源码和旧对象断言 |
| PCB Info/DSN | 有界UTF-8读取 | 当前客户端方法实际返回File且数据校验通过 |
| 源码哈希/检查点 | 哈希元数据 | 不含文档内容，不能恢复工程 |
| 类型化几何/文字写入 | 固定执行器 | prepare guard、旧对象、独立读回 |
| 文件分块 | 未启用 | binaryChunkTransfer=false |
| 基础连接图 | PCB MCP实现 | 复杂铜与孔形未覆盖时PARTIAL |
| 自动布局/自动布线 | 不暴露 | 模型只执行显式计划 |

当前验证客户端为3.2.186；类型包固定0.4.25。更高客户端版本、3D/ODB++与复杂连接模型另行验证。
