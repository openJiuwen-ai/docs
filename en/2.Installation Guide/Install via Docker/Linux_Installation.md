This guide describes how to install openJiuwen on Linux using Docker.

## I. Environment Preparation

Make sure your machine meets the following requirements:

- Hardware:
  - CPU: Minimum 2 cores, 4+ cores recommended
  - RAM: Minimum 4 GB, 8+ GB recommended

- Operating System:
  - Ubuntu: Minimum Ubuntu 20.04, Ubuntu 22.04 (Jammy) or later recommended
    > **Note**: Ubuntu official and mainstream software repositories have stopped supporting Ubuntu 20.04 (Focal) and earlier.
  - EulerOS: Huawei Cloud EulerOS 2.0 or later

- Software:
  - Docker and Docker Compose: Installation methods are described below

### Install Docker and Docker Compose

- Refer to the <a href="https://docs.docker.com/engine/install/" target="_blank" rel="nofollow noopener noreferrer">Docker official installation guide</a> and the <a href="https://docs.docker.com/compose/install/" target="_blank" rel="nofollow noopener noreferrer">Docker Compose official installation guide</a> to complete the setup.

- Verify the installation of Docker and Docker Compose:

    ```
    docker version
    docker-compose version
    ```

## II. Installing openJiuwen (Ubuntu 22.04 as an example)

### 1. Download the release package (skip if you already have it)

- Download the version package based on the machine architecture:

  - Download the x86_64 package:
    ```
    wget https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_0.1.2_amd64.zip
    ```

  - Download the arm package:
    ```
    wget https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_0.1.2_arm64.zip
    ```

### 2. Start openJiuwen

- Place the release package in the installation directory.

- install unzip tool
  ```bash
  sudo apt update && sudo apt install unzip -y
  ```

- Extract the corresponding architecture version package
  - Extract the x86_64 package
    ```
    unzip deployTool_0.1.2_amd64
    ```

  - Extract the arm package
    ```
    unzip deployTool_0.1.2_arm64
    ```

- Enter the *deployTool_0.1.2_xxx64* directory and run the following commands to confirm Docker is running:

  ```bash
  sudo systemctl start docker
  sudo systemctl status docker
  ```
  > **Note**: If the output shows “inactive”, refer to the <a href="https://docs.docker.com/engine/install/" target="_blank" rel="nofollow noopener noreferrer">Docker official installation guide</a> and the <a href="https://docs.docker.com/compose/install/" target="_blank" rel="nofollow noopener noreferrer">Docker Compose official installation guide</a>.

  > **Note**: If you need to enable the memory feature, refer to [How to Enable the Memory Feature](#docker-linux-memory) for configuration.

- Run the following command to start openJiuwen:

  ```bash
  ./service.sh up
  ```

  > **Note**: You may see “up Plugin + Sandbox Server failed” due to network issues. Please rerun `./service.sh up`.

- Upon successful startup, it will output:

  Local access: *local access address*

  Network access: *network access address*

### 3. Access the system

- For local access, copy the *local access address* above into your browser and press Enter to open the openJiuwen interface.

- For access from another machine, copy the *network access address* above into your browser and press Enter to open the openJiuwen interface.

## III. Frequently Asked Questions (FAQ)

### Question 1: Why does the Milvus container suddenly exit during use?

The current version milvus 2.6.2 requires the AVX instruction set in the CPU. If it’s not available, milvus will exit automatically. You can check the CPU flags via `lscpu | grep Flags`.

### <a id="docker-linux-memory"></a>Question 2: How to enable the memory feature

The effectiveness of the memory feature depends on the parameter scale of the large language model.

The memory feature depends on an embedding model. The following steps use Huawei Cloud as an example to obtain an embedding model.

- Click <a href="https://console.huaweicloud.com/modelarts/?locale=zh-cn&region=cn-southwest-2#/model-studio/square" target="_blank" rel="nofollow noopener noreferrer">this link</a> to go to the ModelArts Model Square.

- Click “Vector Models” and find the BGE-M3 model.

  ![Find the embedding model](../images/find_embed.png)

- After finding the BGE-M3 model, click to invoke for inference to enter the model information page.

  ![Get api_base and model_name](../images/embed_api_base_and_model_name.png)

- Record the API address (corresponds to EMBED_API_BASE) and the model parameter (corresponds to EMBED_MODEL_NAME).

- Click “API Key Management” and follow the official instructions to obtain an API Key (corresponds to EMBED_API_KEY).

- After obtaining the embedding model information, configure it in the openJiuwen installation directory as follows:

- If this is the first time starting the openJiuwen platform, add the embedding-related information to .env.custom:

  | Variable Name | Description |
  | --- | --- |
  | **EMBEDDING_MODEL_DIMENTION** | The dimension of the embedding model, determined by the selected EMBED_MODEL_NAME |
  | **EMBED_API_BASE** | The endpoint of the embedding model |
  | **EMBED_MODEL_NAME** | The name of the embedding model |
  | **EMBED_API_KEY** | The API key for the embedding model |
  | **EMBED_TIMEOUT** | The maximum wait time for embedding model requests |
  | **EMBED_MAX_RETRIES** | The maximum number of retries when an embedding model request fails |

- After configuration, start the openJiuwen platform to use the memory feature.

- If enabling the memory feature after openJiuwen has already started, add the embedding-related information to .env; After configuring, restart the openJiuwen platform to apply the settings:

  ```
  ./service.sh up
  ```

> **Note**: After configuring EMBEDDING_MODEL_DIMENTION and enabling the memory feature, do not modify it again, otherwise the memory feature will not work. It is also not recommended to modify other embedding model configurations, as they may affect quality.

### Question 3: Docker deployment fails on openEuler?

On openEuler, Docker deployment may be restricted by the kernel security mechanism seccomp (Secure Computing Model) when creating threads.

Please refer to the <a href="https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS/docs/Container/%E5%AE%89%E5%85%A8%E7%89%B9%E6%80%A7.html" target="_blank" rel="nofollow noopener noreferrer">official guide</a> to adjust the corresponding seccomp security policy. The docker-compose deployment file is *conf/docker-jiuwen.template.yml* in the openJiuwen installation directory.

### Question 4: Docker image list included with openJiuwen

| Image | Version | License | Source Code |
| ------ | ---------------------------- | ------------- | ------------------------------------------------------------ |
| mysql  | 8.4.5                        | GPL 2.0       | <a href="https://github.com/mysql/mysql-server/tree/mysql-8.4.5" target="_blank" rel="nofollow noopener noreferrer"> Source link</a>       |
| minio  | RELEASE.2024-12-18T13-15-44Z | GNU AGPL 3.0      | <a href="https://github.com/minio/minio/tree/RELEASE.2024-12-18T13-15-44Z" target="_blank" rel="nofollow noopener noreferrer"> Source link</a> |
| milvus | 2.6.2                       | Apache 2.0    | -                                                            |
| etcd   | 3.5.18                      | Apache 2.0    | -                                                            |

### Question 5: How to stop openJiuwen

Run the following command to stop openJiuwen:

```
./service.sh down
```