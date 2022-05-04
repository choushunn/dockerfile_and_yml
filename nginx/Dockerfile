FROM nginx:latest
WORKDIR /etc/nginx/conf.d
COPY ./nginx.conf web.conf
WORKDIR /etc/nginx/
CMD ["nginx", "-g", "daemon off;"]




