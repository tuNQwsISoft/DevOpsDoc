# Dùng proxmox để tạo máy ảo Linux, Windows, MacOs

## Các kiến thức liên quan

- **Proxmox Virtual Environment** là 1 solution open source dùng để tạo và quản lý các máy ảo trên 1 giao diện web dễ sử dụng.
- **BIOS** (Basic Input/Output System):
  chịu trách nhiệm khởi tạo và test các thành phần hardware của máy tính khi khởi động.
- **Realm**: authentication methods.
- **CT** (Container Template): Máy ảo được tạo nên bởi template (giống image) của hệ điều hành.
- **VM** (Virtual Machine): Máy ảo được tạo nên bởi các file .iso của hệ điều hành.

## Tạo lại máy ảo ubuntu

### Rename hostname:

- Edit hostname hiện tại:

```properties
$ sudo nano /etc/hostname
```

- Thông tin host:

```properties
$ sudo nano /etc/hosts
```

- Kích hoạt hostname mới:

```properties
$ sudo hostnamectl set-hostname your_new_hostname
```

- Khởi động lại service:

```properties
$ sudo systemctl restart systemd-hostnamed
$ sudo systemctl restart networking
```

## Install Docker, Docker Compose

### 1. Install Docker:

- Update Ubuntu System:

```properties
$ sudo apt update
```

- Cài đặt các Packages cần thiết:

```properties
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

- Thêm Docker’s Official GPG key:

```properties
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

(The official GPG key của Docker được lấy về và thêm vào hệ điều hành. Trước khi cài đặt, Docker packages được kiểm tra dùng key này).

- Thêm Docker Repository:

```properties
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

- Chỉ rõ Source cài đặt:

```properties
$ apt-cache policy docker-ce
```

- Cài đặt Docker:

```properties
$ sudo apt install docker-ce -y
```

- Kiểm tra Docker:

```properties
$ sudo systemctl status docker
$ docker --version
```

### 2. Install Docker Compose:

- Download Compose file into /usr/local/bin:

```properties
$ curl -L "https://github.com/docker/compose/releases/download/v2.17.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- Set quyền để docker-compose command có thể dùng được:

```properties
$ chmod +x /usr/local/bin/docker-compose
```

## Install Dockge

### Tạo thư mục lưu trữ Dockge

```properties
$ mkdir -p /opt/stacks /opt/dockge
$ cd opt/dockge
```

### Download the compose.yaml

```properties
$ curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml
```

### Start the server

```properties
$ docker compose up -d
```
