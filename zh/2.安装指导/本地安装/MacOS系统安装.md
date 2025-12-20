本指南介绍在 MacOS 系统采用本地方式安装 openJiuwen。

## 一、环境准备

请确保机器满足以下要求：

* 硬件：
  * CPU：最低 2 核，推荐 4 核及以上
  * RAM：最低 4GB，推荐 8GB 及以上

* 操作系统：MacOS14.0（Sonoma）及以上

* 软件（安装方法详见下文）
  * Git 2.40及以上
  * Node.js 20.0及以上
  * npm 9.0及以上
  * Python 3.11.4及以上
  * uv 0.5.0及以上
  * MySQL 8.0及以上
  * Milvus 2.6.2及以上

## 二、安装依赖

进行正式安装前需先完成依赖的安装，再执行源码获取和安装等后续步骤。

### 1. 安装 Git

* 在 “终端” 中运行以下命令进行安装：
    ```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" # 若未安装Homebrew

    brew install git
    ```

* 安装完成后，在“终端”中输入：`git --version`，若安装成功会输出 git 版本号。

### 2. 安装 Node.js 和 npm

* 前往 <a href="http://nodejs.cn/download/" target="_blank" rel="nofollow noopener noreferrer"> Node.js官网</a> 下载Node.js 20.0及以上的MacOS安装包，双击安装包按照提示完成安装。
* 安装完成后，打开 “终端”，分别输入：`node -v` 与 `npm -v`，若安装成功会输出 node 与 npm 版本号。

### 3. 安装 Python 和 uv

* 运行以下命令下载并安装pyhon3.11

  ```
  brew install python@3.11
  ```

* 安装完成后，打开 “终端”，输入：`python3 --version`，若安装成功会输出 python 版本号。

* 打开 “终端”，安装 `uv`：
   
   ```bash
   brew install uv
   ```
* 输入：`uv --version`，若安装成功会输出 uv 版本号。

### 4. 安装 MySQL

* 打开 “终端”，运行以下命令安装MySQL：

  ```
  brew install pkg-config # 若未安装pkg-config（用于发现mysql）
  brew install mysql
  ```

* 安装完成后，打开 “终端”，运行以下命令启动并登录 MySQL：
   
   ```bash
   brew services start mysql
   mysql -u root
   ```

* 在 MySQL 中执行以下命令创建数据库：
  > 说明：`your_user_name`、`your_password` 可自行设置。

  ```sql
  CREATE DATABASE jiuwen_agent;
  CREATE DATABASE jiuwen_ops;
  CREATE USER 'your_user_name'@'localhost' IDENTIFIED BY 'your_password';
  GRANT ALL PRIVILEGES ON jiuwen_agent.* TO 'your_user_name'@'localhost';
  GRANT ALL PRIVILEGES ON jiuwen_ops.* TO 'your_user_name'@'localhost';
  FLUSH PRIVILEGES;
  ```

### 5. Milvus（可选组件）

openJiuwen 的记忆功能需依赖 Milvus 支撑，若需要体验记忆功能，可参考 [如何启用记忆功能](#macos-memory) 完成 Milvus 的安装配置。若仅需快速部署并体验 openJiuwen 基础功能，可直接跳过本步骤。

## 三、openJiuwen 安装

### 1. 获取源码

* 请确认已获取 <a href="https://gitcode.com/org/openJiuwen" target="_blank" rel="nofollow noopener noreferrer"> openJiuwen 代码仓</a> 的访问权限，若无权限请及时申请。

* 在 gitcode 代码仓按照图示步骤 2 获取 Git 的全局配置，输入以下命令配置 Git：

  ```bash
  git config --global user.name your_username
  git config --global user.email your_useremail
  ```

  ![image](../images/gitcode-token.png)

* 按照图示步骤 3 获取个人访问令牌，克隆代码时需要输入 gitcode 账号以及个人访问令牌。

* 打开 “终端”，在安装目录下执行以下命令克隆源码并进入源码目录：

  ```bash
  # 安装过程需要多次 git 操作，建议配置凭证存储，避免认证错误。
  git config --global credential.helper store

  git clone https://gitcode.com/openJiuwen/agent-studio.git
  cd agent-studio
  ```

### 2. 生成 AES 密钥（可选）

* 如果不需要对关键字段加密存储，可跳过当前步骤
* 运行以下命令生成密钥：

  ```bash
  cd backend
    
  bash build_AES_master_key.sh
  ```

* 脚本执行完，会将密钥打屏输出，可按需使用，推荐作为环境变量使用并另行保存。

  ```bash
  export SERVER_AES_MASTER_KEY_ENV=your_aes_key
  ```

* 注意，AES密钥需要保持稳定，中途更换密钥会导致已加密数据无法解密。

### 3. 启动 openJiuwen

* 在源码目录打开 “终端”；

* 复制 `.env` 文件：
  ```bash
  cp .env.example .env
  ```

* 在“终端”中使用命令`open .env`打开 `.env` 文件，请按实际情况修改下述配置：
   
   > **说明**：DB_HOST、DB_PORT、DB_USER、DB_PASSWORD 等变量的值可替换为上文 MySQL中创建的数据库信息。
    
   ```env
   # 配置数据库（样例）
   DB_HOST=localhost
   DB_PORT=3306
   DB_USER=your_user_name
   DB_PASSWORD=your_password

  # 配置Milvus（样例）
   MILVUS_HOST="127.0.0.1"
   MILVUS_PORT="19530"
   MILVUS_COLLECTION_NAME="memory_vector"

   # 记忆相关配置（如果不使用记忆功能，可以不提供下面的参数）
   EMBEDDING_MODEL_DIMENTION=1024
   EMBED_API_BASE=""
   EMBED_MODEL_NAME=""
   EMBED_API_KEY=""
   EMBED_TIMEOUT=5
   EMBED_MAX_RETRIES=1
   ```

  变量说明可参考如下表格，如需启用记忆功能，请参考[如何启用记忆功能](#macos-memory)

   | 变量名                                   | 变量说明                                                               | 配置样例                                                                      |
   |---------------------------------------|--------------------------------------------------------------------|---------------------------------------------------------------------------|
   |**DB_HOST**                           | 数据库的主机地址                                                           | `localhost`                                                               |
   | **DB_PORT**                           | 数据库的端口号                                                            | `3306`                                                                    |
   | **DB_USER**                           | 数据库的用户名                                                            | `your_user_name`                                                             |
   | **DB_PASSWORD**                       | 数据库的密码                                                             | `your_password`                                                         |
   | **MILVUS_HOST**                 | Milvus服务的主机地址                                                | `127.0.0.1`                                                                    |
   | **MILVUS_PORT**                 | Milvus服务的端口                                                | `19530`                                                                    |
   | **MILVUS_COLLECTION_NAME**                 | Milvus服务的数据库名                                                | `memory_vector`                                                                    |  **EMBEDDING_MODEL_DIMENTION**         | 嵌入模型的维度，根据EMBED_MODEL_NAME选择的模型确定                | `1024`                                                                    |                  
   | **EMBED_API_BASE**                    | 嵌入模型的接口地址                                                  | `https://example.com/embedding_model`            |            
   | **EMBED_MODEL_NAME**                  | 嵌入模型的名称                                                             | `text-embedding-model`                                                       |
   | **EMBED_API_KEY**                     | 嵌入模型的API密钥，换成自己的                                                 | `sk-xxx`                                                                  |
   | **EMBED_TIMEOUT**                     | 嵌入模型的最大等待时间                                                       | `5`                                                                     |
   | **EMBED_MAX_RETRIES**                 | 嵌入模型请求失败时的最大重试次数                                                | `1`                                                                    |

* 打开一个 “终端”，在源码目录下，逐行单独运行以下命令启动后端服务：
   
  ```bash
  cd backend
  uv venv
  uv sync
  mkdir logs
  mkdir logs/run
  source .venv/bin/activate
  python main.py
  ```

  > **注意**：若执行 `uv sync` 失败，可尝试：`uv sync --native-tls`  强制使用系统原生TLS库（解决HTTPS下载兼容问题）
  
  > **注意**：部分用户在执行python main.py可能会遇到No Module named 'greenlet'的问题，请移步FAQ问题二查看解决方法

  启动成功后，会输出 “Application startup complete”。

* 再打开一个 “终端”，在源码目录下，逐行单独运行以下命令启动前端服务：

  ```bash
  cd frontend
  npm install
  npm run dev
  ```

* 启动成功后会输出:

  Local：*本地访问地址*

  Network：*网络访问地址*


### 4. 访问系统

  * 若在本地查看，control+单击 *本地访问地址* ，接着单击打开链接，可在本地浏览器查看到 openJiuwen 的界面；或者复制上述 *本地访问地址* 到浏览器地址栏，按下“回车键”将看到 openJiuwen 的界面。
  
  * 若在外部机器查看，复制上述 *网络访问地址* 到浏览器地址栏，按下“回车键”将看到 openJiuwen 的界面。

## 四、常见问题（FAQ）

### <a id="macos-memory"></a> 问题一：如何启用记忆功能

记忆功能的体验与大模型的参数规模相关。

记忆功能依赖 Milvus，MacOS 系统推荐使用 Docker 安装，具体安装步骤可参考下文。

#### 1. 安装 Docker Desktop

* 下载：访问 <a href="https://www.docker.com/products/docker-desktop/" rel="nofollow">Docker Desktop 官网</a>，点击 “Download for Mac” 获取 .dmg 安装包。；
* 双击下载的文件，将 **Docker** 图标 拖拽到 Applications 文件夹；

  ![docker1](../images/docker拖拽.png)

* 打开 Launchpad，找到并启动 Docker 应用；
* 首次运行时，系统会提示输入 macOS 密码以授权安装虚拟机组件，点击 OK 继续；
* 首次启动需等待 Docker 完成初始化（下载基础镜像，约需几分钟）。

* 至此 Docker Desktop 安装完成。

> **说明**：若安装过程中出现报错，请参考 <a href="https://docs.docker.com/desktop/setup/install/windows-install/" rel="nofollow">Docker Desktop 官方安装指导</a>。

#### 2. 启动 Milvus

* 在 “终端” 中执行以下命令，将在当前目录下保存 “standalone_embed.sh” 脚本

  ```
  curl -fsSL https://raw.githubusercontent.com/milvus-io/milvus/master/scripts/standalone_embed.sh -o standalone_embed.sh
  ```

* 在 “终端” 执行以下命令拉取镜像：

  ```bash
  # arm 架构
  docker pull swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-arm64:v2.6.2

  # x86 架构
  docker pull swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-amd64:v2.6.2
  ```

* 将 “standalone_embed.sh” 文件内的 milvus 官方镜像名（比如： `milvusdb/milvus:v2.6.7`） 内容修改为 对应的镜像名（arm机器镜像名：`swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-arm64:v2.6.2`）。
  
* 修改完成后，在 “终端” 执行以下命令运行，将 Milvus 作为 Docker 容器启动：

  ```
  bash standalone_embed.sh start
  ```

* 启动后，输入 `docker ps -a` 命令可查看到名为 Milvus-standalone 的 docker 容器在 `19530` 端口启动。

  > **说明**：若在部署过程中出现问题可参考 <a href="https://milvus.io/docs/zh/install_standalone-docker.md" target="_blank" rel="nofollow noopener noreferrer"> Milvus官方指导文档</a>

* 若要停止 Milvus，请执行以下命令

  ```
  bash standalone_embed.sh stop
  ```

#### 3. 获取记忆功能所需的向量模型

记忆功能的运行依赖向量模型，以下流程以华为云为例，介绍向量模型的获取步骤。

* 点击<a href="https://console.huaweicloud.com/modelarts/?locale=zh-cn&region=cn-southwest-2#/model-studio/square" target="_blank" rel="nofollow noopener noreferrer"> 链接</a> 进入模型广场。 

* 点击 “向量模型”，找到 BGE-M3 模型。

  ![找到embedd模型](../images/find_embed.png)

* 找到 BGE-M3 模型后点击推理调用，进入模型信息获取界面。

  ![获取api_base和model_name](../images/embed_api_base_and_model_name.png)

* 记录API地址（对应 EMBED_API_BASE）、model参数（对应 EMBED_MODEL_NAME）。

* 点击 “API Key 管理”，按照官方界面引导获取 API Key（对应 EMBED_API_KEY）。

### 问题二：在后端启动时出现No Module named 'greenlet'应该怎么解决？

部分Apple Silicon芯片的Mac上的python会存在存在兼容问题，标准库中没有greenlet包。可以按如下方法解决：

  ```bash
  # 首先需要离开虚拟环境（如不在虚拟环境中可跳过此步）
  deactivate
  # 向虚拟环境中添加greenlet
  uv add greenlet
  # 接着再次进入虚拟环境完成接下来的操作
  source .venv/bin/activate
  ```
