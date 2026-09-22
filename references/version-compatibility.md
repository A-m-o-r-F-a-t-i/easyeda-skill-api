# 版本兼容

| 组合 | 行为 |
| --- | --- |
| 旧Gateway + Bridge2 | v1 execute继续服务旧工具 |
| Gateway1.1 + 旧Bridge | 仅已验证v1路径；不假设typed RPC可用 |
| Gateway1.1 + Bridge2 + MCP2 | 类型化读写和准备状态 |
| 新MCP连接仅v1的Gateway | 已验证只读原生兼容路径可用，受保护写入返回CLIENT_UNSUPPORTED |
| diagnostics profile | 3项维护工具，生产默认不注册 |
| legacy profile | 29项兼容入口，仅独立回归或明确回滚 |

生产服务名称保持easyeda-pcb，只切换其实际发布路径。旧源码、标签和配置保留但不同时启用。
类型升级与客户端升级分开，客户端方法存在但结果失败时保留实际错误，不按版本号声称支持。
