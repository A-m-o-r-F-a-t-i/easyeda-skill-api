# Gateway Protocol v2

Bridge 保留 GET /health、GET /eda-windows、POST /execute，新协议使用 POST /rpc、GET /capabilities 和 GET /events。
请求字段为 operation、target、expected、arguments，可带 requestId；Bridge 生成实际传输关联 ID。
target 含 windowId/projectUuid/documentUuid/tabId。仅 system.capabilities 与 target.inspect 可以只给窗口身份。
expected 含 generationId/changeEpoch/sourceHash，可额外携带 bridgeGenerationId，由 Bridge 检查后再转发。

固定操作包括 capabilities、target.inspect、events.getState/getSince、pcb.read、pcb.nativeBoardInfo、pcb.exportDsn、document.sourceHash/createCheckpoint、pcb.applyGeometryBatch/applyTextBatch、pcb.nativeTools、pcb.constraints、pcb.save 与 pcb.drc。
写操作使用同一编辑器队列，要求准备状态和 executionId。固定 nativeTools/constraints 根据 operation arguments 区分读取与写入。

统一错误包含 INVALID_REQUEST、TARGET_CHANGED、GENERATION_MISMATCH、EPOCH_MISMATCH、CONCURRENT_CHANGE、CLIENT_UNSUPPORTED、METHOD_FAILED、PARTIAL_SUCCESS、FILE_TOO_LARGE、TRANSFER_HASH_MISMATCH、WINDOW_DISCONNECTED、REQUEST_TIMEOUT、PERMISSION_DENIED。
超时与断连不推断写入失败，details.outcome=unknown 时先检查现场。返回原结果的去重响应携带 replayed=true。
