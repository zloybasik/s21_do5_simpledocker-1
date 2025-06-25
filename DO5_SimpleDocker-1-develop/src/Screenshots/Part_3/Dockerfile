FROM nginx

COPY nginx.conf /etc/nginx
COPY hello_World.c /home
COPY hello_world.fcgi /hom/hello_world.fcgi

WORKDIR /home

RUN apt-get update && \
    apt-get install -y gcc make libfcgi-dev spawn-fcgi && \
    gcc -o Hello_World hello_World.c -lfcgi && \
    chmod +x ./Hello_World &&\
    chmod +x ./hello_world.fcgi &&\
    && apt-get clean && rm -rf /var/lib/apt/lists/*

CMD spawn-fcgi -p 8080 ./Hello_World && nginx -g 'daemon off;'