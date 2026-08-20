# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

同一工作空间里有两条 staged 推理运行，各自绑定不同快照。只启动第一条时，第二条尚未启动的快照也被改成了 materializing。先只查原因，不要修改生产代码；测试和配置同样保持不变。请沿运行状态转换、输入列表查询和数据库过滤条件还原范围如何被放大，并指出实际被误改的关联记录。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-07
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-07.git
- parent SHA：1cede8ae8fc9b2273c00ffd0723a74e1f5b6fbab

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-07.git bug-repro
cd bug-repro
git checkout --detach 1cede8ae8fc9b2273c00ffd0723a74e1f5b6fbab
go test ./internal/service -run "^TestStartingRunDoesNotAdvanceSiblingRunSnapshots$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestStartingRunDoesNotAdvanceSiblingRunSnapshots$" -count=1
--- FAIL: TestStartingRunDoesNotAdvanceSiblingRunSnapshots (0.72s)
    annotation_core_behavior_test.go:224: sibling snapshot changed when another run started: {ID:snapshot_bf65af6b86b280ef57acc6cf WorkspaceID:workspace_9bd2375f2a50655395ba7758 SourceZoneID:data_zone_03ed7f4bf6ea56bc333af3ac SourceRevision:SIBLING-REV SchemaFamily:ranking-v5 PartitionCount:1 EstimatedRows:100 State:materializing ExpiresAt:2026-08-20 08:00:00 +0000 UTC InferenceRunID:run_1d3513700e7d2ba9ef5b2e65 QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	0.730s
FAIL

```

stderr：

```text
warning: internal/service/annotation_core_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_core_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestStartingRunDoesNotAdvanceSiblingRunSnapshots$" -count=1
--- FAIL: TestStartingRunDoesNotAdvanceSiblingRunSnapshots (1.50s)
    annotation_core_behavior_test.go:224: sibling snapshot changed when another run started: {ID:snapshot_36036185ce255162d4634acc WorkspaceID:workspace_b2715e9512a6868cfd4c4322 SourceZoneID:data_zone_dde8bf489fe0877e52ffb53b SourceRevision:SIBLING-REV SchemaFamily:ranking-v5 PartitionCount:1 EstimatedRows:100 State:materializing ExpiresAt:2026-08-20 08:00:00 +0000 UTC InferenceRunID:run_ccfd5dd70d90877ae3958f25 QuarantineNote: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:4}
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	1.697s
FAIL

```

stderr：

```text
warning: internal/service/annotation_core_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_core_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

准确定位根因，指出具体 Go 文件和符号，解释错误动作如何导致题面症状，并给出实际复现、调用链或持久化证据；目标仓库代码、测试和配置零改动。
