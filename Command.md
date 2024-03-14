# coin-community-backend

# Preparation
```
echo 'export WORKSPACE=/workspace' >> ~/.bashrc
```
# firewall
```
# 기본 정책을 차단으로 설정
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 특정 포트 허용
sudo ufw allow 22   # SSH
sudo ufw allow 80   # HTTP
sudo ufw allow 8080   # HTTP
sudo ufw allow 443  # HTTPS
sudo ufw allow 8443 # Custom port 8443
sudo ufw allow 9000 # Custom port 9000

# 방화벽 활성화
sudo ufw enable
```

# Docker
```
$ sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
$ for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
$ echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
 $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
 sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ sudo usermod -aG docker ubuntu
```

# [Docker Compose](https://docs.docker.com/compose/install/standalone/)
```
$ curl -L "https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

# Nginx, Jenkins
```
$ cd /workspace
$ docker-compose up -d
```

## Let's Encrypt
```
docker run -it --rm --name certbot \
            -v "${WORKSPACE}/etc/letsencrypt:/etc/letsencrypt" \
            -v "${WORKSPACE}/var/lib/letsencrypt:/var/lib/letsencrypt" \
            certbot/certbot \
            certonly \
            --webroot \
            -d 221.167.211.152.nip.io \
            --email hoban4336@gmail.com \
            --agree-tos

// dns-01 : DNS 를 이용한 preferred-challenges 를 사용 시 이 옵션을 사용해야 한다,.
// --preferred-challenges : DNS를 이용한 인증서 발급방식을 사용하기 위한 옵션이다.
// --server : Let's Encrypt 의 ACME (자동화된 인증서 관리 환경) 서버 주소, acme-v02 만 와일드 발급 가능.
// 에러 발생시, 수동 실행
docker run -it --rm --name certbot \
            -v "${WORKSPACE}/etc/letsencrypt:/etc/letsencrypt" \
            -v "${WORKSPACE}/var/lib/letsencrypt:/var/lib/letsencrypt" \
            certbot/certbot \
            certonly \
            --manual \
            --preferred-challenges dns-01 \
            --server https://acme-v02.api.letsencrypt.org/directory \
            -d "221.167.211.152.nip.io" \
            --email "hoban4336@gmail.com"
```

### Let's Encrypt 재발급
```
// @todo nginx reload 안해도 되는지 테스트 필요 : docker-compose exec nginx nginx -s reload
# crontab 
0 5 * */3 * /usr/bin/docker run --rm --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "${WORKSPACE}/var/lib/letsencrypt:/var/lib/letsencrypt" certbot/certbot renew >> /var/log/certbot_renew.log
```


# Spring Boot
```
$ cd /coin-community
$ ./gradlew build
$ docker build --tag coin-app:0.1 .
$ docker run --name coin-app -p 8181:8080 -d coin-app:0.1
```


# DNS-01 인증방식
  Let's Encrypt가 도메인 소유 여부를 확인하기 위해 DNS 레코드를 사용하는 방식입니다. 

# HTTP-01 인증방식

WildCard 인증서는 manual 모드로 받아야 하고, manual 모드는 renew 명령어로 갱신할 수 없다.
DNS의 TXT 레코드를 자동으로 변경해야 한다.

CloudFlare DNS


## /etc/letsencrypt/live/coinissue.kr/fullchain.pem": BIO_new_file() failed (SSL: error:0200100D