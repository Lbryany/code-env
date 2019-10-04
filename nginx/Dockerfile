FROM nginx:alpine

LABEL maintainer="Bryan Lee <me@lbryany.com>"

COPY nginx.conf /etc/nginx/


#更新软件包之前设置源为阿里云
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories \
    && apk update \
    && apk upgrade \
    && apk --update add logrotate \
    && apk add --no-cache openssl \
    && apk add --no-cache bash

#创建用户
RUN set -x ; \
    addgroup -g 666 -S www ; \
    adduser -u 666 -D -S -G www www && exit 0 ; exit 1

RUN touch /var/log/messages

#复制logrotate文件
COPY logrotate/nginx /etc/logrotate.d/

RUN rm /etc/nginx/conf.d/default.conf

CMD ["nginx"]

EXPOSE 80 443