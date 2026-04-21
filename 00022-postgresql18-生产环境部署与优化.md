# PostgreSQL 18.3 生产环境部署与优化指南

> 基于 Debian 12，涵盖基础安装、扩展组件、内核调优到维护检查的全流程

## 第一部分：基础安装

```bash
# 1. 导入官方仓库密钥
sudo apt-get update
sudo apt-get install -y curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# 2. 添加官方源
echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# 3. 安装核心组件
sudo apt-get update
sudo apt-get install -y postgresql-18 postgresql-client-18 postgresql-contrib-18
```

## 第二部分：扩展组件安装（全家桶）

安装 AI 向量检索、图数据库、地理信息、定时任务、性能审计等核心插件。

```bash
# 1. 一键安装全家桶插件
sudo apt-get install -y \
  postgresql-18-postgis-3 \
  postgresql-18-pgvector \
  postgresql-18-age \
  postgresql-18-cron \
  postgresql-18-partman \
  postgresql-18-hypopg \
  postgresql-18-pgaudit \
  postgresql-18-pg-qualstats

# 2. 配置内核级预加载（age 必须放首位，partman 使用 _bgw 后缀）
sudo sed -i "/shared_preload_libraries =/c\shared_preload_libraries = 'age, pg_stat_statements, pg_cron, pgaudit, pg_partman_bgw, pg_qualstats'" /etc/postgresql/18/main/postgresql.conf

# 3. 补充 pg_cron 必须的配置
if ! grep -q "cron.database_name" /etc/postgresql/18/main/postgresql.conf; then
    echo "cron.database_name = 'postgres'" | sudo tee -a /etc/postgresql/18/main/postgresql.conf
fi

# 4. 重启服务使内核插件生效
sudo systemctl restart postgresql
```

## 第三部分：数据库内激活扩展

```bash
sudo -u postgres psql -d postgres -c "
CREATE EXTENSION IF NOT EXISTS age;
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS pg_partman;
CREATE EXTENSION IF NOT EXISTS hypopg;
CREATE EXTENSION IF NOT EXISTS pgaudit;
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
"

# 永久设置搜索路径以支持图数据库 cypher 函数
sudo -u postgres psql -d postgres -c "ALTER USER postgres SET search_path = ag_catalog, \"\$user\", public;"
```

## 第四部分：性能优化配置

PostgreSQL 默认配置非常保守，需根据服务器硬件调整。关键参数说明：

| 参数 | 说明 | 建议值 |
|------|------|--------|
| `shared_buffers` | 数据库内存缓冲区 | 总内存的 25% |
| `work_mem` | 单个查询操作可用内存 | (RAM / max_connections / 16) MB |
| `maintenance_work_mem` | 维护操作可用内存 | 总内存的 1/16（上限 2GB） |
| `effective_cache_size` | 操作系统缓存估计 | 总内存的 75% |
| `checkpoint_completion_target` | 检查点完成目标 | 0.9 |
| `random_page_cost` | 随机页面读取成本 | SSD: 1.1 / HDD: 4.0 |

以下以 **64GB 内存、16 核 CPU、SSD 存储** 为例：

```bash
sudo sed -i "s/#shared_buffers = .*/shared_buffers = 16GB/" /etc/postgresql/18/main/postgresql.conf
sudo sed -i "s/#work_mem = .*/work_mem = 256MB/" /etc/postgresql/18/main/postgresql.conf
sudo sed -i "s/#maintenance_work_mem = .*/maintenance_work_mem = 2GB/" /etc/postgresql/18/main/postgresql.conf
sudo sed -i "s/#effective_cache_size = .*/effective_cache_size = 48GB/" /etc/postgresql/18/main/postgresql.conf
sudo sed -i "s/#checkpoint_completion_target = .*/checkpoint_completion_target = 0.9/" /etc/postgresql/18/main/postgresql.conf
sudo sed -i "s/#random_page_cost = .*/random_page_cost = 1.1/" /etc/postgresql/18/main/postgresql.conf

# 重启生效
sudo systemctl restart postgresql
```

> **注意**：请根据实际硬件调整上述数值。应用后务必验证服务能正常启动。

## 第五部分：维护与健康检查

### 检查慢查询

利用 `pg_stat_statements` 查看耗时最长的 5 条 SQL：

```bash
sudo -u postgres psql -c "SELECT query, calls, total_exec_time / 1000 as total_sec, mean_exec_time as avg_ms FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 5;"
```

### 自动化维护建议

- **定时备份**：配合 `pg_back` 或 `pg_dump` 脚本通过 `pg_cron` 定时执行
- **碎片清理**：安装 `pg_repack`（可选），定期对大表进行无锁清理
- **安全审计**：查看 `pgaudit` 产生的日志（`/var/log/postgresql/`），确保无越权访问
