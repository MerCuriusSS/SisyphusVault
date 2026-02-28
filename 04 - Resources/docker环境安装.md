#Areas/Coder/环境配置 

```bash
# 1. 卸载旧版本（如果有）
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 2. 安装所需依赖包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 3. 添加 Docker 官方 yum 源（国内阿里云镜像源，速度更快）
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

```bash
# 1. 安装 Docker CE（指定稳定版本，避免最新版兼容问题）
# 先查看可用版本：yum list docker-ce --showduplicates | sort -r
sudo yum install -y docker-ce-20.10.24 docker-ce-cli-20.10.24 containerd.io

# 2. 启动 Docker 并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 3. 验证 Docker 是否安装成功
sudo docker version
```

```bash
# 创建 Docker 配置目录
sudo mkdir -p /etc/docker

# 写入镜像加速配置（替换为你自己的阿里云加速地址，也可直接用下面的通用地址）
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
      "https://docker.m.daocloud.io",
      "https://docker.1panel.live",
      "https://hub.rat.dev"
    ]
}
EOF

# 重启 Docker 使配置生效
sudo systemctl daemon-reload
sudo systemctl restart docker
```

