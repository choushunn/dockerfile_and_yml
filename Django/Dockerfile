FROM python:3.7
RUN mkdir /wwwroot  && mkdir -p /etc/supervisor/conf.d
WORKDIR /wwwroot
COPY ./requirements.txt ./requirements.txt
#安装 requirements
RUN python -m pip install --upgrade pip \
    && pip config set global.index-url https://pypi.douban.com/simple \
    && pip install -r requirements.txt \
    && pip install uwsgi \
##安装supervisor
    && pip install supervisor \
    && echo_supervisord_conf > /etc/supervisor/supervisord.conf \
    &&  sed -i 's/nodaemon=false/nodaemon=true/g' /etc/supervisor/supervisord.conf \
    && sed -i '$a[include]' /etc/supervisor/supervisord.conf \
    && sed -i '$afiles=/etc/supervisor/conf.d/*.conf' /etc/supervisor/supervisord.conf

#暴露端口
EXPOSE 8000

#运行程序
ENTRYPOINT ["supervisord", "-c", "/etc/supervisor/supervisord.conf"]


