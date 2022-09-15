FROM debian

EXPOSE 9987/udp
EXPOSE 10011
EXPOSE 30033

RUN apt update -y && apt upgrade -y && apt install -y nano software-properties-common dirmngr apt-transport-https gnupg2 ca-certificates lsb-release debian-archive-keyring wget bzip2
RUN wget https://files.teamspeak-services.com/releases/server/3.13.6/teamspeak3-server_linux_amd64-3.13.6.tar.bz2
RUN tar -xf ./teamspeak3-server_linux_amd64-3.13.6.tar.bz2
RUN adduser repe --home /opt/teamspeak --shell /bin/bash --disabled-password
RUN mv teamspeak3-server_linux_amd64/* /opt/teamspeak/
RUN chown -R repe:repe /opt/teamspeak

USER repe

WORKDIR /opt/teamspeak

RUN touch .ts3server_license_accepted

CMD ["./ts3server_minimal_runscript.sh", "start"]

#docker build -t team .
#docker run -p 9987:9987/udp -p 10011:10011 -p 30033:30033 team
