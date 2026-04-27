# 🚀 Enterprise Middleware Ansible

**中间件一键部署平台** —— 基于 Ansible 的多维可配置部署方案，统一管理 Apache Kafka、RocketMQ、RabbitMQ、Redis、ZooKeeper、Tomcat、Nginx、Elasticsearch、MySQL 等九大核心中间件的生命周期。

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/ansible-9%2B-red?logo=ansible)](https://docs.ansible.com/)

---

## ✨ 核心特性

- **一张清单，多组件部署** —— 通过统一的 `hosts.ini` 定义所有中间件节点，集中管控。
- **七种安装方式按需切换** —— 二进制包 / 系统包管理器 / 源码编译 / Docker / Docker Compose / Kubernetes YAML / Helm Chart。
- **无缝切换架构模式** —— 单机、主备、高可用集群（Sentinel、MHA、KRaft、双写等）一键切换。
- **全发行版适配** —— 自动识别 RedHat/CentOS/Anolis/openEuler、Debian/Ubuntu 等，安装依赖与调优策略自适应。
- **金融级系统调优** —— 内核参数、文件句柄、内存管理、THP 透明大页等深度优化，保障中间件高性能稳定运行。
- **安全合规** —— 支持 Ansible Vault 加密密码、自动配置防火墙规则、最小权限运行用户。
- **统一入口脚本** —— `deploy.sh` 封装复杂的 ansible-playbook 命令，提供帮助、测试、试运行等辅助功能。
- **一键免密登录** —— 内置 SSH 公钥分发 playbook，为所有目标主机自动配置免密登录。

---

## 📁 项目目录结构

```markdown
enterprise-middleware-ansible/
├── ansible.cfg                          # Ansible 全局配置（并发、SSH 优化等）
├── deploy.sh                            # 统一部署入口脚本
├── .env                                 # 环境变量配置文件（可选）
├── inventory/                           # 主机清单目录
│   └── production/
│       ├── hosts.ini                    # 生产环境主机清单
│       ├── group_vars/                  # 组变量（中间件独立配置）
│       │   ├── all.yml                  # 全局变量
│       │   ├── redis.yml
│       │   ├── mysql.yml
│       │   ├── kafka.yml
│       │   ├── rocketmq.yml
│       │   ├── rabbitmq.yml
│       │   ├── zookeeper.yml
│       │   ├── nginx.yml
│       │   ├── tomcat.yml
│       │   └── elasticsearch.yml
│       └── host_vars/                   # 主机级变量（可选）
├── playbooks/                           # 部署与维护 Playbook
│   ├── setup_ssh.yml                    # 一键配置 SSH 免密登录
│   ├── system_tuning.yml                # 独立系统调优
│   ├── deploy_all.yml                   # 按依赖顺序一键部署全部中间件
│   ├── deploy_redis.yml
│   ├── deploy_mysql.yml
│   ├── deploy_kafka.yml
│   ├── deploy_rocketmq.yml
│   ├── deploy_rabbitmq.yml
│   ├── deploy_zookeeper.yml
│   ├── deploy_nginx.yml
│   ├── deploy_tomcat.yml
│   └── deploy_elasticsearch.yml
├── roles/                               # Ansible 角色（核心逻辑）
│   ├── common/                          # 公共角色（系统调优、依赖包）
│   ├── redis/
│   ├── mysql/
│   ├── kafka/
│   ├── rocketmq/
│   ├── rabbitmq/
│   ├── zookeeper/
│   ├── nginx/
│   ├── tomcat/
│   └── elasticsearch/
│       ├── tasks/                       # 主任务、各安装方式子任务
│       ├── templates/                   # Jinja2 配置模板
│       ├── handlers/                    # 服务重启等触发器
│       └── defaults/                    # 默认变量
├── files/                               # 离线安装包存放目录
└── docs/                                # 文档
```

## 🚀 快速开始

### 1. 环境准备

- **控制节点**：安装了 Ansible ≥ 9.0 的 Linux/macOS，并确保已安装所需集合：
  ```bash
  pip install ansible jmespath passlib
  ansible-galaxy collection install community.general community.docker kubernetes.core
  ```

- **目标主机**：与 Ansible 控制节点网络互通，已配置好 IP 和 SSH 密码（首次可使用 `deploy.sh ssh` 配置免密）。

### 2. 配置主机清单

编辑 `inventory/production/hosts.ini`，根据你的规划填写主机 IP 和分组：

```ini
[redis_master]
redis-master-01 ansible_host=10.0.1.10

[redis_slave]
redis-slave-01 ansible_host=10.0.1.11

[mysql_master]
mysql-master-01 ansible_host=10.0.2.10 mysql_server_id=1

# ... 以此类推
```

### 3. 设置全局变量

修改 `inventory/production/group_vars/all.yml`，指定默认安装方式和系统调优参数：

```yaml
install_method: "binary"          # 可改为 package / docker / k8s_helm 等
install_mode: "online"            # online / offline
base_install_dir: "/opt"
system_tuning:
  sysctl:
    vm.swappiness: 10
    # ...
```

### 4. 配置中间件特定参数

例如 `inventory/production/group_vars/redis.yml`：

```yaml
redis_version: "7.2.4"
redis_architecture: "sentinel"    # standalone | master_slave | sentinel | cluster
redis_password: "YourSecurePass"  # 建议使用 vault 加密
redis_maxmemory: "4gb"
```

### 5. 一键配置免密登录（可选）

```bash
./deploy.sh ssh
```

根据提示输入远程主机密码后，所有主机的公钥将自动分发。

### 6. 执行部署

```bash
# 使用统一入口（推荐）
./deploy.sh redis --arch sentinel --method binary

# 或直接调用 ansible-playbook（完全等价）
ansible-playbook -i inventory/production/hosts.ini playbooks/deploy_redis.yml \
  -e redis_architecture=sentinel -e redis_install_method=binary
```

---

## 📖 命令参考

### 统一入口 `deploy.sh`

```bash
./deploy.sh <中间件名> <命令> [选项]
```

**命令**

| 命令      | 说明                               |
| --------- | ---------------------------------- |
| `deploy`  | 执行部署（默认）                   |
| `test`    | 测试目标主机连通性（ansible ping） |
| `info`    | 显示当前配置信息                   |
| `dry-run` | 试运行（--check --diff）           |
| `ssh`     | 配置所有主机 SSH 免密登录          |

**中间件名称**：`redis`、`mysql`、`kafka`、`rocketmq`、`rabbitmq`、`zookeeper`、`nginx`、`tomcat`、`elasticsearch`、`all`

**常用选项**

- `--arch, -a`    架构模式：`standalone`、`master_slave`、`sentinel`、`cluster`
- `--method, -i`  安装方式：`binary`、`package`、`compile`、`docker`、`docker_compose`、`k8s_yaml`、`k8s_helm`
- `--mode, -o`    安装模式：`online` / `offline`
- `--version, -v` 指定中间件版本
- `--limit`       限制目标主机或组
- `--check, -C`   试运行，不实际变更
- `--diff, -D`    显示变更差异
- `--extra-vars, -e`  传递额外 Ansible 变量

**示例**

```bash
# 离线二进制部署 Redis Sentinel 集群
./deploy.sh redis --arch sentinel --method binary --mode offline

# Docker Compose 部署单机 Kafka
./deploy.sh kafka --method docker_compose -v 3.7.0

# 试运行 MySQL 主从部署
./deploy.sh mysql --arch master_slave --check --diff

# 查看 Nginx 配置信息
./deploy.sh nginx info
```

---

## 🧩 中间件覆盖矩阵

| 中间件        | 单机 | 主备 | 集群/高可用             | 二进制 | 包管理 | 编译 | Docker | Docker Compose | K8s YAML | Helm Chart |
| ------------- | ---- | ---- | ----------------------- | ------ | ------ | ---- | ------ | -------------- | -------- | ---------- |
| Redis         | ✅    | ✅    | ✅ Sentinel / Cluster    | ✅      | ✅      | ✅    | ✅      | ✅              | ✅        | ✅          |
| MySQL         | ✅    | ✅    | ✅ MHA / MGR             | ✅      | ✅      | ❌    | ✅      | ❌              | ❌        | ❌          |
| Kafka         | ✅    | ❌    | ✅ KRaft / ZooKeeper     | ✅      | ❌      | ❌    | ✅      | ✅              | ✅        | ❌          |
| RocketMQ      | ✅    | ✅    | ✅ 双主双从 / DLedger    | ✅      | ❌      | ❌    | ❌      | ✅              | ❌        | ❌          |
| RabbitMQ      | ✅    | ❌    | ✅ 多节点                | ❌      | ✅      | ❌    | ✅      | ❌              | ❌        | ❌          |
| ZooKeeper     | ✅    | ❌    | ✅ 3/5 节点集群          | ✅      | ❌      | ❌    | ✅      | ❌              | ❌        | ❌          |
| Nginx         | ✅    | ✅    | ✅ Keepalived + 负载均衡 | ❌      | ✅      | ✅    | ✅      | ❌              | ❌        | ❌          |
| Tomcat        | ✅    | ❌    | ✅ 多实例 + Nginx 反代   | ✅      | ❌      | ❌    | ✅      | ❌              | ❌        | ❌          |
| Elasticsearch | ✅    | ❌    | ✅ 3+ 节点集群           | ❌      | ✅      | ❌    | ✅      | ❌              | ✅        | ✅          |

---

## 🔧 配置要点

### 敏感信息加密

生产环境必须使用 Ansible Vault 保护密码：

```bash
# 创建加密变量文件
ansible-vault create inventory/production/group_vars/vault.yml
# 在 vault.yml 中定义 redis_password、mysql_root_password 等

# 部署时自动解密（建议使用密码文件）
echo "your_vault_password" > .vault_pass.txt
chmod 600 .vault_pass.txt
# ansible.cfg 中已配置 vault_password_file = .vault_pass.txt
```

### 离线部署

1. 在 `files/` 目录下提前存放所需软件包（按中间件和版本命名）。
2. 全局变量设置 `install_mode: "offline"`。
3. 各 Role 的离线安装任务会自动从 `files/` 读取对应包。

---

## ❓ 常见问题

**Q1：支持哪些 Linux 发行版？**  
A：RedHat 7/8/9、CentOS 7/8/9、Anolis OS、openEuler、Ubuntu 18/20/22、Debian 10/11 等主流发行版。

**Q2：执行过程中提示 "Failed to set permissions"？**  
A：确保目标主机的 `/etc/sudoers` 允许无终端执行，即去除 `Defaults requiretty`；或关闭 pipelining（`ansible.cfg` 中设置 `pipelining = False`）。

**Q3：如何快速测试只装一个 Redis 单机？**  
```bash
./deploy.sh redis --arch standalone --method package
```

**Q4：如何只对部分主机执行系统调优？**  
A：直接运行 `ansible-playbook -i inventory/production/hosts.ini playbooks/system_tuning.yml --limit redis_master`

**Q5：Windows 控制节点能否使用？**  
A：Ansible 官方不支持 Windows 作为控制节点，推荐使用 WSL2、Linux 虚拟机或 macOS。

---

## 📜 许可

本项目采用 Apache 2.0 许可证，详情见 `LICENSE` 文件。

---

## 🙋 贡献与支持

欢迎通过 Issue 或 Pull Request 提交改进建议。你可根据实际需求自由裁剪和扩展各中间件角色。

> **提示**：文中涉及的项目仓库地址为方案示例，实际使用时请按照你的私有仓库路径进行配置。

