# 🐧 Linux-CLI-Note

**Debian / Armbian / Docker / Linux 高频命令行笔记库**

本项目用于记录在 Linux (Debian/Armbian) 环境下部署服务、容器管理、网络配置以及性能优化的常用命令行。作为个人的高频 CLI 速记本，本仓库特别针对 Homelab 玩家、旁路由网关配置、代理环境调试以及 Docker 容器化部署进行了梳理与总结。

**适用场景与环境：**
- Armbian / Debian / Ubuntu / WSL2
- Docker / Compose 容器化运维
- ARM 设备（玩客云、R2S、R4S、树莓派等）
- 旁路由、透明代理（sing-box、Xray-core、OpenWrt）
- IPTV / Streamlink / FFmpeg / 网络调试
- Cloudflare Pages / Workers / Tunnel 部署
- 日常 Linux CLI 高频命令速查

---

## 📂 目录导航
- 🐧 [1. Debian / Armbian 系统核心调优](#1-debian--armbian-系统核心调优)
- 🐳 [2. Docker & Compose 高效运维](#2-docker--compose-高效运维)
- 🚀 [3. 硬件性能调试与系统监控](#3-硬件性能调试与系统监控)
- 🌐 [4. 网络配置、旁路由与网关优化](#4-网络配置旁路由与网关优化)
- 🛠️ [5. Shell 自动化脚本与运维工具](#5-shell-自动化脚本与运维工具)
- 📺 [6. 影音流媒体与 IPTV 调试](#6-影音流媒体与-iptv-调试)
- 🧰 [7. WSL2 / Cloudflare / 云原生部署](#7-wsl2--cloudflare--云原生部署)
- 📦 [8. Git / GitHub / 版本管理](#8-git--github--版本管理)
- 📘 [9. 常用 Linux CLI 速查表](#9-常用-linux-cli-速查表)

---

## 🐧 1. Debian / Armbian 系统核心调优

### 🔧 基础更新与常用工具
```bash
# 更新软件包列表并升级系统
sudo apt update && sudo apt upgrade -y

# 安装常用基础工具
sudo apt install -y vim htop curl wget git net-tools lsof unzip

### 🧹 清理系统与空间释放



```bash
# 自动移除不需要的依赖包
sudo apt autoremove -y

# 清理下载的归档包缓存
sudo apt clean && sudo apt autoclean

# 限制系统日志体积（释放空间）
sudo journalctl --vacuum-size=100M

```

### 📈 查看系统与硬件信息



```bash
uname -a               # 打印系统内核信息
lsb_release -a         # 查看系统版本发行信息
cat /etc/os-release    # 查看 OS 详细信息
lscpu                  # 查看 CPU 架构与核心信息
lsblk                  # 列出所有块设备（磁盘/分区）
free -h                # 查看内存与 Swap 使用情况
df -h                  # 查看磁盘空间占用

```

### ⏱️ 时间、时区与服务管理



```bash
# 修改系统时区（如设置为 Asia/Shanghai）
dpkg-reconfigure tzdata

# 手动同步网络时间
ntpdate -u pool.ntp.org

# 重载 systemd 守护进程（修改 service 文件后必须执行）
systemctl daemon-reload

# 设置服务开机自启并立即启动 (例如 ssh)
systemctl enable --now ssh

# 查看服务运行状态
systemctl status <service_name>

```

### 🔥 ARM 设备进阶调优（Armbian）



```bash
# CPU 性能模式调整（System → CPU → Performance）
armbian-config

# 防止 SD 卡 / eMMC 过度写入，挂载 /tmp 到内存
sudo nano /etc/fstab
# 尾部添加此行：
tmpfs /tmp tmpfs defaults,noatime,nosuid,size=256m 0 0

```

---

## 🐳 2. Docker & Compose 高效运维



### 📦 安装与基础环境



```bash
# 官方一键安装脚本
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sh get-docker.sh

# 设置开机自启
sudo systemctl enable docker

# 将当前用户加入 docker 组（免 sudo 运行）
usermod -aG docker $USER

```

### 🧱 容器、镜像与日志操作



```bash
docker ps -a                                # 查看所有容器（含未运行）
docker images                               # 列出所有本地镜像
docker stats                                # 实时查看所有容器的 CPU/内存 占用状态
docker logs -f --tail 100 <container_name>  # 实时查看容器日志（尾部100行）
docker exec -it <container_name> /bin/bash  # 进入运行中的容器（bash 或 sh）
docker stop <container_name>                # 停止容器
docker rm <container_name>                  # 删除容器
docker rmi <image_name>                     # 删除镜像

```

### 🧰 Docker Compose 常用命令



```bash
docker compose up -d                        # 后台启动所有服务
docker compose down -v                      # 停止并移除容器、网络、挂载卷
docker compose logs -f                      # 查看编排下所有服务的日志
docker compose pull                         # 拉取最新的镜像
docker compose up -d --build --force-recreate # 重新拉取镜像并重建容器

```

### 🧼 深度清理Docker垃圾



```bash
# 清理所有未使用的镜像、容器、网络和无名卷（⚠️危险操作）
docker system prune -a --volumes

# 单独清理无用的数据卷和网络
docker volume prune
docker network prune

```

---

## 🚀 3. 硬件性能调试与系统监控



### 📊 实时资源监控



```bash
htop    # 动态资源监控（比 top 更直观，支持鼠标）
iotop   # 磁盘 I/O 实时监控
iftop   # 实时网卡流量监控

```

### 🔥 ARM 硬件温度与 CPU 频率监控



```bash
# 查看 CPU 温度 (Armbian 常用路径)
cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000 "°C"}'

# 查看 CPU 当前频率
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq

```

### 🧪 性能测速工具



```bash
# 磁盘顺序读取测速
sudo hdparm -Tt /dev/sda

# 互联网带宽测速
speedtest-cli

```

---

## 🌐 4. 网络配置、旁路由与网关优化



在部署 sing-box、Xray-core 或 OpenWrt 旁路由时，经常需要调试网络和路由表。

### 🔍 IP 与端口排查



```bash
ip a                           # 查看本机所有网卡及 IP 地址
ss -tulpn | grep LISTEN        # 查看监听端口（定位端口冲突非常有用）
sudo netstat -tunlp            # 网络连接、路由表与接口统计
sudo lsof -i :8080             # 查询特定端口被哪个进程占用

```

### 🛤️ 路由表与网关控制



```bash
ip route show                               # 查看当前系统路由表
ip route add default via 192.168.1.x        # 临时修改默认网关（适用于透明代理测试）
ip route del default                        # 删除旧的默认网关

```

### 📡 DNS 测试与调试



```bash
nslookup github.com                         # 测试域名解析是否正常
systemctl restart systemd-resolved          # 强制刷新/重启 DNS 解析服务

```

---

## 🛠️ 5. Shell 自动化脚本与运维工具



### 📜 脚本运行与权限控制



```bash
chmod +x echnet.sh     # 赋予脚本可执行权限
./echnet.sh            # 运行当前目录下的脚本

```

### ⏱️ 定时任务 (Cron)



```bash
crontab -e             # 编辑当前用户的定时任务

# 示例：每天凌晨 3 点执行网关切换脚本，并将日志输出到 /var/log
0 3 * * * /root/echnet.sh >> /var/log/echnet.log 2>&1

```

### 🛡️ 内网穿透与后台守护



```bash
# 使用 nohup 后台运行程序并忽略挂断信号 (如 Cloudflare Tunnel)
nohup ./cloudflared tunnel run <token> > cloudflared.log 2>&1 &

# 查看当前终端的后台任务
jobs -l

```

### ⚙️ 实用自动化单行脚本示例



```bash
# 自动重启 Docker 容器
#!/bin/bash
docker restart myapp

# 自动备份当前目录至 GitHub
#!/bin/bash
cd /root/myrepo
git add .
git commit -m "auto backup $(date)"
git push

# 自动抓取 IPTV 源（基于 Streamlink）
streamlink --stream-url <直播地址>

```

---

## 📺 6. 影音流媒体与 IPTV 调试



### 📡 测试与推流排查



```bash
# 测试直播源格式与是否可用
ffprobe <url>

# 基于 Streamlink 提取网页真实流地址
streamlink --stream-url <直播页面>

# FFmpeg 基础推流测试（将本地 MP4 推送至 RTMP 服务器）
ffmpeg -re -i input.mp4 -c copy -f flv rtmp://server/live/test

```

---

## 🧰 7. WSL2 / Cloudflare / 云原生部署



### 🐧 WSL2 网络代理设置



```bash
# 宿主机使用代理（如 v2rayN）时，让 WSL2 走宿主机代理网络
export http_proxy=[http://127.0.0.1:7890](http://127.0.0.1:7890)
export https_proxy=[http://127.0.0.1:7890](http://127.0.0.1:7890)

```

### ⚡ Cloudflare 前端与无服务器部署



```bash
# Cloudflare Pages 命令行打包与部署
npm run build
wrangler pages deploy dist

# Cloudflare Workers 简易反向代理示例 (JS)
export default {
  async fetch(request) {
    return fetch(request.url)
  }
}

```

---

## 📦 8. Git / GitHub / 版本管理



### 🔑 基础配置与提交



```bash
# 生成全新的 SSH Key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 基础提交三连
git add .
git commit -m "update"
git push

```

### 🧱 常用状态与回滚操作



```bash
git status                  # 查看工作区状态
git log --oneline           # 单行查看简洁的提交历史
git reset --hard HEAD~1     # 危险操作：强制回退到上一个版本并丢弃修改

```

---

## 📘 9. 常用 Linux CLI 速查表



### 📁 文件与目录操作



```bash
cp file1 file2              # 复制文件
mv file1 file2              # 移动 / 重命名文件
rm -rf dir                  # 强制递归删除目录
mkdir dir                   # 创建新目录

```

### 🔍 文本查找与定位



```bash
grep -rn "keyword" .        # 在当前目录及子目录下查找包含关键字的文件内容
find / -name "*.log"        # 全局查找所有后缀为 .log 的文件

```

### 📦 归档与解压缩



```bash
tar -czvf a.tar.gz dir      # 将目录打包并 gzip 压缩
tar -xzvf a.tar.gz          # 解压 gzip 归档文件

```

### 🔐 权限与所有者



```bash
chmod 755 file              # 设置文件为所有者可读写执行，其他人可读执行
chown user:user file        # 将文件所有权更改为指定用户和用户组

```

### 🌐 基础网络探测



```bash
curl -I [https://example.com](https://example.com) # 仅获取目标网址的 HTTP 响应头
ping -c 4 8.8.8.8           # 发送 4 个 ICMP 请求包测试网络连通性

```

```

```
