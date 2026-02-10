git clone https://ghfast.top/https://github.com/openclaw/openclaw.git
cd openclaw
bash docker-setip.sh

node dist/index.js onboard








# 国内 docker 搭建 openclaw


方式一：
docker pull docker.1ms.run/library/node:lts-alpine3.23

docker run -it -d -p 18790:18790 -p 18789:18789 \
-v ~/docker/openclaw:/root/.openclaw \
--name openclaw \
docker.1ms.run/library/node:lts-alpine3.23




docker exec -it openclaw sh 
# 将原有的仓库地址替换为阿里云镜像
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 再次尝试安装
apk add --no-cache git

# 1. 安装基础构建工具（Alpine 专用）
apk add --no-cache git python3 make g++ gcc cmake

# 1. 安装 Alpine 缺失的内核头文件和构建全家桶
apk add --no-cache linux-headers git python3 make g++ gcc cmake

npm install -g npm@11.8.0  --registry=https://registry.npmmirror.com
# 2. 尝试再次安装，并增加禁用构建的参数（如果 openclaw 支持）
# 或者强制指定镜像源防止 npx 环节超时
npm i -g openclaw --registry=https://registry.npmmirror.com

openclaw onboard
openclaw gateway --bind lan --port 18789 --force



openclaw gateway --bind lan --port 18789 --pair --force




ws://0.0.0.0:18789
EuQaqGLIYhRRSacSyvMlRhGqECbmTzQIeUbSvEYPHar


cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e

  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "xxxx"
    },
    "port": 18789,
	
    "bind": "0.0.0.0",
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  },
  
  
  
    "gateway": {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e",
    },
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e"
    },
    "port": 18789,
    "bind": "any",
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  },
  

◇  Optional apps ────────────────────────╮
│                                        │
│  Add nodes for extra features:         │
│  - macOS app (system + notifications)  │
│  - iOS app (camera/canvas)             │
│  - Android app (camera/canvas)         │
│                                        │
├────────────────────────────────────────╯
│
◇  Control UI ───────────────────────────────────────────────────────────────────────────────╮
│                                                                                            │
│  Web UI: http://127.0.0.1:18789/                                                           │
│  Web UI (with token):                                                                      │
│  http://127.0.0.1:18789/?token=cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e            │
│  Gateway WS: ws://127.0.0.1:18789                                                          │
│  Gateway: not detected (gateway closed (1006 abnormal closure (no close frame)): no close  │
│  reason)                                                                                   │
│  Docs: https://docs.openclaw.ai/web/control-ui                                             │
│                                                                                            │
├────────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  Workspace backup ────────────────────────────────────────╮
│                                                           │
│  Back up your agent workspace.                            │
│  Docs: https://docs.openclaw.ai/concepts/agent-workspace  │
│                                                           │
├───────────────────────────────────────────────────────────╯
│
◇  Security ──────────────────────────────────────────────────────╮
│                                                                 │
│  Running agents on your computer is risky — harden your setup:  │
│  https://docs.openclaw.ai/security                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────╯
│
◇  Dashboard ready ────────────────────────────────────────────────────────────────╮
│                                                                                  │
│  Dashboard link (with token):                                                    │
│  http://127.0.0.1:18789/?token=cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e  │
│  Copy/paste this URL in a browser on this machine to control OpenClaw.           │
│  No GUI detected. Open from your computer:                                       │
│  ssh -N -L 18789:127.0.0.1:18789 user@<host>                                     │
│  Then open:                                                                      │
│  http://localhost:18789/                                                         │
│  http://localhost:18789/?token=cf8b050792083c0c8c5da55408a8eb8f73089c26d78dc15e  │
│  Docs:                                                                           │
│  https://docs.openclaw.ai/gateway/remote                                         │
│  https://docs.openclaw.ai/web/control-ui                                         │
│                                                                                  │
├──────────────────────────────────────────────────────────────────────────────────╯
│
◇  Web search (optional) ─────────────────────────────────────────────────────────────────╮
│                                                                                         │
│  If you want your agent to be able to search the web, you’ll need an API key.           │
│                                                                                         │
│  OpenClaw uses Brave Search for the `web_search` tool. Without a Brave Search API key,  │
│  web search won’t work.                                                                 │
│                                                                                         │
│  Set it up interactively:                                                               │
│  - Run: openclaw configure --section web                                                │
│  - Enable web_search and paste your Brave Search API key                                │
│                                                                                         │
│  Alternative: set BRAVE_API_KEY in the Gateway environment (no config changes).         │
│  Docs: https://docs.openclaw.ai/tools/web                                               │
│                                                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  What now ─────────────────────────────────────────────────────────────╮
│                                                                        │
│  What now: https://openclaw.ai/showcase ("What People Are Building").  │
│                                                                        │
├────────────────────────────────────────────────────────────────────────╯
│
└  Onboarding complete. Use the tokenized dashboard link above to control OpenClaw.