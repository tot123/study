# Docker noVNC Chrome 多用户隔离环境部署

> 端口 8070，单容器多用户，登录信息永久保存，显卡 6 加速，Chrome 禁更新+固定主页
>
> **已知限制**：未实现开机自动启动 Chrome、未实现远程密码校验

## docker-compose.yml

```yaml
services:
  novnc-chrome-multi:
    image: dorowu/ubuntu-desktop-lxde-vnc
    container_name: novnc-multi-user
    restart: always
    ports:
      - "8070:80"
      - "9222:9222"
    environment:
      - VNC_PASS=master123
      - RESOLUTION=1920x1080
      - LANG=zh_CN.UTF-8
      - LANGUAGE=zh_CN:zh
      - LC_ALL=zh_CN.UTF-8
      - NO_SCREENSAVER=1
      - IDLE_TIME=0
      - NVIDIA_VISIBLE_DEVICES=6
      - NVIDIA_DRIVER_CAPABILITIES=all
      - STARTUP=bash -c "gsettings set org.gnome.desktop.session idle-delay 0; gsettings set org.gnome.desktop.screensaver lock-enabled false; mkdir -p /etc/opt/chrome/policies/managed; [ -f /etc/opt/chrome/policies/managed/policy.json ] || echo '{\"UpdateDisabled\":true,\"HomepageLocation\":\"https://www.baidu.com\"}' > /etc/opt/chrome/policies/managed/policy.json; id user1 &>/dev/null || useradd -m -d /home/user1 user1 && echo user1:user1pass | chpasswd; id user2 &>/dev/null || useradd -m -d /home/user2 user2 && echo user2:user2pass | chpasswd; id user3 &>/dev/null || useradd -m -d /home/user3 user3 && echo user3:user3pass | chpasswd; chmod 700 /home/user1 /home/user2 /home/user3; sleep 2; exec startlxde"
    volumes:
      - ./users/user1:/home/user1
      - ./users/user2:/home/user2
      - ./users/user3:/home/user3
      - /dev/shm:/dev/shm
    shm_size: 2g
    privileged: true
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

## 启动命令

```bash
docker compose down
docker compose up -d
```

## 访问地址

```
http://你的服务器IP:8070
```

## 账号信息

| 用途 | 用户名 | 密码 |
|------|--------|------|
| VNC 登录 | - | `master123` |
| 系统用户 1 | `user1` | `user1pass` |
| 系统用户 2 | `user2` | `user2pass` |
| 系统用户 3 | `user3` | `user3pass` |

## 功能特性

- 单容器多用户隔离
- 登录信息永久保存（volume 挂载）
- 显卡 6 加速（NVIDIA GPU）
- 禁锁屏/休眠
- Chrome 禁更新+固定主页

## 已知限制

- 未实现开机自动启动 Chrome
- 未实现远程密码校验（VNC 密码为固定值）
