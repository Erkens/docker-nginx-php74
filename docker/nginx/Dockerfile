FROM nginx:1.16.1-alpine AS nginx

FROM nginx AS user-nginx
ARG TIMEZONE=Europe/Amsterdam
ARG USER_ID=1000
ARG GROUP_ID=1000

RUN apk add --update --no-cache \
        tzdata \
    && \
    cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
    echo ${TIMEZONE} > /etc/timezone && \
    apk del tzdata && \
    addgroup -g ${GROUP_ID} user && \
    mkdir -p /data /home/user && \
    adduser -D -u ${USER_ID} -h /home/user -G user user && \
    install -d -m 0755 -o user -g user /home/user && \
    chown -R user:user /data /home/user

COPY dev.conf /etc/nginx/conf.d/default.conf

WORKDIR /data