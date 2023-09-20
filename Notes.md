# Notes on what we did

In order to 
## Setup

- Created a 4-cpu, 16GB memory, Lightsail instance.
- Tied it to a static IPv4 address. 
- Setup to allow firewall to let in 80 and 443.
- Used/Installed `caddy` as a reverse-proxy for the student projects.
- Installed `docker` to support the projects

### Caddyfile for the reverse-proxy

This file mapped the names like `project1name.zipcode.rocks` to `localhost:8086`.
Several groups had to sanitize their code because they had CORS problems (needed to fix a CORS pattern).
Several groups had to sanitize source to remove `localhost:8080` references, after we assigned a new project port.

### Changes to DNS for `xo.zipcode.rocks`

We made a `A` record for each project.
Created a `project1name.zipcode.rocks` record that pointed to the host's IPv4 address.
CNAMEs wouldn't work.
One for each project.

### Each project's PORT assignments

Each team had a reverse-proxy port assigned. 
We used [8086, 8087, 8088, 8089, 8090].
This allowed us to map the internal ports to the names we put into the `.zipcode.rocks` domain.

We also assigned special ports for the DATABASES.
With docker, you cannot have two MySQLs both running on default port 3306.
So we assigned all the MySQL and Postgres servers a different port so they wouldn't collide.
All the student's projects had to change the PORT for the DB _everywhere in their project source._
(Not just the ports in the app.ini files in the jhipster projects. This was counter-intuitive, but `app.ini` was not enough.)

## Student-Group access

Used standard PEM file to grant students access using a simple shell script.

Distributed a TGZ of the shell script `xo` and it's associated .PEM private key file.

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


```

