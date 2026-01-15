# Mac‑OS‑M 系列：使用 Colima 代替 Docker Desktop

> 在 Apple M 系列芯片上使用 Colima 管理 Docker（Colima 是一个轻量级的 Linux 虚拟机，内置 Docker Engine，可作为 Docker Desktop 的开源替代方案）。

本 README 目标：
- 帮助判断是否可以用 Colima 代替 Docker Desktop
- 指导在 macOS（Apple Silicon）上安装、配置和使用 Colima + Docker
- 提供卸载 Docker Desktop 的安全清理命令（可选）

---

## 适用场景
- 你使用的是 Apple M 系列（ARM64/aarch64）Mac
- 想要一个轻量、开源且低占用的 Docker 运行时（只包含虚拟机层与 Docker Engine）
- 不需要 Docker Desktop 的 UI 功能（如 Dashboard、Kubernetes 集成、商业许可等）

---

## 前置条件
- macOS 最新系统更新（建议）
- Homebrew 已安装
  安装 Homebrew 官方源（若已安装可跳过）：
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
- 推荐先更新 Homebrew：
  ```bash
  brew update
  ```

---

## 判断规则（是否可以替代 Docker Desktop）
1. 在 Homebrew 中查找 Docker 相关工具：
   ```bash
   brew search docker
   ```
2. 如果能通过 Homebrew 安装 `docker`、`docker-compose`（或 `docker-compose` v2）等客户端工具，通常可以卸载 Docker Desktop，改用 Colima。
3. 如果找不到你需要的工具或依赖（例如某些私有集成仅 Docker Desktop 提供），建议保留 Docker Desktop。

---

## （可选）彻底清理 Docker Desktop 残留
在确认要替代 Docker Desktop 后，先确认你不再需要 Docker Desktop 的 UI/集成交付功能，再执行下面命令（这些命令会删除程序和用户数据，请谨慎运行并做好备份）：

```bash
# 删除主程序（需管理员权限）
sudo rm -rf /Applications/Docker.app

# 删除用户组容器数据
rm -rf ~/Library/Group\ Containers/group.com.docker

# 删除用户配置目录
rm -rf ~/.docker

# 删除系统级 Docker 命令软链（视安装位置而定，请确认后执行）
sudo rm -rf /usr/local/bin/docker*
sudo rm -rf /usr/local/bin/compose*
```

注意：有些 Docker Desktop 可能将二进制安装到不同路径，请根据实际情况确认。

---

## 安装 Colima 与 Docker CLI
建议使用 Homebrew 安装 Colima 与 Docker CLI（以及 docker-compose，如果你需要独立的 v1 工具）：

```bash
brew install colima docker docker-compose
```

说明：
- `docker`：CLI 客户端
- `docker-compose`：独立的 docker-compose（若你使用的是 Docker Compose v2，可直接使用 `docker compose`）

---

## 启动 Colima（示例配置）
在 M1/M2 等 Apple Silicon 上建议指定架构为 aarch64（ARM64）。下面是一个示例启动命令：

```bash
colima start \
  --arch=aarch64 \
  --cpu=4 \
  --memory=8 \
  --disk=20
```

参数建议：
- --arch=aarch64：强制 ARM64 架构，适配 M 系列芯片
- --cpu：分配 CPU 核心数（建议为物理核心数的一半或按需）
- --memory：分配内存（单位 GB，建议 >=4GB，示例用 8GB）
- --disk：分配磁盘大小（单位 GB）

启动后查看状态：
```bash
colima status
```

---

## 为 Docker 配置镜像加速（示例）
创建或更新 Docker daemon 配置（~/.docker/daemon.json），并重启 Colima 使其生效。示例使用阿里云镜像加速地址（请替换为你自己的镜像地址）：

```bash
# 创建 docker 配置目录（如果不存在）
mkdir -p ~/.docker

# 将镜像加速写入 daemon.json（替换为你自己的 registry 镜像）
cat > ~/.docker/daemon.json <<'EOF'
{
  "registry-mirrors": ["https://lthkby97.mirror.aliyuncs.com/"]
}
EOF

# 重启 Colima 使配置生效
colima restart
```

验证 Docker 是否使用了配置：
```bash
docker info
# 在输出中查找 registry-mirrors 字段或与镜像加速相关的信息
```

---

## 常用命令示例
- 在使用 docker-compose（v1）时：
  ```bash
  # 在 docker-compose.yml 同级目录
  docker-compose down
  docker-compose up -d --build
  docker-compose logs -f
  ```
- 使用 Docker CLI / Compose v2：
  ```bash
  docker compose up -d --build
  docker compose ps
  docker system prune -f
  ```

---

## 常见问题与排查
- Colima 无法启动或��错：
  - 查看日志：`colima status` 或 `colima stop && colima start --verbose`
  - 检查 Homebrew 安装的 colima 版本是否为最新：`brew upgrade colima`
- 镜像拉取慢：确认 daemon.json 中的 registry-mirrors 是否正确、Colima 是否重启生效。
- 兼容性问题：部分工具或集成（如 Docker Desktop 专属功能）无法通过 Colima 完全替代，请根据项目需求决定是否继续使用 Docker Desktop。

---

## 升级与卸载 Colima
- 升级：
  ```bash
  brew upgrade colima
  ```
- 卸载：
  ```bash
  colima stop
  brew uninstall colima
  rm -rf ~/.colima
  ```

---

## 参考与致谢
- Colima 官方仓库：https://github.com/abiosoft/colima
- Homebrew：https://brew.sh
