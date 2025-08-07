# LobeChat 部署使用说明

## 🌟 概述

本部署方案包含以下服务：
- **LobeChat**: AI 聊天应用
- **PostgreSQL**: 向量数据库（使用 pgvector 扩展）
- **MinIO**: 对象存储服务
- **Casdoor**: 认证服务

支持两种部署模式：
- **本地IP模式**: 使用 `http://IP:端口` 访问
- **域名模式**: 使用自定义域名访问（需配置域名解析）

## 📍 服务访问地址

### 本地IP模式（默认）
| 服务 | 访问地址 | 说明 |
|------|----------|------|
| LobeChat | `http://<LOCAL_IP>:<LOBE_PORT>` | 例如：http://10.10.10.10:4000 |
| MinIO 控制台 | `http://<LOCAL_IP>:<MINIO_CONSOLE_PORT>` | 例如：http://10.10.10.10:9001 |
| Casdoor 控制台 | `http://<LOCAL_IP>:<CASDOOR_PORT>` | 例如：http://10.10.10.10:4002 |

### 域名模式
| 服务 | 访问地址 | 说明 |
|------|----------|------|
| LobeChat | `https://<DOMAIN>` | 例如：https://lobe.your-domain.com |
| MinIO 控制台 | `https://<MINIO_CONSOLE_DOMAIN>` | 例如：https://s3-console.your-domain.com |
| Casdoor 控制台 | `https://<CASDOOR_DOMAIN>` | 例如：https://casdoor.your-domain.com |

## 🚀 初始配置步骤

### 1. 获取 MinIO 访问密钥

1. 访问 MinIO 控制台（地址见上表）
2. 使用以下凭据登录：
   - **用户名**: `minio`（或您在 `.env` 中设置的 `MINIO_ROOT_USER`）
   - **密码**: `minio_pwd`（或您在 `.env` 中设置的 `MINIO_ROOT_PASSWORD`）
3. 创建存储桶：
![alt text](image/Create_Bucket.png)
   - 点击 "Buckets" → "Create Bucket"
   - 创建存储桶名称：`lobe`（或您在 `.env` 中设置的 `MINIO_LOBE_BUCKET`）
4. 获取访问密钥：
   - 点击 "Access Keys" → "Create Access Key"
   - 记录 **Access Key** 和 **Secret Key**
5. 将密钥填入 `.env` 文件：
   ```env
   S3_ACCESS_KEY_ID=<Access Key>
   S3_SECRET_ACCESS_KEY=<Secret Key>
   ```

### 2. 配置 Casdoor 应用

1. 访问 Casdoor 控制台（地址见上表）
2. 使用默认凭据登录：
   - **用户名**: `admin`
   - **密码**: `123456`（首次登录后请立即修改）
3. 创建新应用：
   - 点击顶部菜单 "身份认证" → "添加"
   - 填写应用信息：
     - **名称**: `LobeChat`（可自定义）
     - **显示名称**: `LobeChat`（可自定义）
     - **Logo**: `https://lobehub.com/icon-192x192.png`
     - **重定向URLs**: 
        - 本地开发环境：http://localhost:4000/api/auth/callback/casdoor
        - 局域网 IP 部署：http://LOBECHAT_IP:4000/api/auth/callback/casdoor
        - 公网环境：https://lobe.example.com/api/auth/callback/casdoor
     - **强制重定向origin**:
        - 本地开发环境：http://localhost:4000
        - 局域网 IP 部署：http://LOBECHAT_IP:4000
        - 公网环境：https://lobe.example.com
     - **启用注册**: false
     - 点击 "Save"
4. 获取应用密钥(在创建新应用界面)：
   - 记录 **客户端ID** 和 **客户端密钥**
5. 将密钥填入 `.env` 文件：
   ```env
   AUTH_CASDOOR_ID=<Client ID>
   AUTH_CASDOOR_SECRET=<Client Secret>
   ```

### 3. 重新构建 LobeChat 服务

完成上述配置后，清除当前 Lobe-db compose 重新构建：
```docker compose down && docker compose up -d --build```

### 拓展配置
为了完善你的 LobeChat 服务，你可以根据你的需求进行以下拓展配置。

使用 MinIO 存储 Casdoor 头像
允许用户在 Casdoor 中更换头像

你需要首先在 buckets 中创建一个名为 casdoor 的桶，选择自定义策略，复制并粘贴如下内容（如果你修改了桶名，请自行查找替换）
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": ["*"]
      },
      "Action": ["s3:GetBucketLocation"],
      "Resource": ["arn:aws:s3:::casdoor"]
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": ["*"]
      },
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::casdoor"],
      "Condition": {
        "StringEquals": {
          "s3:prefix": ["files/*"]
        }
      }
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": ["*"]
      },
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:GetObject"],
      "Resource": ["arn:aws:s3:::casdoor/**"]
    }
  ],
  "Version": "2012-10-17"
}