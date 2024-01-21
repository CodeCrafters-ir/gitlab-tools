# gitlab-ci with docker compose file
```
export GITLAB_HOME=$(pwd)/gitlab
export GITLAB_DOMAIN=gitlab.localhost.ir

sudo docker compose up -d

docker exec -it gitlab-ce grep 'Password:' /etc/gitlab/initial_root_password
```

***

# gitlab-tools

***

## gitlab-runner with docker for example
```
docker volume create gitlab-runner-config

docker run -d --name gitlab-runner --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v gitlab-runner-config:/etc/gitlab-runner \
     gitlab/gitlab-runner:latest

docker ps

docker exec -it gitlab-runner bash

gitlab-runner register


change user to gitlab-runner and chck this user is added sudo group
create ssh key and add to pub key to server and private key add to
variable gitlab-ci or use directly in container

adduser -aG sudo gitlab-runner
change gitlab-runner
ssh-keygen -t rsa -b 4096 -C "example@gmail.com"
ssh-copy-id -i ~/.ssh/id_rsa.pub username@ip
ssh -i ~/.ssh/id_rsa.pub username@ip
```

***

## gitlab-runner with docker:dind
```
step 1: check for overlay
    # lsmod | grep overlay

step 2: if not oerlay
    # modprobe overlay
    # grep '^overlay$' /etc/modules || echo overlay >>/etc/modules

step 3: config docker
    in this path /etc/docker/daemon.json add text Down:
        {
          "storage-driver": "overlay2"
        }

step 4: restart docker
    # systemctl restart docker


step 5: create docker network
    # docker network create gitlab-runner-net

step 6: dind
    # docker run -d \
      --name gitlab-dind \
      --privileged \
      --restart always \
      --network gitlab-runner-net \
      -v /var/lib/docker \
      docker:17.06.0-ce-dind \
        --storage-driver=overlay2

step 7: config gitlab runner
    # mkdir -p /srv/gitlab-runner
    # touch /srv/gitlab-runner/config.toml

step 8: up gitlab-runner
    # docker run -d \
      --name gitlab-runner \
      --restart always \
      --network gitlab-runner-net \
      -v /srv/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
      -e DOCKER_HOST=tcp://gitlab-dind:2375 \
      gitlab/gitlab-runner:alpine

step 9: register gitlab-runner
    # docker run -it --rm \
      -v /srv/gitlab-runner/config.toml:/etc/gitlab-runner/config.toml \
      gitlab/gitlab-runner:alpine \
        register \
        --executor docker \
        --docker-image docker:17.06.0-ce \
        --docker-volumes /var/run/docker.sock:/var/run/docker.sock

step 10: check config
    # cat /srv/gitlab-runner/config.toml


for more check this link:
    https://medium.com/@tonywooster/docker-in-docker-in-gitlab-runners-220caeb708ca
```
