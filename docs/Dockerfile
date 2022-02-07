# docker build -t guzzle:sphinx .
# Image Size: 119.08 MB

FROM python:alpine

LABEL Joey lixueli@gmail.com

WORKDIR /docs
ADD requirements.txt /docs

ENV MY_PROXY_URL="http://docker.for.mac.host.internal:1087"
ENV HTTP_PROXY=$MY_PROXY_URL \
    HTTPS_PROXY=$MY_PROXY_URL

RUN apk add --no-cache \
      make \
 && python3 -m pip install --no-cache-dir -r requirements.txt \
 && rm -rf /var/cache/apk/* \
 && rm -rf /root/.cache \
 && rm -rf /tmp/* \
 && rm -rf /docs/*

ENV HTTP_PROXY ""
ENV HTTPS_PROXY ""

CMD ["make", "html"]
