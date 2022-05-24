

# LAB-02


## Step
1. build four image
```bash
sudo docker build -t frontend .
sudo docker build -t text .
sudo docker build -t list .
sudo docker build -t img .
```
2. Write `docker-compose.yml`
```yml
version: '3.1'
services:
        frontend:
                image: frontend
                container_name: frontend
                environment:
                        HOSTIP: 10.100.100.49
                ports:
                        - 8080:8083
        text:
                image: text
                container_name: text
        list:
                image: list
                container_name: list
        img:
                image: img
                container_name: img
                ports:
                        - 8081:8081
```
3. `sudo docker-compose up`

## Test

1. 
```bash
sudo docker ps --format "table {{.Names}}" | grep -v "NAMES" | grep "img\|frontend\|text\|list" | sort | wc -l
```
2.
```bash
sudo docker exec -t frontend ping text -c 1 -W 1 | grep -P "64 bytes from .* \(.*\): icmp_seq=1"

sudo docker exec -t text ping list -c 1 -W 1 | grep -P "64 bytes from .* \(.*\): icmp_seq=1"

sudo docker exec -t img ping list -c 1 -W 1 | grep -P "64 bytes from .* \(.*\): icmp_seq=1"
```
3.
```bash
sudo docker port frontend | grep -P "8083\/.* -> 0\.0\.0\.0:8080"
sudo docker port img | grep -P "8081\/.* -> 0\.0\.0\.0:8081"
```
4.
```bash
sudo docker exec -t frontend env | grep HOSTIP | sed "s/[^0-9.]*//g"
```
5.
```bash
curl -s -w "%{http_code}" -o /dev/null 192.168.229.128:8080
curl -s 192.168.229.128:8080 | sha256sum | sed "s/[^0-9a-zA-Z]//g"
curl -s -w "%{http_code}" -o /dev/null $imgurl

```



