# 우분투 기반 테크니카 Q 사설서버 설치과정

Ubuntu 18.04 LTS 로 초기 설치 후 유저는 ubuntu이고 테크니카 서버 소스코드의 위치는
`/home/ubuntu/DMTQ-server`로 복사했다고 가정하고 아래 명령어만 차례대로 수행하면 서버구축이 되도록 함.

웹서버, DNS서버의 설치, 설정이 되며 IOS및 안드로이드에서 정상적으로 동작하는 것을 확인함.
IOS는 DNS부분만 구축한 서버 IP를 넣어주면 되지만 안드로이드는 DNS도 수정해야 하고 CA 인증서를 신뢰하도록 추가해야한다.
추가해야할 인증서는 `ssl/rootCA.crt` 파일이며 해당 인증서 추가로 인한 보안적인 이슈에 대해서는 책임지지 않으며 아니면 직접 만들어서 설정해야 한다.

> 다른 리눅스 환경에서의 정상동작은 보증하지 않으며 다른부분을 직접 찾아 수정해야 할 것임

> ubuntu 유저로 수행한 내용

```sh
$ sudo apt update
$ sudo apt upgrade -y
$ sudo apt install apache2 php php-sqlite3 bind9 -y
$ git clone https://github.com/jbl428/DMTQ-ubuntu.git
$ cd DMTQ-ubuntu/
$ sudo cp -v sites-available/* /etc/apache2/sites-available/
$ sudo cp -rv ssl /etc/apache2/
$ sudo cp -vf bind/* /etc/bind/
$ sudo a2enmod rewrite ssl
$ sudo a2ensite server ssl
$ sudo a2dissite 000-default
$ sudo systemctl restart apache2
```

네임서버의 존파일의 IP부분을 현재 서버의 IP로 수정해야 하는데 vi로 수정해도 되고 아래 명령어에서 `<사설서버 IP>` 부분에 IP를 넣어주어 수행하면 된다.

```sh
$ sudo sed -i 's/192.168.10.20/<사설서버 IP>/' /etc/bind/zone
```

아래 명령어로 네임서버 재시작한다.

```sh
$ sudo systemctl reload bind9
```

네임서버가 정상적으로 된건지 확인하려면 아래 명령어 수행하여 `<사설서버 IP>` 부분에 현재 서버의 IP가 나온다면 OK.

```sh
$ dig @127.0.0.1 www.neonapi.com

; <<>> DiG 9.11.3-1ubuntu1.7-Ubuntu <<>> @127.0.0.1 www.neonapi.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20836
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 2d776bc8afba845e4b9ae2f15cee66bc396c44b6338f117a (good)
;; QUESTION SECTION:
;www.neonapi.com.               IN      A

;; ANSWER SECTION:
www.neonapi.com.        86400   IN      A       <사설서버 IP>

;; AUTHORITY SECTION:
neonapi.com.            86400   IN      NS      localhost.

;; ADDITIONAL SECTION:
localhost.              604800  IN      A       127.0.0.1
localhost.              604800  IN      AAAA    ::1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed May 29 11:02:20 UTC 2019
;; MSG SIZE  rcvd: 155
```

아래 명령어로 테크니카 소스코드내 디렉토리 이름을 변경한다.

```
$ mv /home/ubuntu/DMTQ-server/www.neonapi.com/api/accounts_server/ /home/ubuntu/DMTQ-server/www.neonapi.com/api/accounts
```

사설서버 소스코드내 `C:\dmtq.db3` 부분을 찾아 `/home/ubuntu/DMTQ-server/_info/dmtq.db3` 로 수정해야 하는데
아래 명령어로 해결된다.

```sh
$ cd ~
$ sed -i 's#C:.dmtq.db3#/home/ubuntu/DMTQ-server/_info/dmtq.db3#' DMTQ-server/dmqglb.mb.pmang.com/score/index.php DMTQ-server/dmqglb.mb.pmang.com/djmaxQ/_vendor/_config.php DMTQ-server/pmangplus.com/accounts/v3/global/login_dmq.php DMTQ-server/www.neonapi.com/api/accounts/v3/global/login_dmq.php DMTQ-server/_vendor/config.php
```

아래 명령어 수행하여 디비파일에 쓰기권한을 준다.

```
$ sudo chmod 707 /home/ubuntu/DMTQ-server/_info/
$ sudo chmod 606 /home/ubuntu/DMTQ-server/_info/dmtq.db3
```

`/home/ubuntu/DMTQ-server/_vendor/controller/UserController.php` 파일의 45번째 라인을 아래처럼 수정

```php
$memberId = UserModel::nextUserId(0);
```

이제 스마트폰에서 DNS, CA인증서 설정한 후에 테크니카 Q를 즐길 수 있다.

