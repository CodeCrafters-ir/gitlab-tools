# gitlab-tools

***

# gitlab runner with docker 
```
docker volume create gitlab-runner-config

docker run -d --name gitlab-runner --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v gitlab-runner-config:/etc/gitlab-runner \
     gitlab/gitlab-runner:latest

docker ps

docker exec -it gitlab

gitlab-runner register
```
