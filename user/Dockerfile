# user-service/Dockerfile

# 使用golang官方镜像，并命名为builder
FROM golang:1.18-alpine as builder

# 启用go Modules
ENV GO111MODULE on

# 安装git
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk update && \
    apk add --no-cache git


# 设置工作目录
WORKDIR /app/micro-user-service

# 将目录中代码拷贝到镜像中
COPY . .

# 下载依赖，
RUN  go env -w GOPROXY=https://mirrors.aliyun.com/goproxy,direct && go mod tidy

# 构建二进制文件，添加一些额外参数方便在alpin中运行它
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-w' -o micro-user-service ./main.go

# 第二阶段构造
FROM alpine:latest

# 更新依赖软件
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
    apk update && \
    apk add --no-cache bash ca-certificates &&\
    apk add curl

# 和上个阶段一样设置工作目录
RUN mkdir -p /app/logs
WORKDIR /app

# 这一步不再从宿主机拷贝二进制文件，而是从上一个阶段构建的 builder 容器中拉取
COPY --from=builder /app/micro-user-service/micro-user-service .

EXPOSE 9091

RUN chmod +x ./micro-user-service

# 启动用户服务
CMD ["./micro-user-service"]