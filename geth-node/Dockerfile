FROM ethereum/client-go:v1.8.3 

RUN apk add --update git bash nodejs nodejs-npm perl
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org

RUN cd /root &&\
    git clone https://github.com/cubedro/eth-net-intelligence-api --depth 1 &&\
    cd eth-net-intelligence-api &&\
    cnpm install &&\
    cnpm install -g pm2

ADD start.sh /root/start.sh
ADD app.json /root/eth-net-intelligence-api/app.json
RUN chmod +x /root/start.sh

ENTRYPOINT /root/start.sh
