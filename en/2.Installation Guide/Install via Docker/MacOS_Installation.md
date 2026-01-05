This guide describes how to install openJiuwen on macOS via Docker.

## I. Environment Preparation

Ensure your machine meets the following requirements:

* Hardware: 
  * CPU: Minimum 2 cores, 4 cores or more recommended
  * RAM: Minimum 4GB, 8GB or more recommended

* Operating System: macOS 14.0 (Sonoma) or later

* Software:
  * Git: Install Git by running the following commands: 
    ```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" # If Homebrew is not installed

    brew install git
    ```

  * Docker: Docker Desktop is reocmmended. The installation steps are described below.

### Docker Desktop Installation

* Download: Visit the <a href="https://www.docker.com/products/docker-desktop/" rel="nofollow">Docker Desktop official website</a>, and click "Download for Mac" to download the .dmg installer.
* Double-click the installer and drag **Docker** into the Applications folder.

  ![docker1](../images/docker拖拽.png)

* Find and start the Docker application.
* Upon first opening Docker, the system will prompt you to enter your macOS password to authorize the installation of virtual machine components. Click OK to continue.
* The first startup will require waiting for Docker to complete initialization. (Downloading base images, which may take a few minutes)

* Docker Desktop installation complete.

> **Note**: If you encounter any errors during installation, please refer to the <a href="https://docs.docker.com/desktop/setup/install/mac-install/" rel="nofollow">official Docker Desktop installation guide</a>.


## II. openJiuwen Installation

### 1. Download the release package (Skip this step if already downloaded)

* Click the download link corresponding to your local machine to download the release package:

  x86_64 architecture download link: <a href="https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_v0.1.1_amd64.tar" target="_blank" rel="nofollow noopener noreferrer">openJiuwen v0.1.1</a>

  arm architecture download link: <a href="https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_v0.1.1_arm64.tar" target="_blank" rel="nofollow noopener noreferrer">openJiuwen v0.1.1</a>

### 2. Start openJiuwen

* Create a *openJiuwen installation directory*, place the release package in the installation directory and extract it.

* Navigate to the *openJiuwen installation directory*.

* Before running, run the following command to upgrade bash: 

  ```
  brew install bash
  ```

  > **Note**: If you need to enable the memory feature of openJiuwen, refer to [How to Enable the Memory Feature](#docker-macos-memory) for configuration instructions.

* Open **Terminal**, navigate to the directory where *service.sh* is located, and enter the following command to start openJiuwen: 

  ```bash
  ./service.sh up
  ```

  > **Note**: You may encounter the error message "up Plugin + Sandbox Server failed" due to network issues. Simply rerun `./service.sh up`.

* After a successful startup, the following information will be displayed:

  Local access: *local access address*

  Network access: *network access address*

### 3. Access the System

* To access locally, copy the above *local access address* into your browser's address bar and press "Enter" to see the openJiuwen interface.

* To access on another machine, copy the above *network access address* into your browser's address bar and press "Enter" to see the openJiuwen interface.

## III、Frequently Asked Questions (FAQ) 

### <a id="docker-macos-memory"></a>Question 1: How to Enable the Memory Feature

The effectiveness of the memory feature depends on the number of parameters of the LLM.
  
The memory feature relies on an embedding model. The following steps uses Huawei Cloud as an example to illustrate how to obtain an embedding model.


* Click <a href="https://console.huaweicloud.com/modelarts/?locale=zh-cn&region=cn-southwest-2#/model-studio/square" target="_blank" rel="nofollow noopener noreferrer">this link</a> to enter the Model Square. 

* Click "向量模型" (Embedding model), Locate the BGE-M3 model.

  ![Locate the embedding model](../images/find_embed.png)

* After locating the BGE-M3 model, click 推理调用 (Inference Call) to enter the model information acquisition page.

  ![Obtain api_base and model_name](../images/embed_api_base_and_model_name.png)

* Record the API address (corresponding to EMBED_API_BASE) and model parameters (corresponding to EMBED_MODEL_NAME).

* Click "API Key Management" and follow the instructions on the website to obtain an API Key (corresponding to EMBED_API_KEY).

* After obtaining the embedding model information, perform the following configuration in the *openJiuwen installation directory*: 

* If this is the first time starting the openJiuwen platform, add the following embedding-related information to *.env.custom*: 

  | Variable Name | Description |
  | --- | --- |
  | **EMBEDDING_MODEL_DIMENTION**         | Dimension of the embedding model, determined by the model specified in EMBED_MODEL_NAME                 |
  | **EMBED_API_BASE**                    | API endpoint of the embedding model                                                   |            
  | **EMBED_MODEL_NAME**                  | Name of the embedding model                                                   |
  | **EMBED_API_KEY**                     | API key for the embedding model (replace with your own)                                               |
  | **EMBED_TIMEOUT**                     | Maximum wait time for the embedding model<br>Default:`200`                                  |
  | **EMBED_MAX_RETRIES**                 | Maximum number of retries when an embedding request fails<br>Default:`1000`                                 |

* After completing the configuration, start the openJiuwen platform to use the memory feature.

* If you enable the memory feature after openJiuwen has already been started, add the embedding-related information to *.env*. After configuration, restart the openJiuwen platform for the settings to take effect:

  ```
  ./service.sh up
  ```

> **Note**: Once the memory feature has been enabled after configuring *EMBEDDING_MODEL_DIMENTION*, do not modify this value again, otherwise the memory feature will stop working. It is also not recommended to change other embedding model configurations, as doing so may affect performance or results.

### Question 2: What Docker Images are included in openJiuwen
  
| Image Name | Image Version                     | License       | Source Code                                                     |
| ------ | ---------------------------- | ------------- | ------------------------------------------------------------ |
| mysql  | 8.4.5                        | GPL 2.0       | <a href="https://github.com/mysql/mysql-server/tree/mysql-8.4.5" target="_blank" rel="nofollow noopener noreferrer">Source link</a>       |
| minio  | RELEASE.2024-12-18T13-15-44Z | GNU AGPL 3.0      | <a href="https://github.com/minio/minio/tree/RELEASE.2024-12-18T13-15-44Z" target="_blank" rel="nofollow noopener noreferrer">Source link</a> |
| milvus | 2.6.2                       | Apache 2.0    | -                                                            |
| etcd   | 3.5.18                      | Apache 2.0    | -                                                            |

### Question 3: How to stop openJiuwen

Run the following command to stop openJiuwen: 

```
./service.sh down
```
