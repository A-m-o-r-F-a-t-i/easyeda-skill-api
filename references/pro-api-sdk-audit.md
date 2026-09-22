# 官方 pro-api-sdk 审查与采用边界

## 1. 身份与结论

`easyeda/pro-api-sdk` 是嘉立创EDA专业版扩展开发模板和构建工具，不是带 `SKILL.md` 的 AgentDock Skill。它提供官方 `@jlceda/pro-api-types` 类型依赖、扩展入口、打包脚本和简单对话框示例。现有 EasyEDA Bridge 扩展已经运行在同一个 `eda` API 上下文中，因此为了调用公开PCB API无需再安装第二个插件。

本次核对提交：

```text
2fb050c7889ac5ab64388472f9ba29f9d21b0631
2026-09-11T16:26:43+08:00
ci(deps): #100000 update pro-api-types
```

仓库许可证为 Apache-2.0。审查工作区使用锁文件执行 `npm ci --ignore-scripts --no-audit --no-fund`，没有执行 `npm run update`，也没有把仓库当作Skill安装。

## 2. 供应链与脚本边界

- `build/update.ts` 会从 GitHub/Gitee 下载并覆盖 SDK 框架文件。更新前应固定目标提交、审查差异和锁文件，不能在生产扩展源码上无条件执行。
- 依赖树包含 `esbuild` 和 `simple-git-hooks` 等带安装脚本的包。本次审查阶段关闭生命周期脚本，只使用已下载的类型声明。
- `src/index.ts` 是 `eda.sys_Dialog` 的简单示例，不包含PCB高级操作实现。
- 扩展主进程遵守官方运行时限制，UI应走 `eda.sys_*` 或 iframe，不依赖宿主DOM点击。

## 3. 对当前工具链真正有用的部分

| 能力 | 官方公开API | MCP 1.2.0采用方式 |
| --- | --- | --- |
| 覆铜重建 | `PCB_PrimitivePour.rebuildCopperRegions`、Pour实例 `rebuildCopperRegion` | 运行时检测；3.2.186缺静态方法时逐边界回退，读回`Poured` |
| 独立丝印文本 | `PCB_PrimitiveString` create/get/modify/delete | `easyeda-pcb-text-plan/v1`，限顶/底丝印，旧值断言与读回 |
| 元件属性文字 | `PCB_PrimitiveAttribute` get/getAll/modify/delete | 当前只暴露读取与修改；不伪造无效的create |
| 原生图元查询 | `PCB_Document.getPrimitiveAtPoint/getPrimitivesInRegion` | `pcb_pick`，不改变编辑器选择 |
| 约束组 | `PCB_Drc` 的NetClass、DifferentialPair、EqualLength、PadPair CRUD | 单组受保护操作，不暴露全量覆盖接口 |
| 实时DRC | get/start/stop real-time DRC | 显式status/start/stop工具 |
| 原理图变更导入 | `PCB_Document.importChanges` | 关联原理图、SHA-256和运行时状态守卫后调用，随后完整读回 |
| 关联网表比较 | `SYS_Tool.netlistComparison` | PCB MCP双读并规范化受限枚举；运行时大小写漂移单独适配 |
| 制造文件 | `PCB_ManufactureData.getGerberFile/getPickAndPlaceFile/getBomFile/getTestPointFile/getNetlistFile/getIpcD356AFile` | PCB MCP采用有界File传输、签名/哈希和创建式磁盘读回，不打开对话框或下单 |
| 当前tab图像与图层 | `DMT_EditorControl.getCurrentRenderedAreaImage`、`PCB_Layer`显隐接口 | 当前视口PNG；可选隔离已启用层并在完整读回一致后成功 |
| 全板/区域检查图 | 类型化图元、层与Poured读取 | 两次稳定快照在宿主生成SVG，不改变编辑器视口、焦点、选择或图层 |

## 4. 客户端版本差异

EasyEDA Pro 3.2.186实时探测结果：

- 文本、属性、原生点/区域查询、约束组、实时DRC、原理图导入、网表比较、六类制造File、指定tab图像及图层显隐方法存在；
- `sys_Tool.netlistComparison`的运行时type返回`NET`/`COMPONENT`，与类型示例大小写不同；适配只接受这两个已知值；
- 无参数`dmt_EditorControl.zoomTo`在该版本抛出内部参数错误，公开API没有独立精确视口边界读取器；不据此实现伪恢复的fit-all截图；
- 图层状态存在0、1、2，隔离仅操作已启用的1/2状态层，0状态保持不变；
- 静态 `pcb_PrimitivePour.rebuildCopperRegions` 不存在；
- `pcb_PrimitivePour.getAll()` 返回的Pour实例具备 `rebuildCopperRegion()`；
- 自动布局、自动布线、清空走线和制造下单方法即使存在也不暴露；MCP只报告能力与策略排除。

类型声明和运行时可能不同。先用 `pcb_capabilities` 对目标客户端探测，再选工具，不能因为新版类型存在某方法就假定旧客户端可调用。

## 5. 仍需浏览器/视觉能力的范围

以下任务没有被公开API工具完全替代：

- 丝印真实字形、阻焊开窗、3D装配、连接器插合/遮挡和视觉密度检查；
- 客户端没有稳定公开API的专属交互面板；
- 需要人工判读的复杂制造和装配预览；
- 原生器件体/庭院、精确板框及字形级几何未被类型化SVG完整重建的部分。

浏览器仅作为这些能力的受控补充。覆铜重建、独立/属性文字、约束组、DRC、原生点选、受保护原理图导入、关联网表比较、制造File、当前视口PNG和全板/区域SVG不再默认通过页面点击、图层面板、缩放或导出对话框。

## 6. 何时开发单独扩展

满足以下任一条件时，才从该SDK派生独立扩展：

- 需要持久菜单、生命周期回调、面板/iframe UI或用户本地交互；
- 需要在EasyEDA内部提供独立功能入口，且现有Bridge/MCP不适合；
- 公开API具备能力，但跨Bridge的数据类型无法安全表达，并且扩展内部可完成闭环。

开发时从固定提交建立新工程，不直接修改审查缓存；记录API类型版本、客户端最低版本和权限；打包前执行类型检查、单元测试和独立测试工程验证。
