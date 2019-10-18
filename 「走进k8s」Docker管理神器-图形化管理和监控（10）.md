

#### （一）portainer


``` bash

docker run -d -p 9000:9000 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --name prtainer \
    portainer/portainer

```


#### （二）cAdvisor


``` bash
docker run \
  --volume=/var/run:/var/run:rw \
  --volume=/:/rootfs:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```
