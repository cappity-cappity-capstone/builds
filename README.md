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
WHAT DO I DO?

## Docker
### Install boot2docker

```
brew install boot2docker
```

```
git clone git@github.com:cappity-cappity-capstone/ui.git
git clone git@github.com:cappity-cappity-capstone/auth.git
git clone git@github.com:cappity-cappity-capstone/backend.git
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
```
docker build -t ui .
docker build -t auth .
docker build -t backend .
```

### Run Docker
```
docker run ui
docker run auth
docker run backend
```

### View Docker Instances Running
```
docker ps
```

### View Docker IP Addresses
```
docker ips
```

### SSH into Container
```
docker exec -it CONTAINER_ID bash
```
