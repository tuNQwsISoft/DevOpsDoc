# Chạy các container bằng Dockge

## Các kiến thức liên quan

- **Docker image** là 1 đơn vị đóng gói chứa mọi thứ cần thiết để 1 ứng dụng chạy. Là thứ cần thiết để chạy 1 container. Có thể tự định nghĩa 1 image bằng `Dockerfile`
- **Docker container** là một run-time environment mà ở đó người dùng có thể chạy một ứng dụng độc lập.
- **Docker compose** là một công cụ giúp định nghĩa và chạy multi-container trong những ứng dụng sử dụng Docker. Chúng ta định nghĩa Docker compose ở file `docker-compose.yaml`.

## Run Portainer container using Dockge

### Write docker-compose.yml that install and run Portainer

Chúng ta có thể dịch từ lệnh `docker run` sang docker-compose.yml file dùng ChatGPT hoặc có thể tự viết. Here's the file:

### **File:** `docker-compose.yaml`

```yaml
version: '3'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    ports:
      - 8000:8000
      - 9443:9443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
volumes:
  portainer_data: null
```

### Put it to the dockge and run it

Then we have Portainer container running

## Run Postgress container using Dockge

### **File:** `docker-compose.yaml`

```yaml
version: '3.5'
services:
  postgres:
    container_name: postgres_container
    image: postgres
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-P@ssw0rd}
      PGDATA: /data/postgres
    volumes:
      - postgres:/data/postgres
    ports:
      - 5432:5432
    networks:
      - postgres
    restart: unless-stopped
  pgadmin:
    container_name: pgadmin_container
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-admin@i-soft.com.vn}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-i-soft@2024}
      PGADMIN_CONFIG_SERVER_MODE: 'True'
    volumes:
      - pgadmin:/var/lib/pgadmin
    ports:
      - ${PGADMIN_PORT:-5050}:80
    networks:
      - postgres
    restart: unless-stopped
networks:
  postgres:
    driver: bridge
volumes:
  postgres: null
  pgadmin: null
```

## Run SQLServer container using Dockge

### **File:** `docker-compose.yaml`

```yaml
version: '3'
services:
  sql-server-2022:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: sql-server-2022
    environment:
      PATH: ${PATH:-/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin}
      MSSQL_RPC_PORT: 135
      CONFIG_EDGE_BUILD: null
      ACCEPT_EULA: ${ACCEPT_EULA:-Y}
      MSSQL_SA_PASSWORD: ${MSSQL_SA_PASSWORD:-dev@123456789}
    ports:
      - 1433:1433
    restart: always
    labels:
      com.microsoft.product: Microsoft SQL Server
      com.microsoft.version: 16.0.4035.4
      org.opencontainers.image.ref.name: ubuntu
      org.opencontainers.image.version: 20.04
      vendor: Microsoft
    runtime: runc
networks: {}
```

## Các lỗi đã gặp

- **".... must be a mapping":** thường là lỗi syntax.
