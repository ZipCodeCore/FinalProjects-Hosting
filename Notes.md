# Notes on what we did

In order to HOST the final projects of 9.1, we created a AWS instance to run them all and give them last minute experience with Docker.
Maybe we should Kubernetes next time? (Or both?)

## Setup

- Created a 4-cpu, 16GB memory, Lightsail instance.
- Tied it to a static IPv4 address. 
- Setup to allow firewall to let in 80 and 443.
- Used/Installed `caddy` as a reverse-proxy web server for the student projects.
  - it's very simple to set up
- Installed `docker` to support the projects
- I had to install nd compile the latest `golang` to get `caddy` to compile.
  - There is no binary image of caddy for Amazon Linux (which what I installed when creating instance)


### Caddyfile for the reverse-proxy

This file mapped the names like `project1name.zipcode.rocks` to `localhost:8086`.
Several groups had to sanitize their code because they had CORS problems (needed to fix a CORS pattern).
Several groups had to sanitize source to remove `localhost:8080` references, after we assigned a new project port.

Had to `caddy run` in the directory where the `Caddyfile` was.

`/etc/caddy/Caddyfile` 

```
{
	debug
	log {
		output file /var/log/access.log
	}
}

xo.zipcode.rocks {
	root * /home/ec2-user/zcw

	file_server {
		index index.html
	}
}

pp.zipcode.rocks paperplane.zipcode.rocks {
        reverse_proxy localhost:8086
}
klasschat.zipcode.rocks {
        reverse_proxy localhost:8087
}
newscraft.zipcode.rocks {
        reverse_proxy localhost:8088
}
duryou.zipcode.rocks {
        reverse_proxy localhost:8089
}
zipflix.zipcode.rocks {
        reverse_proxy localhost:8090
}
```
### Changes to DNS for `xo.zipcode.rocks`

We made a `A` record for each project.
Created a `project-name.zipcode.rocks` record that pointed to the host's IPv4 address.
CNAMEs wouldn't work.
One for each project.

```
    A	xo	18.221.94.201
# and then for each project
	A	paperplane	18.221.94.201
    ...
```

(`18.221.94.201` was the IPv4 static address of the instance.)

### Each project's PORT assignments

Each team had a reverse-proxy port assigned. 
We used [8086, 8087, 8088, 8089, 8090].
This allowed us to map the internal ports to the names we put into the `.zipcode.rocks` domain.

We also assigned special ports for the DATABASES.
With docker, you cannot have two MySQLs both running on default port 3306.
So we assigned all the MySQL and Postgres servers a different port so they wouldn't collide.
All the student's projects had to change the PORT for the DB _everywhere in their project source._
(Not just the ports in the app.yml files in the jhipster projects. This was counter-intuitive, but `app.yml` was not enough.)

## Added for 9242 - Group DB server

added a mysql on 3388
there is a strong password on the mysql root account. ask an instructor.

create a spot for a group:

_inside mysql as root_
```
create database blahblah;
CREATE USER 'blahblah_user' IDENTIFIED BY 'blahblah_password';
GRANT ALL PRIVILEGES ON blahblah.* TO 'blahblah';
FLUSH PRIVILEGES;
```

and to test

```
mysql -u some_user -h machinename.zipcode.rocks -P 3388 -p some_db
```

for postgres:

```
CREATE USER davide WITH PASSWORD 'jw8s0F4';
GRANT ALL PRIVILEGES ON DATABASE "someDB" to davide;
```

### to start the shared mysql server...

```
docker run -d --name zcw_942 -e MYSQL_ROOT_PASSWORD=somexstrongxpassword -p 3388:3306 mysql
```

## Student-Group access

Used standard PEM file to grant students access using a simple shell script.

Distributed a TGZ of the shell script `xo` and it's associated .PEM private key file.

```
# xo contents
ssh -i ~/.ssh/LightsailDefaultKey-us-east-2.pem {ec2-user,ubuntu}@xx.xx.xx.xx
```

You PEM file name will be different.

Students need to place PEM file into their `.ssh/` folder, making sure that the chmod is `0400`.
Then, place the `xo` file into `~/bin` making sure that `~/bin` is on the shell PATH...

```
export PATH=$HOME/bin:$PATH
```
### Before first deployment

Each group had to get their project running on a DEV machine on their assigned PORT(s) within a DEV based docker container.

### Each time they make GIT changes...

```
git pull

npm run java:docker

docker-compose -f src/main/docker/app.yml up -d
```

The NPM command did a build and created the jar.
The `docker-compose` did the launch of the tasks needed for the project. The `-d` brings the project up in daemon mode.

### Swap

After I moved the files on 29 Sept, I also increased swap on the xo host. 
I used this [How To Add Swap Space on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04) article to make it happen.

