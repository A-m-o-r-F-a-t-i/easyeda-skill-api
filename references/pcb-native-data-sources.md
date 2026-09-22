# PCB 原生数据来源

PCB Info来自pcb_ManufactureData.getPcbInfoFile，DSN来自getDsnFile，源码哈希来自sys_FileManager.getDocumentSource。
解析器保留原始SHA-256、未知行、冲突字段和缺失字段；无法识别的统计不填成零。

实测3.2.186的中文标签中，“器件”与“元件”不同。componentCount只对应元件实例，deviceCount保留器件统计。
DSN可能将整板聚合到一个image/place，其pin记录可以包含路由逻辑端点。DSN pin计数不能替代PCB物理焊盘数。
DSN坐标采用声明单位，resolution不是缩放除数。MCP标准场景统一为mm，同时保存sourceUnits/resolution与来源坐标系。
没有验证DSN到API坐标变换时，apiCoordinateTransform=null，禁止将场景坐标直接用于编辑。
