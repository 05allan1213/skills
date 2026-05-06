---
name: go-feature-dev
description: Use this skill when implementing new Go backend features, extending APIs, adding business modules, integrating configuration, cache, middleware, clients, or background tasks.
---

# Go Feature Development Skill

本 Skill 用于 Go 项目的新需求开发。

公共开发底线遵守项目根目录中的：

```text
AGENTS.md
CLAUDE.md
```

本 Skill 只补充“新需求开发”场景下的额外规则。

---

## 1. 适用场景

当用户要求以下任务时，使用本 Skill：

```text
新增业务功能
新增 HTTP API
扩展已有接口
新增 service / client / model
接入 Redis / MySQL / Prometheus / Kubernetes Client
新增配置项
新增中间件
新增后台任务
新增缓存、限流、告警、Webhook、WebSocket 等能力
```

---

## 2. 新需求开发原则

新需求开发必须遵守：

1. 优先实现最小可用闭环。
2. 不提前做复杂抽象。
3. 不为了未来可能需求过度设计。
4. 不一次性铺开多个方向。
5. 不把新功能和大规模重构混在一起。
6. 优先复用项目已有结构和风格。
7. 新增配置、依赖、接口前必须先说明影响。
8. 每个阶段完成后必须能独立验证。

---

## 3. 开发前必须确认

实现新功能前，必须先说明：

```text
需求目标：
xxx

现有相关代码：
- xxx
- xxx

建议拆分：
1. xxx
2. xxx
3. xxx

本次先做的最小闭环：
xxx

预计修改文件：
- xxx
- xxx

新增内容：
- API：是否新增
- 配置：是否新增
- 依赖：是否新增
- 数据结构：是否新增
- 后台任务：是否新增

兼容性影响：
- xxx

验证方式：
- xxx

请确认是否开始。
```

---

## 4. 推荐拆分顺序

新需求优先按以下顺序拆：

```text
1. 数据结构 / 请求响应结构
2. 配置项和初始化逻辑
3. client 或 repository 封装
4. service 核心业务逻辑
5. handler / API 接入
6. 单元测试和表驱动测试
7. 文档和部署配置
```

不是所有需求都需要完整分层。

如果一个简单需求只需要改 handler 或 service，不要强行新增目录和复杂抽象。

---

## 5. API 开发约束

新增或修改 API 时必须说明：

```text
接口路径：
请求方法：
请求参数：
响应结构：
错误码：
是否兼容旧接口：
```

不得随意改变已有响应结构。

如果必须变更已有接口，必须提供：

```text
兼容方案
迁移说明
影响范围
测试方式
```

---

## 6. 配置开发约束

新增配置前必须说明：

```text
配置名：
默认值：
用途：
是否必填：
是否敏感：
本地如何配置：
Docker 如何配置：
K8s / Helm 如何配置：
```

配置必须集中管理，不允许散落在业务代码中。

敏感信息不得写入普通配置文件或日志。

---

## 7. 外部依赖接入约束

接入外部系统时，例如：

```text
Redis
MySQL
Prometheus
Kubernetes
HTTP 第三方服务
消息队列
```

必须考虑：

```text
timeout
context
错误处理
重试策略
降级策略
连接关闭
测试替身
```

禁止在业务逻辑中直接到处创建 client。

应优先封装为：

```text
client
repository
adapter
```

但不要过度设计。

---

## 8. 后台任务约束

新增 goroutine、定时任务、消费者、异步任务时必须考虑：

```text
context 取消
graceful shutdown
panic recover
错误日志
重试间隔
最大重试次数
避免 goroutine 泄漏
channel 关闭所有权
```

禁止：

```go
go func() {
    for {
        // 无退出条件
    }
}()
```

---

## 9. 测试重点

新需求优先补以下测试：

```text
参数校验
配置加载
核心业务逻辑
边界条件
错误分支
外部依赖失败
状态流转
```

推荐使用表驱动测试：

```go
tests := []struct {
    name    string
    input   xxx
    want    xxx
    wantErr bool
}{
    // cases
}
```

---

## 10. 开发后输出格式

每完成一个新功能小模块后，输出：

```text
本模块完成：
xxx

新增能力：
1. xxx
2. xxx

修改文件：
- xxx
- xxx

兼容性说明：
- API：xxx
- 配置：xxx
- 数据结构：xxx

已执行验证：
- goimports：通过/失败
- go test：通过/失败
- go vet：通过/失败

未执行：
- xxx，原因：xxx

风险与注意：
- xxx

建议提交：
git add xxx
git commit -m "feat: xxx"

请确认是否继续下一个模块。
```
