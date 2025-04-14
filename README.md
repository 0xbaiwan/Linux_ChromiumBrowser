# 在 Linux 上安装 Chromium 浏览器
Chromium 是由 Google 开发的开源浏览器项目，旨在构建一个更安全、更快速、更稳定的浏览器
* 你可以在没有图形界面的 Linux 服务器上轻松访问浏览器
* 你可以轻松运行 Node 扩展
* 非常适合用来跑 Depin 项目

## 安装 Docker
```console
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 检查 Docker 版本
docker --version
```

## 检查时区
```
realpath --relative-to /usr/share/zoneinfo /etc/localtime
```

## 安装 Chromium
**1. 创建目录**
```
mkdir chromium
cd chromium
```

**2. 创建 `docker-compose.yaml` 文件**
```
nano docker-compose.yaml
```

**3. 在文件中粘贴以下代码**
* `CUSTOM_USER` 和 `PASSWORD`：替换为你喜欢的登录凭据
* `TZ`：替换为你服务器的时区
* `CHROME_CLI`：浏览器打开时的主页
* `ports`：如果端口有冲突，可以替换 `3010` 和 `3011`
```
---
services:
  chromium:
    image: lscr.io/linuxserver/chromium:latest
    container_name: chromium
    security_opt:
      - seccomp:unconfined #可选
    environment:
      - CUSTOM_USER=your_username    #替换为你的用户名
      - PASSWORD=your_password       #替换为你的密码
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai            #替换为你的时区
      - CHROME_CLI=https://github.com/0xbaiwan #可选
    volumes:
      - /root/chromium/config:/config
    ports:
      - 3010:3000                   #至少保留一个端口
      - 3011:3001   #如果需要，将 3011 更改为你喜欢的端口
    shm_size: "1gb"
    restart: unless-stopped
```
> 保存并退出：按 `Ctrl+X+Y+Enter`

## 运行 Chromium
```console
cd $HOME && cd chromium

docker compose up -d
```
**可以通过在本地 PC 浏览器中访问以下地址之一来访问应用程序**
* http://服务器IP:3010/
* https://服务器IP:3011/

---

# ⭐ 在 Chromium 上安装代理
## 1) 购买代理
* 你可以使用任何可靠的平台购买**静态住宅**代理。
- 免费静态住宅代理：
   - [WebShare](https://www.webshare.io/?referral_code=gtw7lwqqelgu)
   - [ProxyScrape](https://proxyscrape.com/)
   - [MonoSans](https://github.com/monosans/proxy-list)
- 付费高级静态住宅代理：
   - [922proxy](https://www.922proxy.com/register?inviter_code=d6416857)
   - [Proxy-Cheap](https://app.proxy-cheap.com/r/Pd6sqg)
   - [Infatica](https://dashboard.infatica.io/aff.php?aff=580)
- 付费动态IP代理
   - [IPRoyal](https://iproyal.com/?r=733417)

## 2) 在 Docker 中安装代理
**1- 停止当前运行的容器**
```bash
docker compose down -v
```

**2- 更新你的 `docker-compose.yml`：**
* 用你的 Chromium 凭据替换 `CUSTOM_USER` 和 `PASSWORD`。
* 根据你的代理是 `http` 还是 `socks5`，删除其中一个 `CHROME_CLI` 行，并用你的代理地址和端口替换 `proxy.example.com:1080`。
```yaml
---
services:
  chromium:
    image: lscr.io/linuxserver/chromium:latest
    container_name: chromium
    security_opt:
      - seccomp:unconfined #可选
    environment:
      - CUSTOM_USER=     #替换用户名
      - PASSWORD=    #替换密码
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - CHROME_CLI=--proxy-server=http://proxy.example.com:1080 https://google.com
      - CHROME_CLI=--proxy-server=socks5://proxy.example.com:1080 https://google.com
    volumes:
      - /root/chromium/config:/config
    ports:
      - 3010:3000   #如果需要，将 3010 更改为你喜欢的端口
      - 3011:3001   #如果需要，将 3011 更改为你喜欢的端口
    shm_size: "1gb"
    restart: unless-stopped
```

**3- 启动容器**
```bash
docker compose down -v
docker compose up -d
```

**4- 使用 `http://服务器IP:3010/` 或 `https://服务器IP:3011/` 访问你的 Chromium**
* 首先需要输入你的 `chromium` 凭据，然后需要输入 `代理` 凭据（如果你的代理有凭据）。


## 可选：停止并删除 Chromium
```
docker stop chromium
docker rm chromium
docker system prune
```
