# 常见问题排查指南

本文档汇总 openJiuwen 使用过程中的常见问题与排查方法。如仍未解决，请携带以下信息提交 Issue。

## 1. pip 安装失败

**现象**：执行 `pip install openjiuwen` 报网络错误或找不到包。

**排查**：

- 确认包名正确（`openjiuwen`）；
- 确认 Python 版本满足要求（建议 Python 3.10+）；
- 如网络受限，可配置镜像源：`pip install openjiuwen -i https://pypi.tuna.tsinghua.edu.cn/simple`。

## 2. import 时报错 ModuleNotFoundError

**现象**：`import openjiuwen` 提示找不到模块。

**排查**：

- 确认安装成功：`pip show openjiuwen`；
- 确认当前环境与安装环境一致（虚拟环境是否激活）；
- 确认未混用多版本 Python。

## 3. 连接模型服务失败

**现象**：Agent 运行时无法连接模型服务，报连接超时/鉴权失败。

**排查**：

- 检查模型服务地址（endpoint）是否可访问；
- 检查 API Key 是否正确配置；
- 检查网络代理是否拦截了请求。

## 4. 多智能体任务超时

**现象**：Swarm 任务运行超时被终止。

**排查**：

- 检查任务超时/重试配置是否过小；
- 检查子 Agent 是否存在死循环；
- 查看日志定位卡住的环节。

## 5. 如何获取日志

- 运行日志默认输出到控制台，可通过环境变量调整日志级别；
- 提交 Issue 时请附上关键日志片段与版本号。

## 6. 提交 Issue 需要哪些信息

请按 Bug 报告模板填写：版本号、问题描述、复现步骤、期望/实际行为、运行环境。