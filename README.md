# RoPython Cluj Meetup - 26 Feb. 2019
## Deploying your Python apps using [Dokku](http://dokku.viewdocs.io/dokku/) and [Digital Ocean](https://www.digitalocean.com/?refcode=be156fd97c45)

Presentation description:

> How to deploy your very own PaaS on a Digital Ocean droplet using Dokku (https://github.com/dokku/dokku/) a Docker powered "poor mans" mini Heroku bash script. +Demo: setup a simple Python app and deploy it using git (and if we have time automate this using Gitlab CI/CD).

1. Create a new droplet using the `One-click apps` option and choosing `Dokku 0.xx.xx on 18.04`. [This shortcut](https://cloud.digitalocean.com/droplets/new?size=s-1vcpu-2gb&region=fra1&appId=48823330&type=applications&options=install_agent&refcode=be156fd97c45) should autocomplete most of the options you need. (NOTE: while the smallest 1GB instance should be enough to get you started, I recommend going with the at least a 2GB droplet later on).

2. Add the `A` DNS record to point a (sub)domain to your new droplet IP address (ex. `ropython.example.com`).

3. Access the domain in a browser and you should be presented with the initial Dokku instance setup (as seen in this [image](./images/ropython-dokku-setup.png)). Press `Finish setup` after making changes.

4. Connect to the Dokku instance to do some additional updates:

```bash
# On the Dokku host
# Update the dokku repo keys as they are outdated on the one-click apps
$ wget -qO - https://packagecloud.io/dokku/dokku/gpgkey | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get upgrade
# Do additional setup you might need (ex.: apt-get install fail2ban)
```

5. Install [PostgreSQL](https://github.com/dokku/dokku-postgres) and [Let's Encrypt](https://github.com/dokku/dokku-letsencrypt) Dokku plugins (full plugin list here: http://dokku.viewdocs.io/dokku/community/plugins/):

```bash
# On the Dokku host
$ sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git
$ sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
$ dokku plugin
```

6. Create a `helloworld` Dokku app where we will deploy our project and a database for the app to use:

```bash
# On the Dokku host
$ dokku apps:create helloworld
# Creating the database container
$ dokku postgres:create helloworld-db
# Link the database to the app
$ dokku postgres:link helloworld-db helloworld
# Check the postgres plugin set the app environment config values for the database connection
$ dokku config helloworld
# Check that the app subdomain vhost was created and assigned to the app
$ dokku domains:report
```

7. On you local machine create a project directory and install Django in a virtual env using `pip` or `pipenv`, then:

```bash
# On your local machine
# This will create a new Django project based on a Heroku open-source template
$ django-admin startproject --template=https://github.com/heroku/heroku-django-template/archive/master.zip --name=Procfile helloworld
$ git init
$ git add -A
$ git commit -m "Initial commit"
```

8. In order to deploy the above project we have to add the Dokku host as a git remote:

```bash
# On your local machine
# The user must be `dokku` otherwise it won't work. Also, the string after `:` must be the name of the Dokku app created earlier
$ git remote add dokku dokku@ropython.example.com:helloworld
# Deploy the app
$ git push dokku master
# We should be able to access the project at http://ropython.example.com/ at this point
```

9. As a final step, we can get the project to be served over HTTPS using a [Let's Encrypt](https://letsencrypt.org/) certificate:

```bash
# On the Dokku host
$ dokku letsencrypt helloworld
# We should be able to access the project at https://ropython.example.com/ at this point
```

10. Profit!
