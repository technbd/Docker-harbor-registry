## Harbor Installation:

Harbor is an open-source container image registry that helps secure and manage container images. It extends the Docker Registry by adding features like role-based access control (RBAC), image vulnerability scanning, content signing, replication, and authentication.

It is widely used in Kubernetes and Docker environments, making it a great choice for on-premises or private cloud container registry solutions.


### Prerequisites:

- Docker engine: Version 20.10.10-ce+ or higher
- Docker Compose: docker-compose (v1.18.0+) or docker compose v2 (docker-compose-plugin)
- Minimum: 2 CPU, 4GB RAM, 40GB Disk 
- Network ports: 80, 443 and 4443



```
docker -v

Docker version 27.5.1, build 9f9e405
```


```
docker compose version

Docker Compose version v2.32.4
```




---
---



### Download Harbor:

```
curl -sSL -O https://github.com/goharbor/harbor/releases/download/v2.12.2/harbor-offline-installer-v2.12.2.tgz

or,

wget https://github.com/goharbor/harbor/releases/download/v2.12.2/harbor-offline-installer-v2.12.2.tgz
```



```
tar -xvzf harbor-offline-installer-v2.12.2.tgz

cd harbor
```


### Configure Harbor without HTTPS (Lab/Home Environment):

```
cp harbor.yml.tmpl harbor.yml
```


_Add/modify the following fields:_

```yaml
vim harbor.yml


#hostname: registry.domain.com
hostname: 192.168.10.191

http:
  port: 80

#https:
#  port: 443
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path


harbor_admin_password: YourStrongPassword

data_volume: /data

trivy:
  enabled: true
```




### Install Harbor:

- **Notary (Image Signing & Trust)** : Notary is a tool for signing and verifying container images, ensuring image integrity and authenticity.
- **Clair (Vulnerability Scanning)** : Clair is a security vulnerability scanner for container images. It scans images for known vulnerabilities (CVEs) in operating system packages and application dependencies.
- **Chart Repository Service (Helm Chart Storage)** : Harbor includes a Helm Chart Repository, allowing users to store and manage Helm charts for Kubernetes deployments.
- **Trivy** : Trivy is an open-source vulnerability scanner for container images, filesystems, and repositories. Harbor integrates Trivy as its default security scanner to detect Common Vulnerabilities and Exposures (CVEs) in images.



_Run the installation script:_
```
./install.sh

or,

./install.sh --with-trivy 

./install.sh --with-notary

./install.sh --with-clair

./install.sh --with-chartmuseum

./install.sh --with-notary --with-clair --with-trivy 

./install.sh --with-notary --with-clair --with-chartmuseum
```



_Check if Harbor containers are running:_
```
docker ps


CONTAINER ID   IMAGE                                   COMMAND                  CREATED          STATUS                             PORTS                                     NAMES
9a58d4c82ec6   goharbor/nginx-photon:v2.12.2           "nginx -g 'daemon of…"   14 seconds ago   Up 9 seconds (health: starting)    0.0.0.0:80->8080/tcp, [::]:80->8080/tcp   nginx
b9b3dda81948   goharbor/harbor-jobservice:v2.12.2      "/harbor/entrypoint.…"   14 seconds ago   Up 9 seconds (health: starting)                                              harbor-jobservice
9909f8f0099b   goharbor/harbor-core:v2.12.2            "/harbor/entrypoint.…"   14 seconds ago   Up 10 seconds (health: starting)                                             harbor-core
73b256ab3f90   goharbor/trivy-adapter-photon:v2.12.2   "/home/scanner/entry…"   14 seconds ago   Up 10 seconds (health: starting)                                             trivy-adapter
4cc91d295d64   goharbor/registry-photon:v2.12.2        "/home/harbor/entryp…"   14 seconds ago   Up 12 seconds (health: starting)                                             registry
29e5a250c8ee   goharbor/harbor-registryctl:v2.12.2     "/home/harbor/start.…"   14 seconds ago   Up 12 seconds (health: starting)                                             registryctl
fc1f328aa394   goharbor/redis-photon:v2.12.2           "redis-server /etc/r…"   14 seconds ago   Up 12 seconds (health: starting)                                             redis
2dfa87ff7a47   goharbor/harbor-db:v2.12.2              "/docker-entrypoint.…"   14 seconds ago   Up 12 seconds (health: starting)                                             harbor-db
df9bf38317f8   goharbor/harbor-portal:v2.12.2          "nginx -g 'daemon of…"   14 seconds ago   Up 12 seconds (health: starting)                                             harbor-portal
2dea8a352250   goharbor/harbor-log:v2.12.2             "/bin/sh -c /usr/loc…"   14 seconds ago   Up 13 seconds (health: starting)   127.0.0.1:1514->10514/tcp                 harbor-log
```





### Allow Insecure Registry (if Harbor is running without HTTPS):

If Harbor is running without SSL (HTTPS), Docker needs to be configured to allow an insecure registry.


```
vim /etc/docker/daemon.json


{
  "insecure-registries": ["192.168.10.191"]
}
```



_Restart the docker engine:_
```
systemctl restart docker
```



_Then, down and up containers:_
```
docker compose down

docker compose up -d
```



_Verify:_
```
curl -u admin:harbor2026 http://192.168.10.191/v2/ -v 
```





### Access Harbor:

Open a browser and go to `http://registry.domain.com` or `http://server_ip` 

- Username: `admin`
- Password: (the one you set in harbor.yml)




---
---




### Configure Docker to Use Harbor as a Registry:

_Login to the Docker registry:_

```
docker login registry.domain.com
```



### Tag this image for the private registry:

```
docker tag nginx:latest registry.domain.com/library/nginx:latest
```


_Example: If pushing to a project called `myproject` on `registry.domain.com`:_

```
docker tag nginx:latest registry.domain.com/myproject/nginx:latest
```


### Push the image to the registry:

```
docker push registry.domain.com/library/nginx:latest
```


_Example:_

```
docker push registry.domain.com/myproject/nginx:latest
```



### Pull the image from the registry: 

```
docker pull registry.domain.com/library/nginx:latest
```


_Example:_
```
docker pull registry.domain.com/myproject/nginx:latest
```



---
---



### Links:
- [github.com | harbor](https://github.com/goharbor/harbor/)
- [github.com | releases](https://github.com/goharbor/harbor/releases)






