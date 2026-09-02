# 竞品分析 Agent v1.5.9 - Docker 部署包（镜像仓库）

本项目提供两种部署方式：

| 方式 | 仓库 | 适用场景 |
|------|------|----------|
| **方式一：镜像包直接部署**（本仓库） | `Competitor-Analysis-Agent` | 只想快速上线，不需要改代码。无需编译环境，加载镜像即可运行 |
| **方式二：源码构建部署** | [Competitor-Analysis-Agent-src](https://github.com/B0006CODE/Competitor-Analysis-Agent-src) | 需要二次开发、审计源码。需自行 `docker build`（3-10 分钟） |

两个仓库内容一致（同一版本发布），按需二选一即可。

## 包内容（本仓库）

```
Competitor-Analysis-Agent/
├── competitive-analysis-agent-v1.5.9.tar   # Docker 镜像包（docker save 导出，Git LFS 存储）
├── deploy.sh                               # 一键部署脚本
├── .env.example                            # 环境变量配置模板
└── README.md                               # 本说明文档
```

## 一、拉取本仓库

> **重要**：镜像 tar 包（约 176MB）通过 Git LFS 存储。克隆前必须先安装 git-lfs，
> 否则拉下来的 tar 只有 134 字节（LFS 指针文件），无法 docker load。

### 方式 A：git clone（推荐）

```bash
# 1. 安装 git-lfs（已安装可跳过）
apt-get install -y git-lfs        # Debian / Ubuntu
yum install -y git-lfs            # CentOS / RHEL

# 2. 初始化 LFS（每台机器只需执行一次）
git lfs install

# 3. 克隆仓库（自动下载 LFS 大文件）
git clone https://github.com/B0006CODE/Competitor-Analysis-Agent.git
cd Competitor-Analysis-Agent

# 4. 校验：tar 应约为 176MB；若只有 134 字节，执行 git lfs pull 补拉
ls -lh competitive-analysis-agent-v1.5.9.tar
git lfs pull
```

### 方式 B：不装 git，直接下载镜像包

浏览器或命令行任选其一（两者等价，均为 LFS 真实内容）：

```bash
# GitHub 页面：进入仓库 → 点击 competitive-analysis-agent-v1.5.9.tar → 右上「下载原始文件」
# 或命令行直链下载：
wget https://media.githubusercontent.com/media/B0006CODE/Competitor-Analysis-Agent/main/competitive-analysis-agent-v1.5.9.tar
```

> 无论哪种方式，请把 deploy.sh、.env.example 一并下载（README 页面 → 右上「下载原始文件」），
> 与 tar 放在同一目录。

## 二、方式一：镜像包部署（本仓库，推荐）

### 快速部署

```bash
# 1. 进入仓库目录（tar / deploy.sh / .env.example 需在同一目录）
cd Competitor-Analysis-Agent

# 2. 复制配置模板并填入真实密钥
cp .env.example .env
vi .env    # 必填: LLM_API_KEY, TAVILY_API_KEY（可选: ZHIHU_API_KEY, BOCHA_API_KEY）

# 3. 一键部署（自动探测版本化镜像包并加载、启动容器、健康检查）
bash deploy.sh
```

部署完成后访问：`http://<服务器IP>:8000`

### 手动部署

```bash
# 1. 加载镜像
docker load -i competitive-analysis-agent-v1.5.9.tar

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

## 三、方式二：源码构建部署

如果需要修改代码，请拉取源码仓库并构建镜像：

```bash
git lfs install                      # 源码仓库无 LFS 大文件，不装 git-lfs 也可
git clone https://github.com/B0006CODE/Competitor-Analysis-Agent-src.git
cd Competitor-Analysis-Agent-src

cp .env.example .env && vi .env      # 先配置密钥
docker build -t competitive-analysis-agent:latest .
bash deploy.sh                       # 本机已有镜像时自动跳过加载，直接启动
```

完整说明（本地运行、验证、常见问题）见源码仓库 README。

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
| `APP_VERSION` | 否 | 版本号，默认 v1.5.9 |
| `APP_ENV` | 否 | auto/test/staging/prod，默认 auto（按 APP_VERSION 自动推断） |

## 常用命令

```bash
docker logs -f competitive-analysis   # 查看日志
docker restart competitive-analysis   # 重启
docker exec -it competitive-analysis /bin/sh   # 进入容器
docker rm -f competitive-analysis     # 删除容器（数据卷 analysis-data 保留）
```

## 更新版本

1. `git pull`（或重新下载新版本的 tar 包替换旧文件）
2. 重新执行 `bash deploy.sh`（自动停止旧容器并启动新容器，数据保留在 volume 中）

## 版本信息

- 版本号：v1.5.9（2026-09-02）
- 镜像标签：`competitive-analysis-agent:latest`
- 主要功能：竞品分析 Agent（LLM + 多来源搜索：Tavily / 知乎 / 博查），Web 界面 + REST API
- 本版主要变更：修复报告弹窗四个来源标签被拉伸成竖条的问题（点开均为「标签胶囊行 + 目录/报告内部滚动」正常形态）
- 变更记录详见源码仓库 CHANGELOG.md
