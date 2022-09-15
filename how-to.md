# How to install
	1. Open Managed Software Center on your Dock
	2. When it opens, write "docker" to the search field.
	3. Click open "Docker" from the search results and click the "Install" button.
	4. Wait
	5. Look for Docker in Finder and open the software to make sure it starts

	Then we'll move Docker's data directory to goinfre folder:
		1. Now you have Docker open and there's a small whale icon on the right side of your top bar next to the wifi icon. Click the docker icon and choose "Quit Docker Desktop" from the menu.
		2. Wait until Docker closes.
		3. Open terminal and create a new folder to store Docker data into: 'mkdir ~/goinfre/dockerdata/'
		4. Copy files to new folder: 'rsync -aP ~/Library/Containers/com.docker.docker/Data ~/goinfre/dockerdata/'
		5. Rename old data folder: 'mv ~/Library/Containers/com.docker.docker/Data ~/Library/Containers/com.docker.docker/Data.old'
		6. Create symbolic link: 'ln -s ~/goinfre/dockerdata/Data ~/Library/Containers/com.docker.docker/Data'

	Then the same for .docker folder in home directory:
		1. Copy files to new folder: 'rsync -aP ~/.docker ~/goinfre/dockerdata/'
		2. Rename old data folder: 'mv ~/.docker ~/.docker.old'
		3. Create symbolic link: 'ln -s ~/goinfre/dockerdata/.docker ~/.docker'

	Let's check that everything works:
		1. Start Docker Desktop. This takes a while so hold your horses. If you click the whale icon on your top right you can see 	the current situation. It's ready when you see green light and text "Docker Desktop is running".
		2. Test functionality by running a container in your terminal: 'docker run hello-world'
		3. If you get a message saying "Hello for Docker! ..." you're good to go.
		4. If everything works you can remove the backup folders: 'rm -rf ~/.docker.old ~/Library/Containers/com.docker.docker/Data.	old'
		5. Try once more that everything works: 'docker run hello-world'
# ------------------------------------- #

# 00_how_to_docker

# 1. Get the hello-world container from the Docker Hub, where it’s available.
	docker pull hello-world

# 2. Launch the hello-world container, and make sure that it prints its welcome message, then leaves it.
	docker run hello-world

# 3. Launch a nginx container, available on Docker Hub, as a background task. It
# should be named overlord, be able to restart on its own, and have its 80 port
# attached to the 5000 port of your machine. You can check that your container
# functions properly by visiting http://localhost:5000 on your web browser.
	docker run --restart always --name overlord -d -p 5000:80 nginx
		-> '--restart always' restarts always.
		-> '--name <NAME>' to give name to the container
		-> '-d' = detach (run container in background and print container ID)
		-> '-p <PORT:PORT>' = publish list (publish a container's port(s) to the host)

# 4. Get the internal IP address of the overlord container without starting its shell and
# in one command.
	docker inspect overlord | grep 'IPAddress'

# 5. Launch a shell from an alpine container, and make sure that you can interact
# directly with the container via your terminal, and that the container deletes itself
# once the shell’s execution is done.
	docker run -it --rm alpine /bin/sh
		-> '-i' = interactive (keep STDIN open even if not attached)
		-> '-t' allocates a pseudo-terminal
		-> '--rm' deletes cotainer after it exits

# 6. From the shell of a debian container, install via the container’s package manager
# everything you need to compile C source code and push it onto a git repo (of
# course, make sure before that the package manager and the packages already in the
# container are updated). For this exercise, you should only specify the commands
# to be run directly in the container.
	docker run -it --rm debian (to start debian)
		-Inside the terminal:
			apt update
			apt upgrade -y
			apt install vim -y
			apt install build-essential -y
			apt-get install git -y
					vim test.c
						#include <unistd.h>
						int main(void)
						{
						write(1, "z\n", 2);
						}
					gcc test.c
					./a.out

# 7. Create a volume named hatchery.
	docker volume create hatchery
		-> -v <source>:<destination>:<options>
		-> For instance, you can mount your local ./target onto the /usr/share/nginx/html directory container or an nginx container to visualize your html files.
		-> echo "<h1>Hello from Host</h1>" > ./target/index.html
			-> docker run -it --rm --name nginx -p 8080:80 -v "$(pwd)"/target:/usr/share/nginx/html nginx
				-> Navigate to http://localhost:8080/ and you should see “Hello from Host”.

# 8. List all the Docker volumes created on the machine. Remember. VOLUMES.
	docker volume ls

# 9. Launch a mysql container as a background task. It should be able to restart on its
# own in case of error, and the root password of the database should be Kerrigan.
# You will also make sure that the database is stored in the hatchery volume, that
# the container directly creates a database named zerglings, and that the container
# itself is named spawning-pool.
	docker run -d --name spawning-pool --restart on-failure -v hatchery:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Kerrigan -e MYSQL_DATABASE=zerglings mysql
		-> '-d' = detatched (runs in the background)
		-> '--restart on-failure' = restarts in case of error
		-> '--name spawning-pool' names the container spawning-pool
		-> '-e' = enviroment variable
		-> '-v' = uses hatchery volume
			-> check that it works: 'docker exec -it spawning-pool mysql -uroot -p'
				Password: 'Kerrigan' -> 'show databases;' and check that zerglings is there. -> 'exit'

# 10. Print the environment variables of the spawning-pool container in one command,
# to be sure that you have configured your container properly.
	docker exec spawning-pool env
		-> 'exec' runs a command in a running container

# 11. Launch a wordpress container as a background task, just for fun. The container
# should be named lair, its 80 port should be bound to the 8080 port of the virtual
# machine, and it should be able to use the spawning-pool container as a database
# service. You can try to access lair on your machine via a web browser, with the
# IP address of the virtual machine as a URL. Congratulations, you just deployed a
# functional Wordpress website in two commands!
	docker run -d --name lair -p 8080:80 --link spawning-pool:mysql \
		-e WORDPRESS_DB_HOST=spawning-pool \
		-e WORDPRESS_DB_USER=root \
		-e WORDPRESS_DB_PASSWORD=Kerrigan \
		-e WORDPRESS_DB_NAME=zerglings \
		wordpress

# 12. Launch a phpmyadmin container as a background task. It should be named roach-warden,
# its 80 port should be bound to the 8081 port of the virtual machine and it should
# be able to explore the database stored in the spawning-pool container.
	docker run -d --name roach-warden -p 8081:80 --link spawning-pool:db phpmyadmin
		Browser -> 'localhost:8081'
			username: root
			password: Kerrigan

# 13. Look up the spawning-pool container’s logs in real time without running its shell.
	docker logs -f spawning-pool
		-> 'logs' fetch the logs of a container
		-> '-f (or --follow)' follow log output

# 14. Display all the currently active containers on your machine
	docker ps

# 15. Relaunch the overlord container.
	docker restart overlord
		-> to test: docker exec -it overlord /bin/sh -c "kill 1"
			-> docker inspect -f {{.RestartCount}} overlord

# 16. Launch a container name Abathur. It will be a Python container, 2-slim version,
# its /root folder will be bound to a HOME folder on your host, and its 3000 port
# will be bound to the 3000 port of your virtual machine. You will personalize this
# container so that you can use the Flask micro-framework in its latest version. You
# will make sure that an html page displaying "Hello World" with <h1> tags can be
# served by Flask. You will test that your container is properly set up by accessing,
# via curl or a web browser, the IP address of your virtual machine on the 3000 port.
# You will also list all the necessary commands in your repository
	docker run -dit --name Abathur -v ~/:/root -p 3000:3000 python:2-slim
		-> run python2slim in the background interactive with pseudo-terminal
		-> root folder is mounted to home folder
		-> port 3000
	docker exec Abathur pip install Flask
	echo "from flask import Flask\napp = Flask(__name__)\n@app.route('/')\ndef hello_world():\n\treturn '<h1>Hello World</h1>'" > ~/app.py
		-> makes small python program to file app.py
	docker exec -e FLASK_APP=/root/app.py Abathur flask run --host=0.0.0.0 --port 3000
		-> use flask as enviroment variable to run the program

	Test that everything works:
		-> Browser: 'localhost:3000'
		-> Terminal: 'curl http://127.0.0.1:3000/'

# 17. Create a local swarm, your local machine (the school iMac) should be its manager.
# & # 18. Create a virtual machine with virtualbox, just like you did in roger-skyline-1.
# Make sure the VM’s networking works both ways with your local machine. Make
# sure you install and configure Docker to work on the VM.
	Virtualbox is the local machine in this exercise. And the school iMac slave.
	Username/password for VM:
		root/0000
		reijjo/1234
	VM:	'su -', 'apt install sudo', 'usermod -aG sudo reijjo', 'exit', 'exit', Login again, 'sudo -v'
		change ssh port: 'sudo nano /etc/ssh/sshd_config' #Port 22 -> Port 55555, 'sudo service ssh restart'
		check VM ip: 'ip a', connect to VM with MAC terminal: 'ssh reijjo@10.13.199.210 (ip from command 'ip a') -p 55555'

	Installing docker to VM:
	Update existing list of packages:
		sudo apt update
	Install prerequisite packages which let apt use packages over HTTPS:
		sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common -y
	Add the GPG key for the official Docker repository to your system:
		curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
	Add the Docker repository to APT sources:
		sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
	Update the package database with the Docker packages from the newly added repo:
		sudo apt update
	Make sure you are about to install from the Docker repo instead of the default Debian repo:
		apt-cache policy docker-ce
	Finally, install Docker:
		sudo apt install docker-ce -y
	Docker is now installed, the daemon started, and the process enabled to start on boot. Check that it's running:
		sudo systemctl status docker
	Add yourself to docker group:
		sudo usermod -aG docker reijjo
	Log out and in:
		su - reijjo
	Confirm that your user is now added to the docker group:
		id -nG
	Check that docker works:
		docker run hello-world

	17. CREATE LOCAL SWARM ON VM:
		docker swarm init --advertise-addr 10.13.199.210 (ip from 'ip a')
		docker node ls
			shows info about nodes

# 19. Turn the VM you made into a slave of the local swarm in which your local machine
# is the leader (the command to take control of the VM is not requested).
	School iMac is the slave.
	VM: 'docker swarm join-token worker' to see command to join the swarm
	iMac: 'docker swarm join --token SWMTKN-1-2uhdy82hbntwudfq01oug13i45b5rzqyzs7p2oz6p476c8e1aq-52t54qbph93xfdk0rm6mnbjgk 10.13.199.210:2377'
	VM: 'docker node ls' to see that there is the iMac is connected to swarm

# 20. Create an overlay-type internal network that you will name overmind.
	In VM because its the swarm leader:
		'docker network create -d overlay overmind'
			-> '-d' = driver to manage the Network (default "bridge")
		'docker network ls'

# 21. Launch a rabbitmq SERVICE that will be named orbital-command. You should define a specific user and password for the RabbitMQ service, they can be whatever you want. This service will be on the overmind network.
	VM:
		'docker service create -d --name orbital-command --network overmind -e RABBITMQ_DEFAULT_USER=reijjo -e RABBITMQ_DEFAULT_PASS=1234 rabbitmq'
		'docker service ls'

# 22. List all the services of the local swarm.
	VM:
		'docker service ls'

# 23. Launch a 42school/engineering-bay service in two replicas and make sure that the service works properly (see the documentation provided at hub.docker.com). This service will be named engineering-bay and will be on the overmind network.
	VM:
		'docker service create -d --name engineering-bay --replicas=2 --network overmind -e OC_USERNAME=reijjo -e OC_PASSWD=1234 42school/engineering-bay'
		'docker service ls' to see that 2/2 replicas are up. It might take a while.

# 24. Get the real-time logs of one the tasks of the engineering-bay service.
	VM:
		docker service logs -f engineering-bay | grep engineering-bay.1 (or 'grep engineering-bay.2)

# 25. ... Damn it, a group of zergs is attacking orbital-command, and shutting down the engineering-bay service won’t help at all... You must send a troup of Marines to eliminate the intruders. Launch a 42school/marine-squad in two replicas, and make sure that the service works properly (see the documentation provided at hub.docker.com). This service will be named... marines and will be on the overmind network.
	Open VM terminals just to see the logs: 'docker service logs -f engineering-bay' 'docker service logs -f marines'
	Not the VM log terminal:
		'docker service create -d --name marines --replicas=2 --network overmind -e OC_USERNAME=reijjo -e OC_PASSWD=1234 42school/marine-squad'
		'docker service ls' to see when marine-squad is working

# 26. Display all the tasks of the marines service.
	docker service ps marines

# 27. Increase the number of copies of the marines service up to twenty, because there’s never enough Marines to eliminate Zergs. (Remember to take a look at the tasks and logs of the service, you’ll see, it’s fun.)
	docker service scale -d marines=20

# 28. Force quit and delete all the services on the local swarm, in one command.
	docker service rm $(docker service ls -q)?
	docker swarm leave -f ?

# 29. Force quit and delete all the containers (whatever their status), in one command.
	docker rm -f $(docker ps -a -q)
		'-f' (--force)
		'-a' (--all) Show all containers
		'-q' (--quiet) Only display container IDs

# 30. Delete all the container images stored on your local machine machine, in one command as well.
	docker rmi -f $(docker images -a -q)

# -------------------------------------#




# 01_dockerfiles
	docker build -t <MY_CONTAINER> .
	docker run <MY_CONTAINER>

# base image
	FROM <container>
# set a directorry for the app
	WORKDIR /../../
# copy all the files to the container
	COPY . .
# install dependencies
	RUN (pip install blaablaa)
# define the port number the container should expose
	EXPOSE (5000)
# run the command
	CMD ["..."]

# difference between CMD and entrypoint:
	CMD instruction will be concatenated to the ENTRYPOINT one
		FROM alpine
		ENTRYPOINT ["echo"]
		CMD ["Hello World!"]
			-> 'Hello World!'
	We can use ENTRYPOINT to define a base command which always gets executed, and CMD to define default arguments for this command, easily overridable by passing custom args to docker container run.

# VI.1 Exercise 00 : vim/emacs
# From an alpine image you’ll add to your container your favorite text editor, vim or emacs, that will launch along with your container.
# Check what you need to the dockerfile by running alpine container: docker run -it alpine
	docker build -t eka .
	docker run -it eka

Dockerfile:
FROM alpine

RUN apk update && apk upgrade
RUN apk add vim

CMD ["vim"]

	-> :e /TAB to see that Im inside the container



# VI.2 Exercise 01 : BYOTSS
# From a debian image you will add the appropriate sources to create a TeamSpeak server, that will launch along with your container. It will be deemed valid if at least one user can connect to it and engage in a normal discussion (no far-fetched setup), so be sure to create your Dockerfile with the right options. Your program should get the sources when it builds, they cannot be in your repository. (For the smarty-pants using the official docker image of TeamSpeak is strictly FORBIDDEN, and will get you -42.)

Dockerfile:
FROM debian

EXPOSE 9987/udp
EXPOSE 10011
EXPOSE 30033
#	-> 9987/udp (for Voice)
#	-> 10011	(for Serverquery (Raw))
#	-> 30033	(for File transfer)
RUN apt update -y && apt upgrade -y && apt install -y nano software-properties-common dirmngr apt-transport-https gnupg2 ca-certificates lsb-release debian-archive-keyring wget bzip2
#	-> All kind of programs to make everything work
RUN wget https://files.teamspeak-services.com/releases/server/3.13.6/teamspeak3-server_linux_amd64-3.13.6.tar.bz2
#	-> TeamSpeak server download
RUN tar -xf ./teamspeak3-server_linux_amd64-3.13.6.tar.bz2
#	-> Running the file
RUN adduser repe --home /opt/teamspeak --shell /bin/bash --disabled-password
#	-> Creating user for the server. Home directory = /opt/teamspeak and won't have a password
RUN mv teamspeak3-server_linux_amd64/* /opt/teamspeak/
#	-> move extracted dir to /opt/teamspeak dir
RUN chown -R repe:repe /opt/teamspeak
#	-> correct permissions to the dir
USER repe
#	-> log in as TeamSpeak user
WORKDIR /opt/teamspeak
#	-> we need to be in the right dir to run the next command
RUN touch .ts3server_license_accepted
#	-> to accept the term of services
CMD ["./ts3server_minimal_runscript.sh", "start"]
#	-> command to start the server

# Build the image
	docker build -t team .
# Run the container
	docker run -p 9987:9987/udp -p 10011:10011 -p 30033:30033 team
# open TeamSpeak 3 client -> Connections -> Connect -> Server Nickname or Address & Server Password = loginname & password under the first I M P O R T A N T text on your terminal

# source: https://www.howtoforge.com/how-to-install-teamspeak-server-on-debian-11/



# VI.3 Exercise 02 : Dockerfile in a Dockerfile... in a Dockerfile ?
# You are going to create your first Dockerfile to containerize Rails applications. That’s a special configuration: this particular Dockerfile will be generic, and called in another Dockerfile, that will look something like this:
	FROM ft-rails:on-build
	EXPOSE 3000
	CMD ["rails", "s", "-b", "0.0.0.0", "-p", "3000"]
#		s = server
#		-b binds Rails to the specified IP, by default it is localhost
#		-p = port

# Your generic container should install, via a ruby container, all the necessary dependencies and gems, then copy your rails application in the /opt/app folder of your container. Docker has to install the approtiate gems when it builds, but also launch the migrations and the db population for your application. The child Dockerfile should launch the rails server (see example below). If you don’t know what commands to use, it’s high time to look at the Ruby on Rails documentation (https://rubyonrails.org/).

# Dockerfile
FROM ruby:3.1.2

RUN apt-get update && apt-get install -y nodejs sqlite3 build-essential libpq-dev npm yarn
#	-> build-essential (needed for certain gems)
#	-> libpq-dev (needed for postgres gem)
#	-> npm (install dependencies)
#	-> yarn (install dependencies)
RUN gem install rails && rails new /opt/app
#	-> install rails and creates all necessery files to /opt/app dir
ONBUILD WORKDIR /opt/app
ONBUILD COPY . /opt/app
ONBUILD RUN gem install bundler
ONBUILD RUN bundle install
ONBUILD RUN rake db:create
ONBUILD RUN rake db:migrate
#	-> ONBUILD executes later when image is used as the base for another build
#	-> COPY . /opt/app (copies all the files to /opt/app directory)
#	-> bundler installs a gem (library) that helps to manage project dependencies
#	-> bundle install all gems needed for the project
#	-> rake db:create makes a database (rake is a task runner in rails)
#	-> db:migrate changes database schema (instead of managing SQL scripts, you define database changes in a domain-specific language (DSL))

# how to run
docker build -t ft-rails:on-build .
docker build -t rails -f Dockerfile.rails .
#	-> ajaa noi komennot just tehtyyn imageen ja sen jalkeen eka Dockerfile ajaa ONBUILD komennot
docker run -p 3000:3000 rails
#	-> browser: 'localhost:3000'



# VI.4 Exercise 03: ... and bacon strips ... and bacon strips ...
# Docker can be useful to test an application that’s still being developed without polluting your libraries. You will have to design a Dockerfile that gets the development version of Gitlab - Community Edition installs (https://gitlab.com/gitlab-org/gitlab-foss) it with all the dependencies and the necessary configurations, and launches the application, all as it builds. The container will be deemed valid if you can access the web client, create users and interact via GIT with this container (HTTPS and SSH). Obviously, you are not allowed to use the official container from Gitlab, it would be a shame...

FROM debian:buster
#	debian10
EXPOSE 22 80 443

RUN RUN apt-get update && apt-get upgrade -y && \
	DEBIAN_FRONTEND=noninteractive apt-get install curl openssh-server ca-certificates perl tzdata postfix -y
#	-> curl = client url
#	-> ca-certificates = verify identity of 3rd parties and encryps data
#	-> tzdata = timezone database
RUN	curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash && \
	apt-get install gitlab-ce && \
	rm -rf /var/lib/apt/lists/*
#	-> removes extra stuff
RUN	mkdir -p /etc/gitlab/ssl && chmod 755 /etc/gitlab/ssl && \
	openssl req -new -x509 -nodes -batch -newkey rsa:4096 -subj '/CN=localhost' -keyout /etc/gitlab/ssl/localhost.key -out /etc/gitlab/ssl/localhost.crt -days 365 && \
	chmod 755 /etc/gitlab/ssl/*
#	-> mkdir -p (p = makes all folders)
#	-> req -x509 = X.509 certificate signing request (CSR) managment
#	-> -nodes = skip the option to secure certificate with passphrase
#	-> -batch = no questions will be asked
#	-> '/CN=localhost'	= cert is valid only if the request hostname matches the certificate common name
#	-> -keyout = where to place the private key we creating
#	-> -out = where to place the cert we creating

RUN echo "external_url \"https://localhost\"" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['redirect_http_to_https'] = true" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['ssl_certificate'] = \"/etc/gitlab/ssl/localhost.crt\"" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['ssl_certificate_key'] = \"/etc/gitlab/ssl/localhost.key\"" >> /etc/gitlab/gitlab.rb && \
	echo "gitlab_rails['gitlab_shell_ssh_port'] = 5252" >> /etc/gitlab/gitlab.rb && \
	echo "gitlab_rails['rake_cache_clear'] = false" >> /etc/gitlab/gitlab.rb && \
	apt-get update && apt install git -y && \
	git config --global http.sslVerify false
#	-> git config gets ridd of cert error

CMD	(/opt/gitlab/embedded/bin/runsvdir-start &) && \
	service ssh start && \
	gitlab-ctl reconfigure && \
	gitlab-ctl tail

# docker build -t lab .
# docker run -p 5252:22 -p 443:443 -p 80:80 --name lab -it --rm -e GITLAB_ROOT_EMAIL="rantarepo@hotmail.com" -e GITLAB_ROOT_PASSWORD="123456789" --privileged lab
	->privileged = grants a Docker container root capabilities to all devices on the host system

# 1.	browser-> 'localhost'
# 2.	log in-> root/123456789
# 3.	new project -> blank project -> give project name and 'Create project'
# 4.	copy SSH key-> 'tr -d '\n' < ~/.ssh/id_rsa.pub | pbcopy'
#			->copy the .ssh/id_rsa.pub to clipboard
#				-> tr	= translate or delete characters
#				-> -d '\n'	= deletes new line
#		add SHH key from your clipboard to GitLab
# 4. 	Clone using https just see that it works (git clone https://localhost/gitlab-instance-251c61a1/toimii.git httpslab)
# 5.	-> go to that folder that you just cloned (cd httpslab) and make a file (touch httpstest) and git add->git commit->git push
#		and it should be now in your gitlab
# 6.	Clone using ssh (git clone ssh://git@localhost:5252/gitlab-instance-251c61a1/toimii.git sshlab)
#		-> go to that folder that you just cloned (cd sshlab) and make a file (touch sshtest) and git add->git commit->git push
