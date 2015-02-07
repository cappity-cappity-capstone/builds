# Builds

This repository coordinates the services for our capstone so that they can "talk" to one another over HTTP.
This repos assumes that the `ui` and `backend` directories are both on your hard drive, located in the same folder as this one.
If that's not the case, please set `UI_LOCATION` and `BACKEND_LOCATION` to their actual paths.
Here's a quick guide on getting set up:

* Install [`boot2docker`](https://docs.docker.com/installation/mac/) and ensure it's running
* `bundle install` in this directory
* `bundle exec rake up` Will start and build your server
* Get your boot2docker ip via `$ boot2docker ip`
* Hit the boot2docker ip on port 1337 in your browser

To see all of the possible commands, run `bundle exec rake -T`.
