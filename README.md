# Mac-OS-m-Colima-Docker
Mac OS m芯片系列 Colima 管理 Docker
**Colima**（开源替代）：仅提供 “轻量级 Linux 虚拟机”，并在虚拟机内预装 Docker Engine，替代 Docker Desktop 的 “虚拟机层”，配合独立的 Docker CLI 使用，轻量、免费、适配 ARM64（M1/M2/M3）

```plain
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

```plain
brew update
```

```plain
brew search docker
```

### 判定规则
1. 若搜索结果中包含 `docker``docker-compose` 等包 → 说明可直接卸载 Docker Desktop，使用 Colima 作为运行时。
2. 若搜索结果无相关包 → 建议先保留 Docker Desktop，或后续通过 Homebrew 安装 Docker 工具链。

若已确认可替代，执行以下命令彻底清理 Docker Desktop 残留文件：

```plain
# 删除主程序
sudo rm -rf /Applications/Docker.app
# 删除用户组容器数据
sudo rm -rf ~/Library/Group\ Containers/group.com.docker
# 删除用户配置目录
sudo rm -rf ~/.docker
# 删除系统级 Docker 命令软链
sudo rm -rf /usr/local/bin/docker*
sudo rm -rf /usr/local/bin/compose*
```

```plain
brew install colima docker docker-compose
```

```plain
colima start \
    --arch aarch64 \          # 强制指定 ARM64 架构，适配 M 芯片
    --cpu 4 \                 # 分配 4 核 CPU（建议为物理核心数的一半）
    --memory 8 \              # 分配 8GB 内存（根据项目需求调整，最小建议 4GB）
    --disk 20                 # 分配 20GB 磁盘空间（存储镜像和容器数据）
```

```plain
colima status
```

```plain
docker info #输出如图2信息
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/1001183/1768447612788-9a9c5015-1309-4125-a290-583acefc18f4.png)

```plain
# 创建 Docker 配置目录
mkdir -p ~/.docker

# 写入镜像源配置 该镜像地址为本人镜像加速地址
cat > ~/.docker/da< EOF
{
  "registry-mirrors": ["https://lthkby97.mirror.aliyuncs.com/"]
}
EOF

# 重启 Colima 使配置生效
colima restart

```

```plain
# cd docker-compose.yml 同级目录
docker-compose down && docker system prune -f # 构建并后台启动所有服务
docker-compose up -d --build
```

