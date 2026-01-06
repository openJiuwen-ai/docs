This guide describes how to install openJiuwen locally on macOS.

## I. Environment Preparation

Ensure your machine meets the following requirements:

* Hardware: 
  * CPU: Minimum 2 cores, 4+ cores recommended
  * RAM: Minimum 4GB, 8GB+ recommended

* Operating System: macOS 14.0 (Sonoma) or later

* Software:
  * Git 2.40 or later
  * Node.js 20.0 or later
  * npm 9.0 or later
  * Python 3.11.4 or later
  * uv 0.5.0 or later
  * MySQL 8.0 or later
  * Milvus 2.6.2 or later

## II. Dependencies Installation

Before proceeding with the main installation, you must first install the required dependencies, then continue with source code retrieval and subsequent installation steps.

### 1. Install Git

* Run the following commands in "Terminal": 
    ```
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" # If Homebrew is not installed

    brew install git
    ```

* After installation, enter `git --version` in "Terminal". If the installation was successful, the Git version number will be displayed.

### 2. Install Node.js and npm

* Visit the <a href="http://nodejs.cn/download/" target="_blank" rel="nofollow noopener noreferrer">Node.js official website</a> and download the macOS installer for Node.js 20.0 or later. Double-click the installer and following the installation instructions to complete the installation.
* After installation, open "Terminal" and run `node -v` and `npm -v`. If the installation was successful, the Node.js and npm version numbers will be displayed. 

### 3. Install Python and uv

* Run the following command to download and install Python 3.11

  ```
  brew install python@3.11
  ```

* After installation, open "Terminal" and run `python3 --version`. If the installation was successful, the Python version number will be displayed. 

* Open "Terminal", install `uv`: 
   
   ```bash
   brew install uv
   ```
* Run `uv --version`. If the installation was successful, the uv version number will be displayed. 

### 4. Install MySQL (Optional Component)

* **Note**: `.env.example` defaults to SQLite. Simply keep `DB_TYPE` as `sqlite` to start the backend service directly—no additional installation or configuration is required. If you prefer to use MySQL, change `DB_TYPE` in `.env.example` to `mysql` and follow the steps below to install and configure MySQL.

* **SQLite vs MySQL**:
  * SQLite requires no extra setup and is suitable for development and testing, but it has limitations (e.g., no support for concurrent writes, no user permission management).
  * MySQL offers more robust features and is better suited for complex scenarios, making it the recommended choice for real-world projects and production environments.

* Open "Terminal" and run the following commands to install MySQL: 

  ```
  brew install pkg-config # If pkg-config is not installed (Used for discovering mysql) 
  brew install mysql
  ```

* After installation, open "Terminal" and run the following commands to start and log in to MySQL: 
   
   ```bash
   brew services start mysql
   mysql -u root
   ```

* Execute the following commands in MySQL to create the required databases:
  > **Note**: you can choose your own values for `your_user_name`、`your_password`. 

  ```sql
  -- Create databases
  CREATE DATABASE openjiuwen_agent;
  CREATE DATABASE openjiuwen_ops;
  -- Create MySQL user
  CREATE USER 'your_user_name'@'localhost' IDENTIFIED BY 'your_password';
  -- Grant privileges and flush
  GRANT ALL PRIVILEGES ON openjiuwen_agent.* TO 'your_user_name'@'localhost';
  GRANT ALL PRIVILEGES ON openjiuwen_ops.* TO 'your_user_name'@'localhost';
  FLUSH PRIVILEGES;
  ```

### 5. Milvus (Optional Component) 

The memory and knowledge base features of openJiuwen depend on Milvus. If you would like to use the memory and knowledge base features, refer to [How to Enable the Memory and Knowledge Base Features](#macos-memory) to complete the Milvus installation and configuration. If you only need a quick deployment to experience the basic features of openJiuwen, you can skip this step.

## III. openJiuwen Installation

### 1. Obtain the Source Code

* Please make sure you have access to the <a href="https://gitcode.com/org/openJiuwen" target="_blank" rel="nofollow noopener noreferrer">openJiuwen repository</a>, If you do not have access, apply for it in advance. 

* In the GitCode repository, follow Step 2 in the image to obtain Git global configuration information, and configure Git by running the following commands:

  ```bash
  git config --global user.name your_username
  git config --global user.email your_useremail
  ```

  ![image](../images/gitcode-token.png)

* Follow Step 3 in the image to obtain a Personal Access Token. When cloning the repository, you will need to enter your GitCode username and this personal access token.

* Open "Terminal", run the following commands in the installation directory to clone the source code and enter the project root directory: 

  ```bash
  # The installation process requires multiple git operations.
  # It is recommended to configure git credential storage to avoid authentication errors. 
  git config --global credential.helper store

  git clone https://gitcode.com/openJiuwen/agent-studio.git
  cd agent-studio
  ```

### 2. Generate an AES Key (Optional) 

* If you do not need to encrypt sensitive fields, you can skip this step.
* Run the following commands to generate the key: 

  ```bash
  cd backend
    
  bash build_AES_master_key.sh
  ```

* After the script finishes executing, the key will be printed to the console. You may use it as needed. It is recommended to store it as an environment variable and save it securely. 

  ```bash
  export SERVER_AES_MASTER_KEY_ENV=your_aes_key
  ```

* **Note**: The AES key must remain consistent. Changing the key midway will cause previously encrypted data to become undecryptable.

### 3. Start openJiuwen

* Open "Terminal" in the root directory of the source code.

* Copy the `.env` file and open it: 
  ```bash
  cp .env.example .env
  open .env
  ```

* In the *.env* file, modify the following variables according to your actual environment (do not overwrite other variables): 
   
   > **Note**: Values such as DB_HOST and DB_PORT can be replaced with your actual database information. DB_USER and DB_PASSWORD should be the MySQL username and password created earlier. If the password contains special characters, refer to [Special Character Escaping Table](#macos-special-char) to replace them with URL-encoded values. 
    
   ```env
   # Database configuration (Example)
   DB_HOST=localhost
   DB_PORT=3306
   DB_USER=your_user_name
   DB_PASSWORD=your_password

   # Milvus configuration (Example) 
   MILVUS_HOST=127.0.0.1
   MILVUS_PORT=19530
   MILVUS_COLLECTION_NAME=memory_vector

   # Memory-related configuration (If the memory feature is not used, the following parameters can be omitted) 
   EMBEDDING_MODEL_DIMENTION=1024
   EMBED_API_BASE=""
   EMBED_MODEL_NAME=""
   EMBED_API_KEY=""
   EMBED_TIMEOUT=5
   EMBED_MAX_RETRIES=1
   ```

  The descriptions of the variables are shown in the table below. If you need to enable the memory and knowledge base features, refer to [How to Enable the Memory and Knowledge Base Features](#macos-memory).

   | Variable Name                                   | Description                                                               | Example                                                                      |
   |---------------------------------------|--------------------------------------------------------------------|---------------------------------------------------------------------------|
   |**DB_HOST**                           | Database host address                                                           | `localhost`                                                               |
   | **DB_PORT**                           | Database port                                                            | `3306`                                                                    |
   | **DB_USER**                           | Database username                                                            | `your_user_name`                                                             |
   | **DB_PASSWORD**                       | Database password                                                             | `your_password`                                                         |
   | **MILVUS_HOST**                 | Host address of the Milvus service                                                | `127.0.0.1`                                                                    |
   | **MILVUS_PORT**                 | Port of the Milvus service                                                | `19530`                                                                    |
   | **MILVUS_COLLECTION_NAME**                | Databse name used by Milvus                                                | `memory_vector`                                                                    
   | **EMBEDDING_MODEL_DIMENTION**         | Dimension of the embedding model, determined by the model specified in EMBED_MODEL_NAME              | `1024`                                                                    |                  
   | **EMBED_API_BASE**                    | API endpoint of the embedding model                                                  | `https://example.com/embedding_model`            |            
   | **EMBED_MODEL_NAME**                  | Name of the embedding model                                                             | `text-embedding-model`                                                       |
   | **EMBED_API_KEY**                     | API key for the embedding model (replace with your own)                                                 | `sk-xxx`                                                                  |
   | **EMBED_TIMEOUT**                     | Maximum wait time for embedding requests                                                       | `5`                                                                     |
   | **EMBED_MAX_RETRIES**                 | Maximum number of retries when an embedding request fails.                                                | `1`                                                                    |

* Open a "Terminal". In the source code root directory, run the following commands to start the backend service. Please wait patiently: 
   
  ```bash
  cd backend
  uv venv
  uv sync
  ```

  > **Note**: If the process remains stuck for more than 20 minutes, press "Ctrl + C", try modifying the url value under [[tool.uv.index]] in the "pyproject.toml" file in this directory to switch to another available source. Then, rerun "uv sync". 

  > **Note**: If `uv sync` fails, you can try `uv sync --native-tls` to force the use of the system's native TLS library (to resolve HTTPS download compatibility issues). 

  ```bash
  mkdir logs
  mkdir logs/run
  source .venv/bin/activate
  python main.py
  ```
  
  > **Note**: Some users may encounter a "No module named 'greenlet'" error when running `python main.py`. Please refer to the [FAQ](#macos-greenlet) for a solution.

  After a successful startup, the message "Application startup complete" will be displayed. 

  > **Tip**: If you need to configure plugins, the sandbox service must be enabled. Refer to [How to Enable the Sandbox Feature](#macos-sandbox) to complete the sandbox service configuration. 


* Open another "Terminal", and in the root directory of the source code run the following command to install frontend dependencies:  

  ```bash
  cd frontend
  npm install
  ```
  > **Note**: The vulnerabilities shown in the screenshot below are known issues reported by npm and do not affect running the application. 

  ![image](../images/npm-error.png)

* Run the following command to start the frontend service: 

  ```
  npm run dev
  ```

* After a successful startup, the following information will be displayed:

  Local: *local access address*

  Network: *network access address*

### 4. Access the System

  * To access locally, Control + click the *local access address*, then click Open Link to view the openJiuwen interface in your local browser. Alternatively, copy the *local access address* into the browser's address bar and press Enter.
  
  * To access on another machine, copy the *network access address* into the browser's address bar and press Enter to view the openJiuwen interface.

## IV. Frequently Asked Questions (FAQ) 

### <a id="macos-memory"></a> Question 1: How to Enable the Memory and Knowledge Base Features

The effectiveness of the memory feature depends on the number of parameters of the LLM.

The memory and knowledge base features rely on Milvus. On macOS, it is recommended to install Milvus using Docker. The detailed steps are described below: 

#### 1. Install Docker Desktop

* Download: Visit the <a href="https://www.docker.com/products/docker-desktop/" rel="nofollow">Docker Desktop official website</a>, and click "Download for Mac" to download the .dmg installer.
* Double-click the installer and drag **Docker** into the Applications folder.

  ![docker1](../images/docker拖拽.png)

* Find and start the Docker application.
* Upon first opening Docker, the system will prompt you to enter your macOS password to authorize the installation of virtual machine components. Click OK to continue.
* The first startup will require waiting for Docker to complete initialization. (Downloading base images, which may take a few minutes)

* Docker Desktop installation complete.

> **Note**: If you encounter any errors during installation, please refer to the <a href="https://docs.docker.com/desktop/setup/install/mac-install/" rel="nofollow">official Docker Desktop installation guide</a>.

#### 2. Start Milvus

* Run the following command in "Terminal" to save the "standalone_embed.sh" script to the current directory:

  ```
  curl -fsSL https://raw.githubusercontent.com/milvus-io/milvus/master/scripts/standalone_embed.sh -o standalone_embed.sh
  ```

* In Terminal, run the following commands to pull the Milvus images:

  ```bash
  # x86 architecture
  docker pull swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-amd64:v2.6.2
  ```

  ```bash
  # arm architecture
  docker pull swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-arm64:v2.6.2
  ```

* Edut the "standalone_embed.sh" file and replace the official Milvus image name inside the script (e.g. `milvusdb/milvus:v2.6.7`) with the corresponding image name (e.g. for ARM machines: `swr.cn-north-4.myhuaweicloud.com/openjiuwen/milvusdb/milvus-arm64:v2.6.2`). 
  
* After making the change, run the following command in Terminal to start Milvus as a Docker container: 

  ```
  bash standalone_embed.sh start
  ```

* After startup, run `docker ps -a` to verify that a Docker container named Milvus-standalone is running on port `19530`. 

  > **Note**: If you encounter issues during deployment, refer to the <a href="https://milvus.io/docs/zh/install_standalone-docker.md" target="_blank" rel="nofollow noopener noreferrer">official Milvus documentation</a>.

* To stop Milvus, run the following command:

  ```
  bash standalone_embed.sh stop
  ```

#### 3. Obtain the Embedding Model

The memory and knowledge base features rely on an embedding model. The following steps uses Huawei Cloud as an example to illustrate how to obtain an embedding model.

* Click <a href="https://console.huaweicloud.com/modelarts/?locale=zh-cn&region=cn-southwest-2#/model-studio/square" target="_blank" rel="nofollow noopener noreferrer">this link</a> to enter the ModelArts Model Square.  

* For memory feature, click "向量模型" (Embedding model) and locate the BGE-M3 model. For knowledge base feature, select a model according to your scenario.

  ![Locate the embedding model](../images/find_embed.png)

* After locating the BGE-M3 model, click 推理调用 (Inference Call) to enter the model information acquisition page.

  ![Obtain api_base and model_name](../images/embed_api_base_and_model_name.png)

* Record the API address (corresponding to EMBED_API_BASE) and model parameters (corresponding to EMBED_MODEL_NAME).

* Click "API Key Management" and follow the instructions on the website to obtain an API Key (corresponding to EMBED_API_KEY).

> **Note**: Once the memory feature has been enabled after configuring *EMBEDDING_MODEL_DIMENTION*, do not modify this value again, otherwise the memory feature will stop working. It is also not recommended to change other embedding model configurations, as doing so may affect performance or results.

### <a id="macos-sandbox"></a> Question 2: How to Enable the Sandbox Feature

To configure plugins, the sandbox service must be enabled. Perform the following steps:

1. Refer to the `sandbox_server/python_server/.env.example` file and create a `.env` file in the `sandbox_server/python_server` directory. Example: 

   ```
   HOST=0.0.0.0
   PORT=5001
   ```

   Then, start the sandbox Python service by running the `sandbox_server/python_server/kernel.py` script. `HOST` and `PORT` specify the IP address and port on which the sandbox Python service runs. 

2. Start the sandbox JavaScript service by running the `sandbox_server/js_server/kernel.js` script. The IP address and port for the JS service are defined as follows: 

   ```javascript
   const PORT = process.env.PORT || 5002;
   server.listen(PORT, "0.0.0.0", () => {
     console.log(`✅ JS sandbox listening on http://0.0.0.0:${PORT}`);
   });
   ```

3. Refer to the `sandbox_server/gateway/.env.example` file and create a `.env` file in the `sandbox_server/gateway` directory. Example: 

   ```env
   HOST=0.0.0.0
   PORT=8188
   PYTHON_SANDBOX_URL=http://localhost:5001/run
   JS_SANDBOX_URL=http://localhost:5002/run
   ```

   `PYTHON_SANDBOX_URL` and `JS_SANDBOX_URL` are the URLs of the Python and JS services started in the previous steps. Then, start the sandbox gateway service by running the `sandbox_server/gateway/server.py` script. 

### <a id="macos-greenlet"></a> Question 3: How to Resolve "No Module named 'greenlet'" When Starting the Backend

On some Macs with Apple Silicon chips, Python may have compatibility issues, and the greenlet package may be missing from the standard environment. You can resolve this as follows:

  ```bash
  # First, exit the virtual environment (skip this step if you are not in one) 
  deactivate
  # Add greenlet to the virtual environment
  uv add greenlet
  # Re-enter the virtual environment to continue
  source .venv/bin/activate
  ```

### <a id="macos-special-char"></a> Question 4: Special Character Escaping Table

| Character   | URL Encoding | Character   | URL Encoding | Character   | URL Encoding | Character   | URL Encoding | Character   | URL Encoding |
|--------|---------|--------|---------|--------|---------|--------|---------|--------|---------|
| Space | %20    | "      | %22     | #      | %23     | %      | %25     | &   | %26     |
| (      | %28    | )      | %29     | +      | %2B     | ,      | %2C     | /      | %2F     |
| :      | %3A    | ;      | %3B     | <   | %3C     | =      | %3D     | >   | %3E     |
| ?      | %3F    | @      | %40     | \      | %5C     | \|     | %7C     | -      | -       |

### Question 5: Why does local installation default to HTTP instead of HTTPS?

In local installation mode, the system defaults to HTTP for communication. This design choice is primarily based on the fact that local environments are typically used for development and testing, and avoiding mandatory TLS certificate setup helps reduce the initial usage barrier.

By contrast, the Docker installation method comes with built-in HTTPS support, allowing users to use secure communication out of the box without additional configuration.

If HTTPS is required in a local environment, developers must manually generate and configure TLS certificates according to their deployment needs.
