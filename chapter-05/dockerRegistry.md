mkdir -p ~/docker-registry/data
sudo docker run -d -p 80:5000 --restart=always --name registry \
  -v ~/docker-registry/data:/var/lib/registry \
  registry:2
sudo nano /etc/docker/daemon.json
{
  "insecure-registries": ["registry.local"]
}
sudo systemctl restart docker
sudo nano /etc/hosts
127.0.0.1 registry.local
create Dockerfile
FROM nginx
# Add index.html
RUN echo "<html><body><h1>Hello World!</h1></body></html>" \
    >/usr/share/nginx/html/index.html
docker build -t hello .
docker tag hello-world registry.local/hello
docker push registry.local/hello
curl -X GET http://registry.local/v2/_catalog

------------------
docker run --name nginx -d nginx
docker exec -it nginx /bin/bash

ldd $(which nginx)
dd if=/dev/urandom of=/tmp/data bs=1M count=10

docker inspect -s nginx | jq '.[0].SizeRw'
docker system df -v
docker run -d -name redis1 redis:6.0.13-alpine
docker run -d --name redis1 redis:6.2.3-alpine
docker logs redis1 | grep version

mkdir /tmp/{lower,upper,work,mount}
echo "hello1" > /tmp/lower/hello1
echo "hello2" > /tmp/upper/hello2
mount -t overlay \
 -o rw,lowerdir=/tmp/lower,upperdir=/tmp/upper,workdir=/tmp/work \
 overlay /tmp/mount
ls -l /tmp/mount/
total 8
-rw-r--r-- 1 root root 7 Mar  1 21:26 hello1
-rw-r--r-- 1 root root 7 Mar  1 21:26 hello2
echo "hello3">/tmp/mount/hello3
ls -l /tmp/mount
total 12
-rw-r--r-- 1 root root 7 Mar  1 21:26 hello1
-rw-r--r-- 1 root root 7 Mar  1 21:26 hello2
-rw-r--r-- 1 root root 7 Mar  1 21:29 hello3

ls -l /tmp/upper/
total 8
-rw-r--r-- 1 root root 7 Mar  1 21:26 hello2
-rw-r--r-- 1 root root 7 Mar  1 21:29 hello3

rm /tmp/mount/hello1
ls -l /tmp/upper/
total 8
c--------- 2 root root 0, 0 Mar  1 21:30 hello1
-rw-r--r-- 1 root root    7 Mar  1 21:26 hello2
-rw-r--r-- 1 root root    7 Mar  1 21:29 hello3
----------------------

ROOT=$(docker inspect nginx | jq -r '.[0].GraphDriver.Data.MergedDir')
echo $ROOT 
/var/snap/docker/common/var-lib-docker/overlay2/bdd5f47bd1d23fd6b893189bdbaa3cd37afae0b308d6442bf8e4f69abd2b15bd/merged

ROOT=$(docker inspect nginx | jq -r '.[0].GraphDriver.Data.MergedDir')
ROOT=${ROOT%/merged}
docker exec nginx mount | grep $ROOT | tr [:,] '\n'

lowerdir=/var/snap/docker/common/var-lib-docker/overlay2/l/VZUGAPI536QDVAKM4UWK3ARWT5
/var/snap/docker/common/var-lib-docker/overlay2/l/6ADFU3EJR3K75CJC6RYQDJJ2OQ
/var/snap/docker/common/var-lib-docker/overlay2/l/V2O67STQUR3EW63RERYA3GEFE4
/var/snap/docker/common/var-lib-docker/overlay2/l/7TM5THAXTEVHRKVHP52XI6VZLV
/var/snap/docker/common/var-lib-docker/overlay2/l/GMGYTLMAOGI6P4AC7ESXQK5VXL
/var/snap/docker/common/var-lib-docker/overlay2/l/JOV2OUQJ2RFF5LIILQZQBDUSIM
/var/snap/docker/common/var-lib-docker/overlay2/l/N7A357MR23U5JLXGFRDTS55CJO
/var/snap/docker/common/var-lib-docker/overlay2/l/Y76LFXWV5XMM5TCVIGONKHLIT3
upperdir=/var/snap/docker/common/var-lib-docker/overlay2/bdd5f47bd1d23fd6b893189bdbaa3cd37afae0b308d6442bf8e4f69abd2b15bd/diff
workdir=/var/snap/docker/common/var-lib-docker/overlay2/bdd5f47bd1d23fd6b893189bdbaa3cd37afae0b308d6442bf8e4f69abd2b15bd/work
nouserxattr)

s -al /var/snap/docker/common/var-lib-docker/overlay2/bdd5f47bd1d23fd6b893189bdbaa3cd37afae0b308d6442bf8e4f69abd2b15bd/diff/tmp/
total 10248
-rw-r--r-- 1 root root 10485760 Mar  1 21:24 data

docker pull busybox:latest
skopeo copy docker-daemon:busybox:latest oci:busybox:latest
cd busybox
MANIFEST=$(jq -r .manifests[0].digest index.json | sed -e 's/sha256://')
LAYER=$(jq -r .layers[0].digest blobs/sha256/$MANIFEST | sed -e 's/sha256://')
tar tvf blobs/sha256/$LAYER
