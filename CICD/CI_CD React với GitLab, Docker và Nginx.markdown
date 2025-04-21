# Hướng Dẫn Cấu Hình CI/CD cho Ứng Dụng React (Vite) với GitLab Runner, Docker và Nginx

Hướng dẫn này sẽ giúp bạn thiết lập quy trình CI/CD để tự động hóa việc build, test và deploy ứng dụng React sử dụng Vite lên server Ubuntu thông qua GitLab Runner, Docker và Nginx.

## Chuẩn bị Server

### Cài đặt Docker trên Ubuntu VPS

Cài đặt Docker để quản lý các container chứa ứng dụng. [Hướng dẫn cài Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)

### Cài đặt Nginx trên Ubuntu VPS

Cài đặt Nginx để phục vụ các tệp tĩnh của ứng dụng React. [Hướng dẫn cài Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)

## Các bước thiết lập CI/CD

1. Tạo file `.gitlab-ci.yml` để cấu hình pipeline CI/CD.
2. Tạo file `Dockerfile` để định nghĩa cách build và chạy ứng dụng.
3. Cập nhật file `package.json` với các lệnh build phù hợp.
4. Thiết lập các biến môi trường trong GitLab để quản lý thông tin bảo mật.

## Quy trình CI/CD và Deploy

Quy trình CI/CD bao gồm các bước sau:

1. **Build Docker Image**:

   - Thiết lập thư mục làm việc `/app`.
   - Sao chép `package.json` và `package-lock.json` vào thư mục làm việc.
   - Cài đặt dependencies với `npm install`.
   - Sao chép toàn bộ mã nguồn vào thư mục làm việc.
   - Build ứng dụng React dựa trên môi trường (test hoặc production).
   - Sử dụng một image nhẹ (Node.js Alpine) để phục vụ các tệp tĩnh với `serve`.

2. **Đẩy Docker Image lên Docker Hub**:

   - Đăng nhập vào Docker Hub và đẩy image đã build lên.

3. **SSH vào Server và Pull Image**:

   - Kéo image mới từ Docker Hub về server.

4. **Cập nhật Container**:

   - Dừng và xóa container cũ (nếu có).
   - Xóa các image không sử dụng để tiết kiệm dung lượng.
   - Chạy container mới từ image vừa kéo về.

## Cấu Hình Chi Tiết

### 1. Cấu Hình File `.gitlab-ci.yml`

File `.gitlab-ci.yml` định nghĩa pipeline CI/CD với hai stage: `docker` (build và đẩy image) và `deploy` (triển khai lên server).

```yaml
image: node:latest

stages:
  - docker
  - deploy

services:
  - docker:dind

# Build và đẩy image cho nhánh develop
build_and_push_develop:
  image: docker
  services:
    - docker:dind
  stage: docker
  only:
    - develop
  script:
    - echo "Building Docker image for develop..."
    - docker build --build-arg BUILD_ENV=test --build-arg NGINX_PORT=$NGINX_PORT_TEST -t $IMAGE_NAME:test .
    - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
    - docker tag $IMAGE_NAME:test $DOCKER_USERNAME/$IMAGE_NAME:test
    - docker push $DOCKER_USERNAME/$IMAGE_NAME:test

# Build và đẩy image cho nhánh production
build_and_push_production:
  image: docker
  services:
    - docker:dind
  stage: docker
  only:
    - production
  script:
    - echo "Building Docker image for production..."
    - docker build --build-arg BUILD_ENV=production --build-arg NGINX_PORT=$NGINX_PORT_PRODUCTION -t $IMAGE_NAME:latest .
    - echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
    - docker tag $IMAGE_NAME:latest $DOCKER_USERNAME/$IMAGE_NAME:latest
    - docker push $DOCKER_USERNAME/$IMAGE_NAME:latest

# Deploy lên môi trường test
deploy_test:
  stage: deploy
  image: alpine:latest
  only:
    - develop
  before_script:
    - apk add --no-cache openssh-client sshpass
  script:
    - sshpass -p $SSH_PASSWORD ssh -p $SSH_PORT -o StrictHostKeyChecking=no $SSH_USERNAME@$SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:test && docker stop ${IMAGE_NAME}_test || true && docker rm ${IMAGE_NAME}_test || true && docker image prune -f && docker run -d --name ${IMAGE_NAME}_test -e NGINX_PORT=$NGINX_PORT_TEST -p $NGINX_PORT_TEST:$NGINX_PORT_TEST $DOCKER_USERNAME/$IMAGE_NAME:test"
  when: on_success

# Deploy lên môi trường production
deploy_production:
  stage: deploy
  image: alpine:latest
  only:
    - production
  before_script:
    - apk add --no-cache openssh-client sshpass
  script:
    - sshpass -p $SSH_PASSWORD ssh -p $SSH_PORT -o StrictHostKeyChecking=no $SSH_USERNAME@$SERVER_IP "docker pull $DOCKER_USERNAME/$IMAGE_NAME:latest && docker stop ${IMAGE_NAME}_production || true && docker rm ${IMAGE_NAME}_production || true && docker image prune -f && docker run -d --name ${IMAGE_NAME}_production -e NGINX_PORT=$NGINX_PORT_PRODUCTION -p $NGINX_PORT_PRODUCTION:$NGINX_PORT_PRODUCTION $DOCKER_USERNAME/$IMAGE_NAME:latest"
  when: manual
```

### 2. Cấu Hình File `Dockerfile`

File `Dockerfile` định nghĩa cách build ứng dụng React và chạy nó trong container.

```yaml
# Stage 1: Build ứng dụng React
FROM node:18 AS build

WORKDIR /app

ARG BUILD_ENV=test
ARG NGINX_PORT

# Hiển thị các biến môi trường
RUN echo "BUILD_EN
V: $BUILD_ENV"
RUN echo "NGINX_PORT: $NGINX_PORT"

# Sao chép package.json và package-lock.json
COPY package*.json ./

# Cài đặt dependencies
RUN npm install

# Sao chép mã nguồn
COPY . .

# Build ứng dụng dựa trên môi trường
RUN npm run build:$BUILD_ENV

# Stage 2: Phục vụ ứng dụng
FROM node:18-alpine

WORKDIR /app

# Cài đặt serve để chạy ứng dụng
RUN npm install -g serve

# Sao chép các tệp đã build
COPY --from=build /app/dist /app

# Chạy ứng dụng
CMD ["sh", "-c", "echo NGINX_PORT=$NGINX_PORT && serve -s . -l $NGINX_PORT"]

EXPOSE ${NGINX_PORT}
```

### 3. Cập Nhật File `package.json`

Thêm các script build cho các môi trường test và production.

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "build:test": "tsc && vite build --mode test",
    "build:production": "tsc && vite build --mode production",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  }
}
```

### 4. Thiết Lập Biến Môi Trường trong GitLab

1. Truy cập dự án trên GitLab.
2. Vào **Settings &gt; CI/CD &gt; Variables**.
3. Thêm các biến sau:
   - `DOCKER_USERNAME`: Tên người dùng Docker Hub.
   - `DOCKER_PASSWORD`: Mật khẩu hoặc token Docker Hub.
   - `SSH_USERNAME`: Tên người dùng SSH của server.
   - `SSH_PASSWORD`: Mật khẩu SSH.
   - `SSH_PORT`: Cổng SSH (mặc định là 22).
   - `SERVER_IP`: Địa chỉ IP của server.
   - `IMAGE_NAME`: Tên image Docker.
   - `NGINX_PORT_TEST`: Cổng Nginx cho môi trường test.
   - `NGINX_PORT_PRODUCTION`: Cổng Nginx cho môi trường production.
