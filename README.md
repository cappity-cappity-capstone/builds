# Builds

This repository coordinates the services for our capstone so that they can "talk" to one another over HTTP.
This repos assumes that the `ui` and `backend` directories are both on your hard drive, located in the same folder as this one.
If that's not the case, please set `UI_LOCATION` and `BACKEND_LOCATION` to their actual paths.

# Setup

Here's how to get the various projects up and running on a new machine.

1. [Backend Local](#user-content-backend)
2. [UI Local](#user-content-ui)
3. [Auth Local](#user-content-auth)
4. [Docker](#user-content-docker)
5. [SSH](#user-content-ssh)

## Backend
### Installing the Backend Dependencies

1. Install Ruby >= 2.1.5, ruby-gems, bundler, gems

	```
	git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
	git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
	rbenv install 2.1.5
	rbenv rehash
	gem install bundler
	bundle install
	```
2. Install Redis

	You're on your own here, brew install redis, pacman -S redis, build from source, I don't really care.

### Running Backend
1. Run Redis

	```
	redis-server
	```
2. Run Clockwork

	```
	bundle exec rake clockwork
	```
3. Run Resque

	```
	bundle exec rake resque
	```
4. Run Webserver

	```
	bundle exec rake web
	```

## UI

### Installing the UI Dependencies
1. Install ruby, ruby-gems, bundler

	```
	git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
	git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
	rbenv install 2.1.5
	rbenv rehash
	gem install bundler
	```
2. Install nodejs

	Use the package manager of your choice or the binary from [nodejs.org](http://nodejs.org/).

3. Run init.sh as sudo

	This script should run fine on OSX and on most Linux distributions, but I think DanO was having problems on Ubuntu because the command to get the non-sudo user doesn't work. In that case, just open the script and run the first set of commands as sudo and the rest a normal user.

	```
	sudo ./init.sh
	```

### Running UI

#### Build

Build our React and SASS into JS/CSS with `gulp build`

#### Run Server

Run the server with `gulp`; it should open a window in your default browser automagically.

## Auth

#### Installing the Auth Dependencies
1. Install MySQL
2. Install Ruby >= 2.1.5, ruby-gems, bundler, gems

	```
	git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
	git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
	rbenv install 2.1.5
	rbenv rehash
	gem install bundler
	bundle install
	```
#### Create DB
```
bundle exec rake db:create
bundle exec rake db:migrate
```

#### Create a test user

```
curl --verbose \
       --request POST \
       --data @- \
       http://localhost:4567/auth/users/ <<EOF
{
  "name": "Jonny",
  "email": "jonny@gmail.com",
  "password": "trustno1"
}
EOF
```

#### Running Auth

Run Webserver

	```
	bundle exec rake web
	```

## Docker
### Install boot2docker

```
brew install boot2docker
```

These repos should all be checked out to the same directory

> Ex: ~/capstone/builds ~/capstone/ui ...

```
git clone git@github.com:cappity-cappity-capstone/builds.git
git clone git@github.com:cappity-cappity-capstone/ui.git
git clone git@github.com:cappity-cappity-capstone/auth.git
git clone git@github.com:cappity-cappity-capstone/backend.git
git clone git@github.com:cappity-cappity-capstone/backend-nginx.git
```

### Run boot2docker
```
boot2docker up
```

### Setup Env Variables
```
eval $(boot2docker shellinit)
```


### Build Docker
From the builds repo:

```
bundle exec rake build
```

OR each repo seperately with:

```
bundle exec rake ui:build
bundle exec rake auth:build
bundle exec rake backend:build
bundle exec rake backend-nginx:build
```

### Run Auth and Frontend together
```
docker run -d -e APP_ENV=production -e RACK_ENV=production auth
docker run -d -p 1337:1337 --link auth:auth ui
```

### Run Backend and Nginx together
```
docker run -d redis # This will take a minute to pull if it hasn't already been pulled
docker run -d -e APP_ENV=production -e RACK_ENV=production -e REDIS_HOST=redis --link redis:redis backend bundle exec rake web
docker run -d -e APP_ENV=production -e RACK_ENV=production -e REDIS_HOST=redis --link redis:redis backend bundle exec rake resque
docker run -d -e APP_ENV=production -e RACK_ENV=production -e REDIS_HOST=redis --link redis:redis backend bundle exec rake clockwork
docker run -d -p 4567:80 --link web:backend backend-nginx
```

### View Docker Instances Running
```
docker ps
```

### View Docker IP Addresses
```
boot2docker ips
```

### SSH into Container when running
```
docker exec -it CONTAINER_ID bash
```

### See logs from container
```
docker logs CONTAINER_ID
```

### Getting docker images ready for deploy:
```
docker save backend:latest backend-nginx:latest | gzip -c > backend_images.tar.gz
docker save auth:latest ui:latest | gzip -c > auth_images.tar.gz
```

## SSH

### AWS Instance

#### Make sure SSH key is read+write by yourself only (600)
```
chmod 600 ~/path/to/ssh-key.pem
```

#### Get the ssh key into your agent
```
ssh-add ~/path/to/ssh-key.pem
```

#### SSH onto instance

Since the ssh key is in your agent, this should work:

```
ssh ubuntu@cappitycappitycapstone.com
```

If not, try this:

```
ssh -i ~/path/to/ssh-key.pem ubuntu@cappitycappitycapstone.com
```

#### Copying images to instance

From local:

```
scp auth_images.tar.gz ubuntu@cappitycappitycapstone.com:~/.
```

From instance:

```
cat auth_images.tar.gz | sudo docker load
```

#### Restarting fig
```
screen -r

# force kill the current process running by spamming Ctrl+c

sudo docker ps -a -q | xargs docker rm -f # Removes all old containers

sudo -E fig up

# Ctrl-a d to detach from screen leaving process running
```

### Zotac

#### Make sure SSH key is read+write by yourself only (600)
```
chmod 600 ~/path/to/ssh-key.pem
```

#### Get the ssh key into your agent
```
ssh-add ~/path/to/ssh-key.pem
```

#### SSH onto box
Find IP from router or Auth server and substitute for 192.168.1.160

Since the ssh key is in your agent, this should work:

```
ssh capstone@192.168.1.160
```

If not, try this:

```
ssh -i ~/path/to/ssh-key.pem capstone@192.168.1.160
```

#### Copy images to Zotac

From local:

```
scp backend_images.tar.gz capstone@192.168.1.160:~/.
```

From Zotac:

```
cat backend_images.tar.gz | sudo docker load
```

#### Restart service
```
sudo systemctl restart ccs.service
```
