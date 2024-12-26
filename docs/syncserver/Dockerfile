FROM rust:1.82.0-alpine3.20 AS builder

ARG ANKI_VERSION

# 更新apk包索引并安装构建依赖项，安装完成后清理缓存
RUN apk update && apk add --no-cache build-base protobuf && rm -rf /var/cache/apk/*

# 安装 Anki Sync Server，使用指定的 Git 标签
RUN cargo install --git https://github.com/ankitects/anki.git \
    --tag ${ANKI_VERSION} \
    --root /anki-server  \
    anki-sync-server

# 删除源代码和构建文件，减少镜像大小
RUN rm -rf /anki-server/.cargo /anki-server/src

# 使用更小的 Alpine 镜像作为最终运行环境
FROM alpine:3.20.2

# 创建一个非特权用户，并设定用户的家目录
RUN adduser -D -h /home/anki anki

# 复制已构建的二进制文件到最终镜像
COPY --from=builder /anki-server/bin/anki-sync-server /usr/local/bin/anki-sync-server

# 安装bash并清理缓存
RUN apk update && apk add --no-cache bash && rm -rf /var/cache/apk/*

# 设置运行时用户为 anki
USER anki

# 设置同步服务器的端口环境变量
ENV SYNC_PORT=${SYNC_PORT:-"8080"}

# 暴露端口
EXPOSE ${SYNC_PORT}

# 默认命令
CMD ["anki-sync-server"]

# 镜像维护者标签
LABEL maintainer="Jean Khawand <jk@jeankhawand.com>"
