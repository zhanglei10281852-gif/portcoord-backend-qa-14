# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

任务已经完成后再次执行会向调用方返回错误，而不是作为幂等重复请求处理。请修复重复执行的错误分类。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/portcoord-backend-qa-14
- 仓库地址：https://github.com/zhanglei10281852-gif/portcoord-backend-qa-14.git
- parent SHA：4b86f02e50fab75857f668c4204f44e0b3f6a91d

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/portcoord-backend-qa-14.git bug-repro
cd bug-repro
git checkout --detach 4b86f02e50fab75857f668c4204f44e0b3f6a91d
go test ./internal/worker -run "^TestWorker_IdempotentReport$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run "^TestWorker_IdempotentReport$" -count=1
--- FAIL: TestWorker_IdempotentReport (0.01s)
    worker_test.go:197: second execute should not error: task e540f7b3-e601-4b48-8c92-f812f07ce6e7 cannot be replayed: invalid_state: claim state rejected: invalid_state: illegal transition for pilot_task: completed -> claimed
FAIL
FAIL	portcoord/internal/worker	0.009s
FAIL

```

stderr：

```text
warning: internal/worker/worker_test.go has type 100755, expected 100644
warning: internal/worker/worker_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run "^TestWorker_IdempotentReport$" -count=1
--- FAIL: TestWorker_IdempotentReport (0.37s)
    worker_test.go:197: second execute should not error: task d22c36e2-50fd-4f01-8d06-741993c6e790 cannot be replayed: invalid_state: claim state rejected: invalid_state: illegal transition for pilot_task: completed -> claimed
FAIL
FAIL	portcoord/internal/worker	0.680s
FAIL

```

stderr：

```text
warning: internal/worker/worker_test.go has type 100755, expected 100644
warning: internal/worker/worker_test.go has type 100755, expected 100644

```

## 通过条件

在题面触发条件下，公开行为必须恢复且原始异常不再出现；定向命令 go test ./internal/worker -run ^TestWorker_IdempotentReport$ -count=1、相关包测试、全量测试、race、vet 和 build 必须通过；不得删除或跳过测试，也不得绕过目标逻辑。
