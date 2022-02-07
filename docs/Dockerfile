# docker build -t guzzle:sphinx .
# Image Size: 119.08 MB

FROM python:alpine

LABEL Joey lixueli@gmail.com

WORKDIR /docs
ADD requirements.txt /docs

RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories \
 && apk add --no-cache \
      make \
 && python3 -m pip install \
      --index-url http://mirrors.aliyun.com/pypi/simple \
      --trusted-host mirrors.aliyun.com \
      --no-cache-dir \
      -r requirements.txt \
 && rm -rf /var/cache/apk/* \
 && rm -rf /root/.cache \
 && rm -rf /tmp/* \
 && rm -rf /docs/*

CMD ["make", "html"]
