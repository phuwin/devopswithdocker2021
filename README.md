# devopswithdocker2021

# 3.1 A deployment pipeline to heroku
URL: https://github.com/phuwin95/docker-hy

# 3.3 Non-root user for backend and frontend
Dockerfile for the frontend:
```Dockerfile
FROM node:14

EXPOSE 5000

WORKDIR /usr/src/app

COPY . .

RUN useradd -m app 
RUN	chown app -R /usr/src/app
RUN	chown app -R /usr/local/lib/node_modules

USER app

RUN npm install
RUN REACT_APP_BACKEND_URL=http://localhost:8080 npm run build
RUN npm install -g serve

CMD serve -s -l 5000 build
```
Dockerfile for the backend:
```Dockerfile
FROM golang:1.16

EXPOSE 8080

WORKDIR /usr/src/app

RUN useradd -m app 
RUN	chown app -R /usr/src/app
USER app

COPY . .

ENV REQUEST_ORIGIN *

RUN go build

CMD ./server
```

# 3.4

Optimized Frontend Dockerfile (went from )
```Dockerfile
FROM ubuntu:18.04 

EXPOSE 5000

WORKDIR /usr/src/app

COPY . .

RUN apt-get update && apt-get install -y curl && \
    curl -sL https://deb.nodesource.com/setup_14.x | bash && \
    apt install -y nodejs && \
    apt-get purge -y --auto-remove curl && \
  	rm -rf /var/lib/apt/lists/* && \
    useradd -m app && \
    chown app -R /usr/src/app && \
    npm install && \
    REACT_APP_BACKEND_URL=http://localhost:8080 npm run build && \
    npm install -g serve 

USER app

CMD serve -s -l 5000 build
```

Optimized Backend Dockerfile
```Dockerfile
FROM ubuntu:18.04 

EXPOSE 8080

WORKDIR /usr/src/app

RUN apt-get update && apt-get install -y curl && \
    curl -s https://storage.googleapis.com/golang/go1.16.4.linux-amd64.tar.gz| tar -v -C /usr/local -xz && \
    apt-get purge -y --auto-remove curl && \
  	rm -rf /var/lib/apt/lists/* && \
    useradd -m app  && \
    chown app -R /usr/src/app

COPY . .

ENV REQUEST_ORIGIN *

RUN /usr/local/go/bin/go build

USER app

CMD ./server

```
Size differences:

```bash
backend-optimized        602MB
frontend-optimized       454MB
frontend                 1.17GB
backend                  1.01GB
```


# 3.5

Frontend optimized with alpine Dockerfile
```Dockerfile
FROM alpine:3.13

EXPOSE 5000

WORKDIR /usr/src/app

COPY . .

RUN apk add --update nodejs npm && \
    adduser -D app && \
    chown app -R /usr/src/app && \
    npm install && \
    REACT_APP_BACKEND_URL=http://localhost:8080 npm run build && \
    npm install -g serve 

USER app

CMD serve -s -l 5000 build
```

Backend optimized with alpine Dockerfile
```Dockerfile
FROM golang:alpine

EXPOSE 8080

WORKDIR /usr/src/app

RUN adduser -D app  && \
    chown app -R /usr/src/app

COPY . .

ENV REQUEST_ORIGIN *

RUN go build

USER app

CMD ./server
```

Size comparisons:
```bash
backend-optimized-alpine       447MB (from golang:alpine image)
frontend-optimized-alpine      290MB (from alpine image)
backend-optimized              602MB (from ubuntu image)
frontend-optimized             454MB (from ubuntu image)
frontend                       1.17GB (from node image)
backend                        1.01GB (from golang image)
```

# 3.6 Multistage backend

Multistage backend Dockerfile (Image generated 13.1mb)
```Dockerfile
FROM golang:alpine AS builder

EXPOSE 8080

WORKDIR /usr/src/app

RUN adduser -D app  && \
    chown app -R /usr/src/app

COPY . .

ENV REQUEST_ORIGIN *

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-w -s"

RUN ls

FROM scratch

COPY --from=builder /usr/src/app/server /server

EXPOSE 8080

ENTRYPOINT ["/server"]

```