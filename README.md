## REFERENCE
[github](https://github.com/ALEXANDERSSONN)
## awaresome-compose 
[django](https://github.com/docker/awesome-compose/tree/master/django)

## wakatime project
[wakatime](https://wakatime.com/@spcn25/projects/glewvkqqyj?start=2023-02-27&end=2023-03-05)



# BUILD-IMAGE & TAG
- คำสั่งการ Build image
```
docker compose "django/compose.yaml" up -d --build
```
- คำสั่งการ Tag
```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

# PUSH IMAGE TO DOCKER HUB 
- คำสั่งเข้าสู่ระบบ Docker ใน VSCODE
```
docker login
```
- คำสั่ง Push Image To Docker Hub
```
docker push TARGET_IMAGE[:TAG]
```

# CREATE STACK DEPLOY
- สร้างไฟล์ compose.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    volumes:
      - static_data:/usr/src/app/static
    ports:
      - "8088:8000"
    depends_on:
      - db
    deploy:
      replicas: 1
      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
```
- นำ compose.yaml ไป Stack Deploy on local

# SWARM CLUSTER
- Revert Proxy compose.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - default
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    networks:
     - webproxy
     - default
    volumes:
      - static_data:/usr/src/app/static
    depends_on:
      - db
    deploy:
      replicas: 1
      labels:
        - traefik.docker.network=webproxy
        - traefik.enable=true
        - traefik.http.routers.${APPNAME}-https.entrypoints=websecure
        - traefik.http.routers.${APPNAME}-https.rule=Host("${APPNAME}$ URL ")
        - traefik.http.routers.${APPNAME}-https.tls.certresolver=default
        - traefik.http.services.${APPNAME}.loadbalancer.server.port=8000


      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
  webproxy:
    external: true
```

![image](https://user-images.githubusercontent.com/98762543/224471831-22d8396b-2c5b-4724-999c-69f6225a548f.png)