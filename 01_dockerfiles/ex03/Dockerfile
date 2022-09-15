FROM debian:buster

EXPOSE 22 80 433

RUN apt-get update && apt-get upgrade -y && \
	DEBIAN_FRONTEND=noninteractive apt-get install curl openssh-server ca-certificates perl tzdata postfix -y

RUN	curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash && \
	apt-get install gitlab-ce && \
	rm -rf /var/lib/apt/lists/*

RUN	mkdir -p /etc/gitlab/ssl && chmod 755 /etc/gitlab/ssl && \
	openssl req -new -x509 -nodes -batch -newkey rsa:4096 -subj '/CN=localhost' -keyout /etc/gitlab/ssl/localhost.key -out /etc/gitlab/ssl/localhost.crt -days 365 && \
	chmod 755 /etc/gitlab/ssl/*

RUN echo "external_url \"https://localhost\"" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['redirect_http_to_https'] = true" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['ssl_certificate'] = \"/etc/gitlab/ssl/localhost.crt\"" >> /etc/gitlab/gitlab.rb && \
	echo "nginx['ssl_certificate_key'] = \"/etc/gitlab/ssl/localhost.key\"" >> /etc/gitlab/gitlab.rb && \
	echo "gitlab_rails['gitlab_shell_ssh_port'] = 5252" >> /etc/gitlab/gitlab.rb && \
	echo "gitlab_rails['rake_cache_clear'] = false" >> /etc/gitlab/gitlab.rb && \
	apt-get update && apt install git -y && \
	git config --global http.sslVerify false

CMD	(/opt/gitlab/embedded/bin/runsvdir-start &) && \
	service ssh start && \
	gitlab-ctl reconfigure && \
	gitlab-ctl tail

#docker build -t lab .
#docker run -p 5252:22 -p 443:443 -p 80:80 --name lab -it --rm -e GITLAB_ROOT_EMAIL="rantarepo@hotmail.com" -e GITLAB_ROOT_PASSWORD="123456789" --privileged lab

# 1.	browser-> 'localhost'
# 2.	log in-> root/123456789
# 3.	new project -> blank project -> give project name and 'Create project'
# 4.	copy SSH key-> 'tr -d '\n' < ~/.ssh/id_rsa.pub | pbcopy'
#			->copy the .ssh/id_rsa.pub to clipboard
#			-> tr	= translate or delete characters
#			-> -d '\n'	= deletes new line
#		add SHH key from your clipboard to GitLab
# 4. 	Clone using https just see that it works (git clone https://localhost/gitlab-instance-251c61a1/toimii.git httpslab)
# 5.	-> go to that folder that you just cloned (cd httpslab) and make a file (touch httpstest) and git add->git commit->git push
#		and it should be now in your gitlab
# 6.	Clone using ssh (git clone ssh://git@localhost:5252/gitlab-instance-251c61a1/toimii.git sshlab)
#		-> go to that folder that you just cloned (cd sshlab) and make a file (touch sshtest) and git add->git commit->git push
