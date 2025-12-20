本指南介绍在 Linux 系统采用 Docker 方式安装 openJiuwen

## 一、环境准备

请确保机器满足以下要求：

* 硬件：
  * CPU：最低 2 核，推荐 4 核及以上
  * RAM：最低 4GB，推荐 8GB 及以上

* 操作系统：
  * Ubuntu：最低 Ubuntu 20.04，推荐 Ubuntu 22.04 (Jammy) 及以上
    > **注意**：Ubuntu 官方与主流软件源已停止支持 Ubuntu 20.04 (Focal) 及以下版本系统。
  * EulerOS：Huawei Cloud EulerOS 2.0及以上

* 软件
  * Docker 和 Docker Compose：安装方法详见下文

### 安装 Docker 和 Docker Compose

* 请参照 <a href="https://docs.docker.com/engine/install/" target="_blank" rel="nofollow noopener noreferrer">Docker 官方安装指南</a> 以及 <a href="https://docs.docker.com/compose/install/" target="_blank" rel="nofollow noopener noreferrer">Docker Compose 官方安装指南</a> 完成配置。

* 验证 Docker 和 Docker Compose 安装:

    ```
    docker version
    docker-compose version
    ```

## 二、openJiuwen 安装（以下以 Ubuntu 22.04 为例）

### 1. 下载版本包

* 复制版本链接：

  x86_64架构下载链接：https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_v0.1.0_amd64.tar

  arm架构下载链接：https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_v0.1.0_arm64.tar

* 运行以下命令下载版本包（将版本包下载链接替换成上述下载链接）

    ```bash
    wget 版本包下载链接
    ```

### 2. 启动 openJiuwen
* 解压该版本包（将 xxx64 换为对应的机器架构）。

  ```bash
  tar -xf deployTool_v0.1.0_xxx64.tar
  ```

* 进入 *deployTool_v0.1.0_xxx64* 目录，输入以下命令确认 Docker 已启动：

  ```bash
  sudo systemctl start docker
  sudo systemctl status docker
  ```
  > **说明**：若输出 “inactive” ，请参考 <a href="https://docs.docker.com/engine/install/" target="_blank" rel="nofollow noopener noreferrer">Docker 官方安装指南</a> 以及 <a href="https://docs.docker.com/compose/install/" target="_blank" rel="nofollow noopener noreferrer"> Docker Compose 官方安装指南</a>。

  > **说明**：若需要启用记忆功能，可参考 [如何启用记忆功能](#docker-linux-memory) 进行配置。

* 输入以下命令启动 openJiuwen：

  ```bash
  ./service.sh up
  ```

* 启动成功后会输出 

  Local access: *本地访问地址*

  Network access: *网络访问地址*

* 若要停止 openJiuwen，请输入以下命令：

  ```
  ./service.sh down
  ```

### 3. 访问系统

* 若在本地查看，复制上述 *本地访问地址* 到浏览器地址栏，按下“回车键”将看到 openJiuwen 的界面。

* 若在外部机器查看，复制上述 *网络访问地址* 到浏览器地址栏，按下 “回车键” 将看到 openJiuwen 的界面。

## 三、常见问题（FAQ）

### 问题一：在使用过程中出现 milvus 容器突然退出是什么原因？

目前使用的 milvus2.6.2 需要 CPU 中存在 AVX 指令，如果不存在 milvus 会自动退出。可以通过 `lscpu | grep Flags` 查看 cpu 指令。

### <a id="docker-linux-memory"></a>问题二：如何启用记忆功能

记忆功能的体验与大模型的参数规模相关。
  
记忆功能的运行依赖向量模型，以下流程以华为云为例，介绍向量模型的获取步骤。

* 点击 <a href="https://console.huaweicloud.com/modelarts/?locale=zh-cn&region=cn-southwest-2#/model-studio/square" target="_blank" rel="nofollow noopener noreferrer"> 链接</a> 进入模型广场。 

* 点击 “向量模型”，找到 BGE-M3 模型。

  ![找到embedd模型](../images/find_embed.png)

* 找到 BGE-M3 模型后点击推理调用，进入模型信息获取界面。

  ![获取api_base和model_name](../images/embed_api_base_and_model_name.png)

* 记录API地址（对应 EMBED_API_BASE）、model参数（对应 EMBED_MODEL_NAME）。

* 点击 “API Key 管理”，按照官方界面引导获取 API Key（对应 EMBED_API_KEY）。

* 获取向量模型信息后，请在 *openJiuwen 的安装目录* 进行如下配置：

* 若是初次启动 openJiuwen 平台，请在 *.env.custom* 中添加 embedding 相关的信息：

  | 变量名 | 变量说明 |
  | --- | --- |
  | **EMBEDDING_MODEL_DIMENTION**         | 向量模型的维度，根据 EMBED_MODEL_NAME 选择的模型确定                |
  | **EMBED_API_BASE**                    | 向量模型的接口地址                                                  |            
  | **EMBED_MODEL_NAME**                  | 向量模型的名称                                                             |
  | **EMBED_API_KEY**                     | 向量模型的 API 密钥                               |
  | **EMBED_TIMEOUT**                     | 向量模型的最大等待时间 |
  | **EMBED_MAX_RETRIES**                 | 向量模型请求失败时的最大重试次数                 |

* 配置完成后启动 openJiuwen 平台即可使用记忆功能。

* 若是在启动 openJiuwen 之后启用记忆功能，请在 *.env* 文件同级目录运行 `cp .env.xxxxx .env`（xxxxx为需要使用记忆功能的容器运行时生成的随机码，可以通过docker ps -a查看），在 *.env* 中添加 embedding 相关的信息；配置完成后，重新启动 openJiuwen 平台使配置生效即可使用记忆功能：

  ```
  ./service.sh up -f .env
  ```

> **注意**：在配置 *EMBEDDING_MODEL_DIMENTION* 之后不要再次修改。

### 问题三：openEuler 环境中，Docker 部署失败？

openEuler 环境中，Docker 部署时创建线程会收到 seccomp(Secure Computing Model)内核安全机制的限制。

请参考 <a href="  https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS/docs/Container/%E5%AE%89%E5%85%A8%E7%89%B9%E6%80%A7.html" target="_blank" rel="nofollow noopener noreferrer"> 官方指导</a>，调整相应的 seccomp 安全策略，对应的 docker-compose 部署文件是 openJiuwen 安装目录下：*conf/docker-jiuwen.template.yml* 文件。

### 问题四：openJiuwen 包含的 Docker 镜像清单

| 镜像名 | 镜像版本                     | license       | 源码地址                                                     |
| ------ | ---------------------------- | ------------- | ------------------------------------------------------------ |
| mysql  | 8.4.5                        | GPL 2.0       | <a href="https://github.com/mysql/mysql-server/tree/mysql-8.4.5" target="_blank" rel="nofollow noopener noreferrer"> 源码链接</a>       |
| minio  | RELEASE.2024-12-18T13-15-44Z | GNU AGPL 3.0      | <a href="https://github.com/minio/minio/tree/RELEASE.2024-12-18T13-15-44Z" target="_blank" rel="nofollow noopener noreferrer"> 源码链接</a> |
| milvus | 2.6.2                       | Apache 2.0    | -                                                            |
| etcd   | 3.5.18                      | Apache 2.0    | -                                                            |
