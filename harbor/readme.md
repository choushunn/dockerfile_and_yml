1. wget https://github.com/goharbor/harbor/releases/download/v2.1.5/harbor-offline-installer-v2.1.5.tgz
2. tar -zxvf harbor.tgz

### 推送镜像


- 
```
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: 192.168.2.220

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 9000

# https related config
#https:
  # https port for harbor, default is 443
 # port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
```

- ```
  vim /etc/docker/daemon.json
  # daemon.json
  {
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [
    "192.168.2.222:9000"
  ]
  }
  ```
- 重启docker systemctl restart docker.service
- 重启daemon
  `systemctl daemon-reload`
- 登录授权
  `docker login 192.168.5.13:9000 -u admin -p password`
- docker tag images
- docker push

/wwwroot/bzy_api                    /wwwroot
/wwwroot/bzy_api/config/supervisor   /etc/supervisor/conf.d
/wwwroot/bzy_api/config/nginx          /etc/nginx/conf.d

/wwwroot/bzy_wsbsdt/config/supervisor
docker pull 192.168.2.220:9000/web/django_web@sha256:d97e6b1c0519439c78b87e71942a8e427abf9a4479eda1ef464bce0cb6b63613

sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 10080:80/tcp \
  -p 10081:10081/udp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.2.220:10080" \
  -e KUBOARD_AGENT_SERVER_UDP_PORT="10081" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3

kubectl delete pod * --force --grace-period=0 -n kuboard