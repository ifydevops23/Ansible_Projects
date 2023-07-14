- Install Ansible

```
sudo apt update
sudo apt install ansible -y
sudo apt install python3-pip
#Incase there's an available upgrade
pip install --upgrade ansible
```

- Get the content of nginx.conf ready to be pasted in nginx.conf.j2
  
```
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    upstream backend {
        server 172.31.89.129:80;
        server 172.31.80.32:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}

```
