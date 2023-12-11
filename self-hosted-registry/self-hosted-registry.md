# Build local registry để lưu trữ image nội bộ

## Các kiến thức liên quan

- **Registry** là nơi lưu trữ image.
- Cần **SSL/TSL certificate** để set được registry truy cập được từ bên ngoài.
- **Giao thức SSL/TSL** là giao thức mật mã được thiết kế để bảo mật các thông tin nhạy cảm trong quá trình truyền trên internet như, thông tin cá nhân, thông tin thanh toán, thông tin đăng nhập.

## Khi nào nên dùng local registry

- Muốn kiểm soát hoàn toàn các image của mình.
- Implement CI/CD system.
- Chia sẻ image trong mạng nội bộ.

## Chạy docker container Registry có thể truy cập từ bên ngoài

Để Registry có thể truy cập được từ bên ngoài, bắt buộc phải áp dụng chứng chỉ SSL/TLS cho nó.

### 1. Lấy chứng chỉ SSL/TLS

Chúng ta có thể tự tạo chứng chỉ SSL/TLS miễn phí với [Let's Encrypt](https://letsencrypt.org/).\
Để lấy chứng chỉ cho domain của mình từ _Let's Encrypt_, chúng ta phải chứng minh được mình sở hữu domain đó bằng [ACME protocol](https://datatracker.ietf.org/doc/html/rfc8555) thông qua các [ACME clients](https://letsencrypt.org/docs/client-options/)

#### Cerbot

[Certbot](https://certbot.eff.org/) là client được khuyên dùng để bắt đầu. Nó đơn giản, tương thích với nhiều hệ điều hành và có document đầy đủ, dễ hiểu.\
Để dùng được _Certbot_, chúng ta cần đảm bảo:

- Truy cập được vào command line của server domain.
- Có website HTTP đang chạy trên port 80.

1. (Optional) Nếu không có website default chạy trên port 80, chúng ta host 1 website nginx trên server của mình:

   - Cài đặt Nginx:

   ```properties
   $ sudo apt update
   $ sudo apt install nginx
   ```

   - Cấu hình domain website nginx:

   ##### **File:** `/etc/nginx/sites-enabled`

   ```
   server {
       listen 80 default_server;
       listen [::]:80 default_server;

       server_name my.example.domain.com;

       location / {
               ...
       }
   }
   ```

   **\*** Thay _my.example.domain.com_ bằng tên domain của mình.

2. Cài chứng chỉ SSL/TLS từ Letsencrypt:

   - Cài đặt certbot và plugin của Nginx server:

   ```properties
   $ sudo apt install certbot python3-certbot-nginx
   ```

   - Cài chứng chỉ:

   ```properties
   $ sudo certbot --nginx -d my.example.domain.com
   ```

   **\*** Thay _my.example.domain.com_ bằng tên domain của mình.

   Certbot sẽ tự động thay đổi cấu hình mặc định của Nginx.

   ##### **File:** `/etc/nginx/sites-enabled`

   ```
   server {

     server_name my.example.domain.com;

     location / {
       ...
     }

       listen [::]:443 ssl ipv6only=on; # managed by Certbot
       listen 443 ssl; # managed by Certbot
       ssl_certificate /etc/letsencrypt/live/my.example.domain.com/fullchain.pem; # managed by Certbot
       ssl_certificate_key /etc/letsencrypt/live/my.example.domain.com/privkey.pem; # managed by Certbot
       include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
       ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

   }
   server {
       if ($host = my.example.domain.com) {
           return 301 https://$host$request_uri;
       } # managed by Certbot

     listen 80 default_server;
     listen [::]:80 default_server;

     server_name my.example.domain.com;
       return 404; # managed by Certbot
   }
   ```

   - Test auto-renewal

   ```properties
   $ certbot renew --dry-run
   ```

   Bây giờ, domain của chúng ta đã được cài chứng chỉ SSL/TLS thông qua port 443.

   SSL Certificate ở đường dẫn: `/etc/letsencrypt/live/my.example.domain.com/fullchain.pem`
   SSL Certificate Key ở đường dẫn: `/etc/letsencrypt/live/my.example.domain.com/privkey.pem`

   Chi tiết tại: [Reverse proxy nginx letsencrypt tutorial](https://github.com/christianlempa/videos/tree/main/nginx-reverseproxy)

3. Chạy container Registry với chứng chỉ vừa cài:

   - Tạo thư mục `certs`:

   ```properties
   $ mkdir -p certs
   ```

   - Copy _Certificate_ sang `domain.crt` và _Certificate Key_ sang `domain.key` ở thư mục `certs`:

   ```properties
   $ cat /etc/letsencrypt/live/my.example.domain.com/fullchain.pem > domain.crt
   $ cat /etc/letsencrypt/live/my.example.domain.com/privkey.pem > domain.key
   ```

   - (Optional) Dừng container Registry nếu nó đang chạy:

   ```properties
   $ docker container stop registry
   ```

   - Chạy container Registry với chứng chỉ vừa cài:

   ##### Chạy bằng `docker run`

   ```properties
   $ docker run -d \
     --restart=always \
     --name registry \
     -v "$(pwd)"/certs:/certs \
     -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
     -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
     -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
     -p 443:443 \
     registry:2
   ```

   ##### Hoặc **File:** `docker-compose.yaml`

   ```yaml
   version: '3'
   services:
     registry:
       image: registry:2
       container_name: registry
       restart: always
       ports:
         - '443:443'
       environment:
         - REGISTRY_HTTP_ADDR=0.0.0.0:443
         - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
         - REGISTRY_HTTP_TLS_KEY=/certs/domain.key
       volumes:
         - './certs:/certs'
   ```

   - Các máy khác bây giờ có thể pull và push tới registry vừa host sử dụng domain của nó ở port 443. Ví dụ:

   ```properties
   $ docker pull ubuntu:16.04
   $ docker tag ubuntu:16.04 my.example.domain.com:443/my-ubuntu
   $ docker push my.example.domain.com:443/my-ubuntu
   $ docker pull my.example.domain.com:443/my-ubuntu
   ```

4. Nhược điểm:
   Mặc dù `certbot` có cơ chế tự động gia hạn sau 3 tháng (Chứng chỉ lấy từ **Let's encrypt** hết hạn sau 3 tháng), nhưng tại vì chúng ta copy thủ công chứng chỉ từ file gốc ở bước **3.2** nên sau 3 tháng chúng ta vẫn phải gia hạn thủ công bằng cách lặp lại bước **3.2** để lấy chứng chỉ đã được gia hạn.

Chi tiết tại: [Run an externally-accessible registry](https://distribution.github.io/distribution/about/deploying/#run-an-externally-accessible-registry)

## Cấu hình Registry

Chúng ta có thể cấu hình Registry thông qua file docker-compose. Ví dụ:

- Storage: Mặc định cấu hình sẽ là:

```yaml
storage:
  filesystem:
    rootdirectory: /var/lib/registry
```

Để viết đè lên cấu hình này, set biến environment như này:

```properties
REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/somewhere
```

Chi tiết tại: [Configuring a registry](https://distribution.github.io/distribution/about/configuration/)

## Reference

[Let's Encrypt](https://letsencrypt.org/)

[ACME protocol](https://datatracker.ietf.org/doc/html/rfc8555)

[Certbot](https://certbot.eff.org/)

[Reverse proxy nginx letsencrypt tutorial](https://github.com/christianlempa/videos/tree/main/nginx-reverseproxy)

[Run an externally-accessible registry](https://distribution.github.io/distribution/about/deploying/#run-an-externally-accessible-registry)

[Configuring a registry](https://distribution.github.io/distribution/about/configuration/)
