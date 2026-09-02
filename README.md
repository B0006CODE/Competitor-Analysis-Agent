# 竞品分析 Agent v1.5.8 - Docker 部署包

## 包内容

```
competitive-analysis-agent-v1.5.8/
├── competitive-analysis-agent-v1.5.8.tar   # Docker 镜像包（docker save 导出）
├── deploy.sh                               # 一键部署脚本
├── .env.example                            # 环境变量配置模板
└── README.md                               # 本说明文档
```

## 系统要求

- Linux 服务器（x86_64 / ARM64 均可）
- Docker 已安装并运行（未安装：`curl -fsSL https://get.docker.com | sh`）
- 服务器可访问外网（调用 LLM API、搜索 API）
- 建议 2 核 4G 以上

## 快速部署（推荐）

```bash
# 1. 上传整个文件夹到服务器，进入目录
cd competitive-analysis-agent-v1.5.8

# 2. 复制配置模板并填入真实密钥
cp .env.example .env
vi .env    # 必填: LLM_API_KEY, TAVILY_API_KEY（可选: ZHIHU_API_KEY, BOCHA_API_KEY）

# 3. 一键部署（自动加载镜像、启动容器、健康检查）
bash deploy.sh
```

部署完成后访问：`http://<服务器IP>:8000`

## 手动部署

```bash
# 1. 加载镜像
docker load -i competitive-analysis-agent-v1.5.8.tar

# 2. 创建 .env 配置文件（参照 .env.example，必填 LLM_API_KEY、TAVILY_API_KEY）

# 3. 启动容器
docker run -d \
    --name competitive-analysis \
    --restart unless-stopped \
    -p 8000:8000 \
    --env-file .env \
    -v analysis-data:/data \
    competitive-analysis-agent:latest
```

## 环境变量说明

| 变量 | 必填 | 说明 |
|------|------|------|
| `LLM_API_KEY` | 是 | LLM API Key（OpenAI 兼容，默认 deepseek） |
| `LLM_BASE_URL` | 否 | LLM API 地址，默认 `https://api.deepseek.com` |
| `LLM_MODEL` | 否 | 模型名，默认 `deepseek-v4-flash` |
| `TAVILY_API_KEY` | 是 | Tavily 搜索 Key（https://app.tavily.com 免费注册，1000 次/月） |
| `ZHIHU_API_KEY` | 否 | 知乎 Access Secret（https://developer.zhihu.com/profile） |
| `BOCHA_API_KEY` | 否 | 博查搜索 Key（https://bocha-ai.feishu.cn/wiki/RXEOw02rFiwzGSkd9mUcqoeAnNK） |
| `SEARCH_PROVIDER` | 否 | 服务端搜索回退策略：auto/tavily/bocha/zhihu/zhihu_site/zhida，默认 auto |
| `FRONTEND_ORIGIN` | 否 | 前端访问地址，deploy.sh 会自动补齐，如 `http://<IP>:8000` |
| `APP_VERSION` | 否 | 版本号，默认 v1.5.8 |
| `APP_ENV` | 否 | auto/test/staging/prod，默认 auto（按 APP_VERSION 自动推断） |

## 常用命令

```bash
docker logs -f competitive-analysis   # 查看日志
docker restart competitive-analysis   # 重启
docker exec -it competitive-analysis /bin/sh   # 进入容器
docker rm -f competitive-analysis     # 删除容器（数据卷 analysis-data 保留）
```

## 更新版本

1. 用新镜像包替换 `competitive-analysis-agent-v1.5.8.tar`
2. 重新执行 `bash deploy.sh`（自动停止旧容器并启动新容器，数据保留在 volume 中）

## 版本信息

- 版本号：v1.5.8（2026-09-02）
- 镜像标签：`competitive-analysis-agent:latest`
- 主要功能：竞品分析 Agent（LLM + 多来源搜索：Tavily / 知乎 / 博查），Web 界面 + REST API
- 变更记录详见项目 CHANGELOG.md