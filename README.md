# decyphertek.io
=================
Decoding Technology

1. Kubernetes:
--------------
```
curl -sfL https://get.k3s.io | sh - 
# Check for Ready node, takes ~30 seconds 
sudo k3s kubectl get node
```

2. Kubeflow:
------------
```
sudo snap install microk8s --classic
sudo usermod -a -G microk8s $USER
newgrp microk8s
sudo chown -f -R $USER ~/.kube
# Modified - Delayed install metallb and ingress , so it works on startupscript, so doesnt delete anything.
microk8s enable dns ingress cert-manager hostpath-storage rbac
microk8s status --wait-ready
sudo snap install juju --classic 
mkdir -p ~/.local/share
microk8s config | juju add-k8s my-k8s --client
juju bootstrap my-k8s uk8sx
juju add-model kubeflow
sudo sysctl fs.inotify.max_user_instances=1280
sudo sysctl fs.inotify.max_user_watches=655360
sudo sh -c 'echo "fs.inotify.max_user_instances=1280" >> /etc/sysctl.conf'
sudo sh -c 'echo "fs.inotify.max_user_watches=655360" >> /etc/sysctl.conf'
juju deploy kubeflow --trust --channel=latest/stable
sudo systemctl enable snap.microk8s.daemon-containerd
sudo systemctl enable snap.microk8s.daemon-kubelite
sudo systemctl enable snap.microk8s.daemon-cluster-agent
sudo systemctl enable snap.microk8s.daemon-k8s-dqlite
sudo systemctl enable snap.microk8s.daemon-apiserver-kicker
juju status --watch 5s
```

3. Open Web UI:
---------------
* I may need Ollama3 added to the docker-compose.yml , still in Dev.
* Need a GPU or Very slow during testing. 
~~~
vim docker-compose.yml

services:

  ingress:
    image: nginx:latest
    container_name: nginx-reverse-proxy
    restart: always
    ports:
      - 9443:9443
      - 443:443
    volumes:
      - nginx_conf:/etc/nginx/conf.d/
      - nginx_key:/etc/ssl/private/
      - nginx_cert:/etc/ssl/certs/
      - nginx_log:/var/log/
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.19

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    expose:
      - "9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.20

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    expose:
      - "8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - /home/adminotaur/.docker:/app/backend/data
    networks:
      decyphertek-net:
        ipv4_address: 172.19.0.21

volumes:
  open-webui:
  nginx_conf:
  nginx_key:
  nginx_cert:

networks:
  decyphertek-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16

docker compose up -d
# need default.conf
# openssl self signed certs
# .env
~~~

4. Brainboard ( Terraform )
---------------------------
* https://www.brainboard.co/

5. Theory
--------
* AI should simulate brain function. The brain optimizes energy consumption and this can be achieved with Kubernetes.
