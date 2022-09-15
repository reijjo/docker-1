FROM ubuntu

EXPOSE 8080

RUN apt update && apt upgrade -y && \
	apt install nodejs -y && \
	apt install npm -y && \
	npm install -g yarn -y && \
	mkdir firstone && cd firstone && \
	yarn set version berry \
	yarn init && \
	yarn install

RUN echo "var http = require('http');" >> myfirst.js && \
	echo "http.createServer(function (req, res) {" >> myfirst.js && \
	echo "	res.writeHead(200, {'Content-Type': 'text/html'});" >> myfirst.js && \
	echo "	res.end('Hello World yabadabaduu');" >> myfirst.js && \
	echo "}).listen(8080);" >> myfirst.js

#docker build -t node .
#docker run -p 8080:8080 -it node
#	->node myfirst.js -->localhost:8080
