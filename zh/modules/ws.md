---
outline: deep
---

# ws

## 职责定位

`ws` 是 infrago 的轻量实时客户端通道模块。

它主要解决这些问题：

- 接管 `http/web.Context.Upgrade()` 后的 WebSocket 连接
- 管理 `Conn / Session / User / Group`
- 定义客户端上行 `Message` 与服务端下行 `Command`
- 提供 `Reply / Push / PushUser / Broadcast / Groupcast`
- 提供基础背压、队列、压缩、导出和运行指标

它**不是**完整实时业务平台：

- 业务事件编排仍然优先走 `bus`
- 更重的 presence / ack / offline / state sync 可以以后在 `ws` 之上扩展

## 分层边界

- `http/web`：路由、鉴权、参数、`Upgrade`
- `ws`：连接、协议、会话、客户端实时消息
- `bus`：节点间广播、按用户/组/全量分发

推荐理解为：

- `bus` 负责模块与节点之间的消息
- `ws` 负责节点到客户端之间的实时通道

## Upgrade 与 Space

`http/web` 直接依赖 `ws`，`ctx.Upgrade(spaces ...string)` 会在升级后进入 `ws.Accept(...)`。

默认规则：

- `ctx.Upgrade()`：默认 `space = ctx.Name`
- 如果 `ctx.Name == ""`，则回退 `infra.DEFAULT`
- `ctx.Upgrade("custom")`：显式使用 `custom` 空间

这样做的目的不是接入多种 websocket 实现，而是让同一项目中的多个 websocket 场景天然隔离。

## 基本模型

### 连接入口

```go
infra.Register(".socket", web.Router{
    Uri:  "/socket",
    Name: "socket",
    Action: func(ctx *web.Context) {
        if err := ctx.Upgrade(); err != nil {
            ctx.Error(infra.Fail.With(err.Error()))
        }
    },
})
```

### 自定义 Space

```go
infra.Register(".socket.custom", web.Router{
    Uri:  "/socket/custom",
    Name: "custom socket",
    Action: func(ctx *web.Context) {
        if err := ctx.Upgrade("custom"); err != nil {
            ctx.Error(infra.Fail.With(err.Error()))
        }
    },
})
```

### 上行消息

```go
infra.Register("demo.echo", ws.Message{
    Name: "echo",
    Args: base.Vars{
        "text": {Type: "string", Required: true},
    },
    Action: func(ctx *ws.Context) {
        _ = ctx.Reply("demo.echoed", base.Map{
            "text": ctx.Value["text"],
        })
    },
})
```

### 下行命令

```go
infra.Register("demo.echoed", ws.Command{
    Name: "echoed",
    Args: base.Vars{
        "text": {Type: "string"},
    },
})
```

## 协议结构

默认下行 envelope：

```json
{"code":0,"name":"demo.notice","data":{"text":"hello"},"time":1770000000}
```

字段语义：

- `code`：`0` 表示成功，其它表示失败
- `name`：消息名 / 命令名
- `data`：业务数据
- `text`：失败文案，成功时通常为空
- `time`：服务器时间戳

接收端兼容：

- 消息名：`name` / `msg`
- 参数体：`data` / `args`
- 若没有 `data/args`，则把除消息名外的其它字段全部并入参数

## 上下行类型

- `ws.Message`
  客户端发给服务端的上行消息
- `ws.Command`
  服务端发给客户端的下行命令

这种拆分比把上下行都叫 event 更清楚，尤其在做协议导出、文档和权限控制时更稳。

## Space 隔离

`ws` 内部这些能力都按 `space` 隔离：

- `Message / Command`
- `Hook / Filter / Handler`
- `Session / User / Group`
- `Broadcast / Groupcast / PushUser`
- 节点间 `_ws.dispatch`

注册与查找规则：

- `Message / Command / Handler`
  先查当前 `space`，找不到再回退全局 `infra.DEFAULT`
- `Filter / Hook`
  执行全局 `infra.DEFAULT` + 当前 `space`

如果不在 `ctx` 内，但要显式指定空间，可以使用：

- `ws.PushUserIn(space, uid, msg, data)`
- `ws.BroadcastIn(space, msg, data)`
- `ws.GroupcastIn(space, gid, msg, data)`

## Hook / Filter / Handler

- `ws.Hook`
  原始生命周期钩子：`Open / Receive / Send / Close`
- `ws.Filter`
  入站 `Message` 执行链
- `ws.Handler`
  异常处理：`Invalid / Denied / Error`

如果要统一错误输出，推荐在 `Handler` 中调用：

```go
_ = ctx.Answer("demo.notice", nil, ctx.Result())
```

## Session / User / Group

`ws` 内部维护：

- `session -> conn`
- `user -> sessions`
- `group -> sessions`

常用方法：

- `ctx.BindUser(uid)`
- `ctx.Join(group...)`
- `ctx.Leave(group...)`
- `ctx.LeaveAll()`
- `ctx.InGroup(group)`
- `ctx.Groups()`

## 发送 API

- `ctx.Reply(name, data)`
- `ctx.Answer(name, data, res)`
- `ctx.Push(sid, name, data)`
- `ctx.PushUser(uid, name, data)`
- `ctx.Broadcast(name, data)`
- `ctx.Groupcast(gid, name, data)`

带统计版本：

- `ReplyResult`
- `PushResult`
- `PushUserResult`
- `BroadcastResult`
- `GroupcastResult`

这些统计主要表示“是否成功进入本地发送路径/队列”，不是客户端最终确认收到。

## 配置

```toml
[ws]
format = "text"
codec = "json"
ping_interval = "30s"
read_timeout = "75s"
write_timeout = "10s"
max_message_size = 4194304
queue_size = 128
queue_policy = "close"
compression = false
compress_level = 0
observe_interval = "30s"
observe_log = false
observe_trace = false
```

核心字段：

- `format`：`text` / `binary`
- `codec`：默认 `json`
- `ping_interval` / `read_timeout` / `write_timeout`
- `max_message_size`
- `queue_size` / `queue_policy`
- `compression` / `compress_level`
- `observe_interval` / `observe_log` / `observe_trace`

协议字段固定为：

- 请求：`name` + `data`
- 响应：`code` + `name` + `data` + `text` + `time`
- 输入端继续兼容 `msg` / `args`

## 队列与背压

模块级发送队列支持：

- `block`
- `close`
- `drop`

另外 `ws.Command.Setting` 支持更细粒度控制：

```go
ws.Command{
    Setting: base.Map{
        "priority": "high",      // high | normal | low
        "queue_policy": "block", // 可选覆盖
    },
}
```

默认策略：

- `high`：优先保留，默认按 `block`
- `normal`：走模块默认策略
- `low`：高压时自动丢弃，默认按 `drop`

## 协议导出

`ws.Export()` 返回前端友好的文档结构，包括：

- 配置
- 导出 schema 元信息
- 请求/响应 envelope
- `spaces` 汇总视图
- `Message / Command` 定义
- 每条协议的 `request / response` 示例
- 内置错误码说明

其中：

- `space` 只作为导出文档中的隔离元信息，不会出现在实际 websocket 包体里
- 顶层 `schema.version` 用于前端缓存、校验和协议展示版本对齐
- `spaces` 会统计每个空间下的消息、命令、过滤器、处理器、钩子数量
- `messages / commands` 仍然按 `space` 分组导出，便于程序直接读取

推荐把它暴露到一个调试接口，方便前端查看协议。

## 运行指标

`ws.Metrics()` 返回：

- `connections`
- `users`
- `messages_received`
- `messages_sent`
- `receive_failed`
- `send_failed`
- `bytes_received`
- `bytes_sent`
- `avg_receive_bytes`
- `avg_send_bytes`
- `queued`
- `dropped`

其中：

- `queued` 是当前发送队列深度
- `dropped` 是累计被丢弃的消息数

如果配置了：

- `observe_log = true`
- `observe_trace = true`

模块会周期性把这些指标发到 `log` / `trace`。

## 设计建议

对 infrago 来说，`ws` 更适合作为“轻量实时通道模块”，而不是直接做成重型 realtime 平台。

建议把边界保持在：

- 连接
- 协议
- 会话
- 分发
- 观测

而把更重的业务语义继续留给：

- `bus`
- 以后可能出现的更高层实时模块
