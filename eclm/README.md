## Manual build steps

1. Build Docker image

```sh
IMG_VERSION="0.1"
docker build --tag hpscterrsys/eclm:latest --tag hpscterrsys/eclm:$IMG_VERSION .
```

2. Commit image to DockerHub

```sh
docker login
docker push hpscterrsys/eclm:latest
docker push hpscterrsys/eclm:$IMG_VERSION
docker logout
```
