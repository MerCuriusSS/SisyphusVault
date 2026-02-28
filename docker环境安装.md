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

