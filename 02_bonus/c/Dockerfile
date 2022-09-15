FROM  ubuntu

RUN apt update && apt upgrade -y && \
	apt install build-essential -y && \
	apt install vim -y && \
	apt install git -y

RUN echo "syntax on" >> ~/.vimrc && \
	echo "set number" >> ~/.vimrc && \
	echo "set tabstop=4" >> ~/.vimrc && \
	echo "colorscheme elflord" >> ~/.vimrc

#docker build -t cee .
#docker run -it cee
