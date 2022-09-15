FROM ruby:3.1.2

RUN apt-get update && apt-get install -y nodejs sqlite3 build-essential libpq-dev npm yarn
RUN gem install rails && rails new /opt/app

ONBUILD WORKDIR /opt/app
ONBUILD COPY . /opt/app
ONBUILD RUN gem install bundler
ONBUILD RUN bundle install
ONBUILD RUN rake db:create
ONBUILD RUN rake db:migrate


#docker build -t ft-rails:on-build .
#docker build -t rails -f Dockerfile.rails .
#docker run -p 3000:3000 rails
