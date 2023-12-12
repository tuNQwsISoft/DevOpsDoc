# Docker Document

## Image

**Docker image** là 1 đơn vị đóng gói chứa mọi thứ cần thiết để 1 ứng dụng chạy. Là thứ cần thiết để chạy 1 container. Có thể tự định nghĩa 1 image bằng `Dockerfile`

## Container

**Docker container** là một run-time environment mà ở đó người dùng có thể chạy một ứng dụng độc lập.

## Docker compose

**Docker compose** là một công cụ giúp định nghĩa và chạy multi-container trong những ứng dụng sử dụng Docker. Chúng ta định nghĩa Docker compose ở file `docker-compose.yaml`.

## Volumes

[Volumes](https://docs.docker.com/storage/volumes/) là cơ chế để đồng bộ hóa dữ liệu của container (sau khi xóa/dừng container và tạo/chạy lại thì dữ liệu vẫn được đảm bảo).

Một cơ chế khác là [bind mount](https://docs.docker.com/storage/bind-mounts/), khi dùng _bind mount_, một file hay thư mục ở máy chủ sẽ được mount vào container. File hay thư mục đó được tham chiếu bằng 'đường dẫn tuyệt đối'. Ngược lại, khi dùng **Volumes**, một thư mục mới được tạo trong thư mục lưu trữ của Docker (Docker's storage directory) trên máy chủ, và Docker quản lý nội dung của thư mục đó.

**Volumes** có nhiều lợi thế hơn _bind mounts_ ở chỗ:

- Volumes dễ back up hay di chuyển sang container hơn.
- Có thể quản lý _volumes_ thông qua Docker CLI commands hay Docker API.
- Volumes hoạt động trên cả Linux và Windows.
- Volumes có thể chia sẻ an toàn giữa nhiều container.
- Driver volumes cho phép lưu trữ volumes trên cloud hay remote host, cho phép mã hóa nội dung của volume và thêm chức năng khác.
- Nội dung của volumes mới có thể được điền trước bởi container.
- Volumes trên Docker Desktop có hiệu suất cao hơn nhiều so với `bind mounts` từ các máy chủ Mac và Windows.

2 Ví dụ dưới đây cho ra cùng 1 kết quả:

#### `--mount`

```properties
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

#### `-v`

```properties
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```
