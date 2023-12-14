# Triển khai svn server trên linux

Cài đặt svn-server trên linux server bằng docker

Triển khai svn-client

## Cài đặt svn-server

Sử dụng docker image "elleflorio/svn-server" để triển khai server svn-docker

**Chuẩn bị:**

- Server linux có cài sẵn docker

- Truy cập server bằng ssh

**Bước 1:**

Tạo docker volumn cho image

```bash
docker volume create svn-root
```

Lưu ý: Cài đặt docker tool trước khi tạo volume

```bash
sudo apt install docker.io
```

**Bước 2:**

Chạy lệnh sau để chạy docker svn-server, với cấu hình port 7443 là port chính dùng cho svn server

```bash
docker run -dit --restart unless-stopped --name svn-server -v svn-root:/home/svn -p 7443:80 -p 3960:3960 -w /home/svn elleflorio/svn-server
```

**Bước 3:**

Tạo user và pass đầu tiên với tên root

```bash
docker exec -t svn-server htpasswd -b /etc/subversion/passwd root root@123
```

**Bước 4:**

Tạo thư mục chính tên là root

```bash
docker exec -it svn-server svnadmin create root
```

**Bước 5:**

Fix lỗi commit

```bash
docker exec -t svn-server sed -i "s/= r/= rw/g" /etc/subversion/subversion-access-control
```

Fix lỗi từ chối truy cập

```bash
docker exec svn-server chown -R apache:apache /home/svn/root
```

Bước 6:

Log in to your svn server using URL: http://localhost:7443/svn

### Script

Install SVN Server

```bash
#!/bin/bash
# Step 1
sudo apt install docker.io
docker volume create svn-root
# Step 1
docker run -dit --restart unless-stopped --name svn-server -v svn-root:/home/svn -p 7443:80 -p 3960:3960 -w /home/svn elleflorio/svn-server
# Step 1
docker exec -t svn-server htpasswd -b /etc/subversion/passwd root root@123
docker exec -it svn-server svnadmin create root
```

Create Member Account

```bash
#!/bin/bash
docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
khoavo \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
tritruong \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
dungdo \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
khoatran \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
vietnguyen \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
ductruong \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
minhhuynh \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
sybui \
iSoft@123

docker exec -t svn-server htpasswd -b /etc/subversion/passwd \
dungnguyen \
iSoft@123
```

Create Folders structure

```bash
#!/bin/bash
docker exec -it svn-server svnadmin create \
root

docker exec -it svn-server svnadmin create \
root/isoft

docker exec -it svn-server svnadmin create \
root/isoft/soft-team

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/projects/

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/docs

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/khoavo

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/tritruong

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/dungdo

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/dungnguyen

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/khoatran

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/vietnguyen

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/ductruong

docker exec -it svn-server svnadmin create \
root/isoft/soft-team/members/minhhuynh
```

Done!
