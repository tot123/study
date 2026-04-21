这是一份为您整理的完整、规范的 **PostgreSQL 15 扩展安装与全能数据库（All-in-One）配置教程**。本教程已针对 Conda 环境干扰、源码编译报错及 GitHub 访问加速进行了优化。

---

# PostgreSQL 15 全能数据库扩展安装手册

本手册涵盖了从基础向量检索、地理信息、图数据库到时序处理及 NoSQL 模拟的全套配置。

## 0. 环境全局配置

在开始前，请务必确保退出 Conda 环境，以防止编译时链接到错误的库文件。

```bash
# 彻底退出 Conda 环境
conda deactivate
# 更新系统包列表
sudo apt update

```

---

## 1. 核心依赖与 PostGIS 安装

PostGIS 是地理信息系统的核心，建议优先通过官方仓库安装。

```bash
# 安装 PostgreSQL 15 开发环境及 PostGIS
sudo apt install -y postgresql-contrib-15 postgresql-server-dev-15 postgresql-15-postgis-3 \
build-essential git flex bison libreadline-dev zlib1g-dev

```

---

## 2. 向量检索：pgvector 源码编译

使用 `gh-proxy.com` 加速 GitHub 源码下载。

```bash
cd /tmp
rm -rf pgvector
git clone --branch v0.5.1 https://gh-proxy.com/https://github.com/pgvector/pgvector.git
cd pgvector

# 强制使用系统 GCC 编译以避开环境干扰
make clean
make CC=gcc
sudo make install CC=gcc

```

---

## 3. 图数据库：Apache AGE 修复安装

针对 PostgreSQL 15 存在的头文件缺失（StringInfo 错误）进行手动修复。

```bash
cd /tmp
rm -rf age
git clone https://gh-proxy.com/https://github.com/apache/age.git
cd age

# 切换至 PG15 分支
git checkout PG15

# 修复头文件引用
sed -i '1i #include "lib/stringinfo.h"' src/include/utils/agtype.h

# 编译并安装
make clean
make CC=gcc
sudo make install CC=gcc

```

---

## 4. 时序与任务调度：TimescaleDB & pg_cron

```bash
# 安装 TimescaleDB (时序) 与 pg_cron (定时任务)
sudo apt install -y timescaledb-2-postgresql-15 timescaledb-tools postgresql-15-cron

# 自动配置性能参数（一路按 y）
sudo timescaledb-tune --quiet --yes

```

---

## 5. 中文分词：zhparser (含 SCWS)

```bash
# 1. 安装 SCWS 引擎
cd /tmp
wget http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2
tar xvjf scws-1.2.3.tar.bz2
cd scws-1.2.3
./configure && make && sudo make install

# 2. 编译安装 zhparser
cd /tmp
git clone https://gh-proxy.com/https://github.com/amytab/zhparser.git
cd zhparser
make CC=gcc
sudo make install CC=gcc
sudo ldconfig

```

---

## 6. 关键系统配置：预加载库

必须修改配置文件，使 TimescaleDB、pg_cron 等扩展生效。

```bash
# 编辑配置文件
sudo nano /etc/postgresql/15/main/postgresql.conf

```

找到 `shared_preload_libraries` 修改为：
`shared_preload_libraries = 'timescaledb, pg_cron, pg_stat_statements, age'`

```bash
# 重启服务使配置生效
sudo systemctl restart postgresql

```

---

## 7. 数据库初始化逻辑 (SQL)

进入 PostgreSQL 终端：

```bash
sudo -u postgres psql

```

### 7.1 激活基础与内置扩展

```sql
-- 在当前数据库激活
CREATE EXTENSION IF NOT EXISTS vector;          -- 向量检索
CREATE EXTENSION IF NOT EXISTS postgis;         -- 地理信息
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";    -- UUID 生成
CREATE EXTENSION IF NOT EXISTS pgcrypto;       -- 加密算法
CREATE EXTENSION IF NOT EXISTS pg_trgm;        -- 模糊匹配
CREATE EXTENSION IF NOT EXISTS zhparser;       -- 中文分词
CREATE EXTENSION IF NOT EXISTS age;            -- 图数据库 (需单独 LOAD 'age')

-- 配置中文分词器
CREATE TEXT SEARCH CONFIGURATION chinese (PARSER = zhparser);
ALTER TEXT SEARCH CONFIGURATION chinese ADD MAPPING FOR n,v,a,i,e,l WITH simple;

```

### 7.2 模拟 Redis (极简 Key-Value)

```sql
CREATE UNLOGGED TABLE cache_store (
    key text PRIMARY KEY,
    value jsonb,
    expired_at timestamp
);
CREATE INDEX idx_cache_expiry ON cache_store (expired_at);

```

### 7.3 模拟 MongoDB (Schema-less)

```sql
CREATE TABLE doc_store (
    id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
    data jsonb
);
CREATE INDEX idx_doc_gin ON doc_store USING GIN (data);

```

---

## 8. 自动化维护计划 (pg_cron)

在 `postgres` 库中设定：

```sql
-- 每分钟清理一次 Redis 模拟表的过期数据
SELECT cron.schedule('*/1 * * * *', $$DELETE FROM cache_store WHERE expired_at < now()$$);

```

---

## 9. 常用管理命令速查

| 目标 | SQL 命令 |
| --- | --- |
| **查看已安装扩展** | `\dx` |
| **查看系统可用扩展** | `SELECT * FROM pg_available_extensions;` |
| **查看图数据库中的图** | `SELECT * FROM ag_graph;` |
| **重载配置文件** | `SELECT pg_reload_conf();` |

---

**下一步建议：**
您已经打造了一个强大的“全栈”数据库。需要我为您提供一份基于 **Python (psycopg2)** 的统一访问类代码，直接在代码中操作这几个模拟的 Redis/Mongo 库吗？
