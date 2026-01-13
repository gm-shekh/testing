# WhisperOcean Server

一个用于语音克隆和故事播放的后端服务，基于 index-tts 模型实现。

## 功能特性

- 🔐 用户注册和认证系统 (JWT)
- 🎵 声音样本上传和管理
- 🎭 语音克隆服务 (基于 index-tts)
- 🔊 生成的音频文件管理
- 📊 用户数据管理

## 技术栈

- **框架**: FastAPI
- **数据库**: PostgreSQL + SQLAlchemy
- **认证**: JWT + OAuth2
- **语音克隆**: index-tts
- **文件处理**: Python 标准库
- **部署**: Uvicorn

## 快速开始

### 1. 环境准备

```bash
# 克隆项目
git clone <your-repo-url>
cd WhisperOcean-Server

# 1. 创建后端服务环境
conda env create -f environment.yml
conda activate whisper-ocean

# 2. 创建 index-tts 环境（如果还没有）
conda create -n index-tts python=3.8
conda activate index-tts
pip install index-tts  # 或者按照 index-tts 的官方安装说明进行安装
```

### 2. 环境配置

复制环境变量配置文件：
```bash
cp .env.example .env
```

编辑 `.env` 文件，配置以下信息：

```bash
# 数据库配置
DATABASE_URL=postgresql://username:password@localhost:5432/whisper_ocean

# 安全配置
SECRET_KEY=your-very-secure-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# index-tts 配置
CONDA_ENV_NAME=index-tts
INDEX_TTS_MODEL_DIR=checkpoints
INDEX_TTS_CONFIG=checkpoints/config.yaml

# 服务器配置
HOST=0.0.0.0
PORT=8000
DEBUG=True
```

### 3. 数据库设置

确保 PostgreSQL 已安装并运行，然后创建数据库：

```sql
CREATE DATABASE whisper_ocean;
```

### 4. index-tts 环境

确保你已经安装了 index-tts 并且 conda 环境可用：

```bash
conda activate index-tts
# 验证 indextts 命令可用
indextts --help
```

### 5. 启动服务

```bash
# 开发模式
python -m app.main

# 或使用 uvicorn 直接启动
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

服务启动后，访问：
- API 文档: http://localhost:8000/docs
- 健康检查: http://localhost:8000/health

## API 端点

### 认证相关

- `POST /api/auth/register` - 用户注册
- `POST /api/auth/login` - 用户登录
- `GET /api/auth/me` - 获取当前用户信息
- `GET /api/auth/test` - 测试认证

### 语音相关

- `POST /api/voice/samples` - 上传声音样本
- `GET /api/voice/samples` - 获取用户声音样本列表
- `GET /api/voice/samples/{id}` - 获取特定声音样本
- `DELETE /api/voice/samples/{id}` - 删除声音样本
- `POST /api/voice/clone` - 克隆声音

## 使用示例

### 1. 用户注册

```bash
curl -X POST "http://localhost:8000/api/auth/register" \
     -H "Content-Type: application/json" \
     -d '{
       "email": "user@example.com",
       "username": "testuser",
       "password": "testpassword",
       "full_name": "Test User"
     }'
```

### 2. 用户登录

```bash
curl -X POST "http://localhost:8000/api/auth/login" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "username=testuser&password=testpassword"
```

### 3. 上传声音样本

```bash
curl -X POST "http://localhost:8000/api/voice/samples" \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -F "name=My Voice Sample" \
     -F "description=Test voice sample" \
     -F "audio_file=@/path/to/your/audio.wav"
```

### 4. 克隆声音

```bash
curl -X POST "http://localhost:8000/api/voice/clone" \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "text": "你好，这是一个语音克隆测试。",
       "voice_sample_id": 1
     }'
```

## 项目结构

```
WhisperOcean-Server/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI 应用入口
│   ├── config.py            # 配置管理
│   ├── database.py          # 数据库配置
│   ├── models/              # 数据库模型
│   ├── schemas/             # Pydantic 模型
│   ├── api/                 # API 路由
│   ├── services/            # 业务逻辑
│   └── utils/               # 工具函数
├── uploads/                 # 上传的声音文件
├── outputs/                 # 生成的音频文件
├── requirements.txt         # Python 依赖
├── .env.example            # 环境变量示例
└── README.md               # 项目文档
```

## 部署

### 生产环境配置

1. 设置生产环境变量：
   - 使用强密码和安全的 SECRET_KEY
   - 配置具体的 CORS 域名
   - 关闭 DEBUG 模式

2. 使用 Gunicorn 或类似的 WSGI 服务器：

```bash
pip install gunicorn
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker
```

### Docker 部署 (可选)

```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 注意事项

1. **安全性**: 
   - 在生产环境中修改默认的 SECRET_KEY
   - 配置适当的 CORS 策略
   - 使用 HTTPS

2. **性能**:
   - 语音克隆操作较为耗时，考虑使用任务队列 (如 Celery)
   - 大文件上传可能需要调整超时设置

3. **存储**:
   - 音频文件会占用大量存储空间
   - 考虑定期清理或使用云存储

## 许可证

[Your License Here]

## 贡献

欢迎提交 Issue 和 Pull Request！

## 联系方式

[Your Contact Information] 
