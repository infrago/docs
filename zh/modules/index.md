---
outline: deep
---

# 模块总览

infrago 以模块化组织能力，每个模块都通过 `infra.Mount(module)` 接入统一生命周期。

## 阅读顺序

1. 先看 [infrago 核心模块](/zh/modules/infrago)
2. 再看基础模块：`config / log / cache / mutex`
3. 再看通信模块：`bus / event / queue`
4. 再看调度、搜索、数据与 Web：`cron / search / data / trace / http / web / ws / view / storage`

## 模块分类

- 核心：`infrago`
- 配置：`config`
- 日志与状态：`log` `cache` `mutex`
- 通信：`bus` `event` `queue`
- 调度：`cron`
- 搜索：`search`
- 数据：`data`
- 追踪：`trace`
- 接口与站点：`http` `web` `ws` `view`
- 文件：`storage`

## 通用配置习惯

绝大多数模块都支持：

- `driver`：选择驱动
- `setting`：驱动私有参数
- `weight`：多实例权重（部分模块）
- `prefix`：键或主题前缀（部分模块）
