# common

openJiuwen 的公共模块提供异常、日志、常量、schema、安全、工具等能力。模型相关配置见 [foundation.llm](./foundation/llm.README.md)（`ModelRequestConfig`、`ModelClientConfig` 等）。

**Classes**：

| CLASS | DESCRIPTION |
|-------|-------------|
| [JiuWenBaseException](./common/exception/exception.md#class-openjiuwencorecommonexceptionexceptionjiuwenbaseexception) | openJiuwen 异常基类。 |
| [StatusCode](./common/exception/status_code.md#class-openjiuwencorecommonexceptionstatus_codestatuscode) | 错误码信息。 |
| [logger](./common/logging/__init__.md#openjiuwencorecommonlogginglogger) | 默认日志器。 |
| [LoggerProtocol](./common/logging/protocol.md#class-openjiuwencorecommonloggingprotocolloggerprotocol) | 自定义日志器基类。 |
| [LogManager](./common/logging/manager.md#class-openjiuwencorecommonloggingmanagerlogmanager) | 日志器管理类。 |
