# OpenJiuWen Studio 0.1.1 to 0.1.2 Upgrade Guide

## Upgrade Note

⚠️ **Important Notice:** The current version upgrade does not support cross-version data compatibility in the database. Please back up all important data before upgrading.

Please carefully read this guide before upgrading and follow the steps to complete the upgrade operation.

## I. Local Installation Upgrade

### 1. Switch to the New Version Branch

1. Enter the project root directory
2. Pull the latest code:

   ```bash
   git pull
   ```

3. Switch to the 0.1.2 version branch:

   ```bash
   git checkout v0.1.2
   ```

### 2. Frontend Upgrade

1. Enter the frontend project directory:

   ```bash
   cd frontend
   ```

2. Re-execute the dependency installation command:

   ```bash
   npm install
   ```

3. Start the development server to apply updates:

   ```bash
   npm run dev
   ```

### 3. Backend Upgrade

1. Enter the backend project directory:

   ```bash
   cd backend
   ```

2. Re-execute the dependency synchronization command:

   ```bash
   uv sync
   ```

3. Activate the virtual environment and start the backend service:

   ```bash
   source .venv/bin/activate
   python main.py
   ```

## II. Docker Installation Upgrade

### 1. Redeploy Containers

#### 1.1 Download the 0.1.2 Version Package

Choose the corresponding version package according to your system architecture:

- x86_64 architecture download link:

  ```
  https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_0.1.2_amd64.zip
  ```

- arm64 architecture download link:

  ```
  https://openjiuwen-ci.obs.cn-north-4.myhuaweicloud.com/agentstudio/deployTool_0.1.2_arm64.zip
  ```

#### 1.2 Start OpenJiuWen Service

1. Extract the downloaded version package to the installation directory
2. Enter the extracted directory
3. Execute the following command to start the service:

   ```bash
   ./service.sh up
   ```

To stop the service, you can use the following command:

```bash
./service.sh down
```

> **Note:** For detailed installation steps, please refer to the corresponding system installation documentation in the "Install via Docker" directory.

## III. Verify Upgrade

After completing the above steps, please verify the upgrade is successful through the following methods:

1. Frontend page can be accessed normally
2. Backend service can respond to API requests normally

If you encounter any issues during the upgrade process, please check the project logs, FAQ in the installation guide, or contact project maintainers.
