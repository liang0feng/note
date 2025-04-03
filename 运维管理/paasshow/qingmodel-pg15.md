已经下载了 PostgreSQL 15 的 RPM 安装包，并且这些文件位于 `/var/pginstall` 目录下，那么可以通过以下步骤在 CentOS 上离线安装 PostgreSQL 15。以下是详细的安装过程：

### 1. 安装 RPM 包
进入 `/var/pginstall` 目录，使用 `rpm` 命令安装 PostgreSQL 的 RPM 包。安装顺序很重要，通常先安装 `libs` 包，然后是主程序包和服务器包。

```bash
cd /var/pginstall
sudo rpm -ivh postgresql15-libs-15.8-1PGDG.rhel7.x86_64.rpm
sudo rpm -ivh postgresql15-15.8-1PGDG.rhel7.x86_64.rpm
sudo rpm -ivh postgresql15-server-15.8-1PGDG.rhel7.x86_64.rpm
```

如果系统提示缺少依赖项，可以尝试使用 `--nodeps` 参数强制安装，但不推荐，因为这可能导致运行时问题。如果确实缺少依赖项，需要手动下载并安装这些依赖项。

### 2. 初始化数据库
安装完成后，需要初始化 PostgreSQL 数据库。运行以下命令：

```bash
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
```

如果系统提示找不到命令，可能是因为环境变量未配置。可以手动指定路径运行初始化脚本。

### 3. 启动 PostgreSQL 服务
初始化完成后，启动 PostgreSQL 服务并设置开机自启：

```bash
sudo systemctl start postgresql-15
sudo systemctl enable postgresql-15
```

### 4. 配置 PostgreSQL
#### (1) **修改配置文件**
如果需要允许远程访问，需要修改 PostgreSQL 的配置文件。

- **编辑 `postgresql.conf` 文件**：
  ```bash
  sudo vim /var/lib/pgsql/15/data/postgresql.conf
  ```
  找到 `listen_addresses` 参数，将其值改为 `'*'`，允许从任何 IP 地址连接：
  ```plaintext
  listen_addresses = '*'
  ```

- **编辑 `pg_hba.conf` 文件**：
  ```bash
  sudo vim /var/lib/pgsql/15/data/pg_hba.conf
  ```
  添加以下内容以允许远程主机通过密码认证访问数据库：
  ```plaintext
  host    all             all             0.0.0.0/0               md5
  ```

#### (2) **重启服务**
修改配置文件后，需要重启 PostgreSQL 服务以使更改生效：
```bash
sudo systemctl restart postgresql-15
```

### 5. 配置环境变量（可选）
为了方便使用 PostgreSQL 命令，可以将 PostgreSQL 的二进制目录添加到环境变量中。

- **编辑 `/etc/profile` 文件**：
  ```bash
  sudo vim /etc/profile
  ```
  在文件末尾添加以下内容：
  ```bash
  export PATH=$PATH:/usr/pgsql-15/bin
  ```

- **使环境变量生效**：
  ```bash
  source /etc/profile
  ```

### 6. 测试连接
使用 `psql` 命令测试是否可以成功连接到 PostgreSQL 数据库：
```bash
psql -U postgres
```
如果能够成功进入 PostgreSQL 的命令行界面，说明安装和配置成功。

### 7. 创建用户和数据库（可选）
如果你需要创建新的用户和数据库，可以使用以下命令：

- **创建用户**：
  ```bash
  sudo -u postgres createuser --interactive
  ```

- **创建数据库**：
  ```bash
  sudo -u postgres createdb mydatabase
  ```