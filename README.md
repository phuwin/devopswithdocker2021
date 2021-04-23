# devopswithdocker2021
`docker-course-backend` and `docker-course-frontend` images are built using Dockerfiles mentioned in [1.14 anwser](https://github.com/phuwin95/devopswithdocker2021/tree/part1#114-environment)
# 2.3
docker-compose.yaml

```yaml
version: '3.5'

services:
  backend:
    image: docker-course-backend
    ports: 
      - 8080:8080

  frontend:
    image: docker-course-frontend
    ports: 
      - 5000:5000
```

# 2.4 redis
docker-compose.yaml

```yaml
version: '3.5'

services:
  backend:
    image: docker-course-backend
    ports: 
      - 8080:8080
    environment: 
      - REDIS_HOST=redis

  frontend:
    image: docker-course-frontend
    ports: 
      - 5000:5000

  redis: 
    image: redis
```

# 2.5 Scaling
commands:
```
docker-compose up -d --scale compute=10
```

# 2.6 Postgres
docker-compose.yaml
```yaml
version: '3.5'

services:
  postgres:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
  backend:
    image: docker-course-backend
    ports: 
      - 8080:8080
    environment: 
      - REDIS_HOST=redis
      - POSTGRES_HOST=postgres
    depends_on: 
      - postgres

  frontend:
    image: docker-course-frontend
    ports: 
      - 5000:5000

  redis: 
    image: redis
```

# 2.8 Nginx and 2.9 Db volume
`/nginx/nginx.conf`:
```conf
  events { worker_connections 1024; }

  http {
    server {
      listen 80;

      location / {
        proxy_pass http://frontend:5000;
      }

      location /api/ {
        proxy_set_header Host $host;
        proxy_pass http://backend:8080/;
      }
    }
  }
```

`docker-compose.yaml`:
```yaml
version: '3.5'

services:
  nginx:
    image: nginx
    ports: 
      - 80:80
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    networks: 
      - my-network

  postgres:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - ./data:/var/lib/postgresql/data
    networks: 
      - my-network

  backend:
    image: docker-course-backend
    ports: 
      - 8080:8080
    environment: 
      - REDIS_HOST=redis
      - POSTGRES_HOST=postgres
    depends_on: 
      - postgres
    networks: 
      - my-network

  frontend:
    image: docker-course-frontend
    ports: 
      - 5000:5000
    networks: 
      - my-network

  redis: 
    image: redis

networks:
  my-network:

```

