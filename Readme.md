# A Docker image for Nikola with Org mode

This is a [Docker](https://www.docker.com/) image for the
[Nikola](https://getnikola.com/) static web site generator, that includes the
plugins and dependencies necessary to author sites in
[Emacs](https://www.gnu.org/software/emacs/) with [Org
Mode](https://orgmode.org/)

This image is based on the Nikola image provided by Olaf Meuwissen (see
[here](https://gitlab.com/paddy-hack/nikola)) and adds emacs, git, and the
Nikola [orgmode plugin](https://plugins.getnikola.com/v7/orgmode/)


You can pull the image from the docker hub registry with

```
docker pull abellmann/nikola-orgmode
```

Please note that the image does not have a default command set.  Also
note that the image uses `/bin/sh` rather than `/bin/bash`.

## Image Versions

The above command will pull the `latest` image. Images tagged with the upstream
version used in the build are available as well. An image tagged with 7.8.7 was
built using Nikola version 7.8.7. Have a look at the [docker hub
registry](https://hub.docker.com/r/abellmann/nikola-orgmode/tags/) to see what's
available.

## Using the Image

I am using this Image on a Windows 7 system with cygwin and docker toolbox
installed. If your setup differs, your usage may differ. Please also have a look
at the [documentation of the base image](https://gitlab.com/paddy-hack/nikola)
for instructions on MacOs

*Important*: The docker toolbox only replicates the User Home directory. This
means, that your Nikola site should be checked out into a directory below the
User Directory.


Assuming your Nikola site lives in the current directory and your current
directory is inside your user home directory, you can use something
like

``` sh
docker run -it -v $PWD:/site -w /site abellmann/nikola-orgmode /bin/sh
```

to start a container. Your site will be volume mounted on `/site`,
which is also the working directory inside the container. 

On Windows, the Nikola database file (.doit.db) cannot be in a mounted directory
or the `nikola build` will fail. So the first time you start the container, you
should create a symlink for this file.

You will need permissions on your Windows Hosts to create symbolic links. A guide on 
setting up permission for symlinks with Virtualbox is [here](https://stackoverflow.com/questions/17895256/creating-symbolic-link-protocol-error/39218200?sgp=2#39218200)   

``` sh
>pwd
/site
>ln -s /tmp/.doit.db
```

Once in the container, you can use the regular `nikola` commands to
maintain your site interactively. Of course, you can also start up a
container for a one-off `nikola` command.  For example


``` sh
docker run -it -v $PWD:/site -w /site abellmann/nikola-orgmode nikola check -l
```


## Serving your Site locally

Want to proofread your site?  No problem!  You can use `nikola`'s
`serve` command for this.  It serves your site on port 8000 by
default but you have to expose that container port in order to make it
accessible from the outside.  That goes like this (note the -p
option)

```
docker run -it -v $PWD:/site -w /site abellmann/nikola-orgmode -p 8000:8000 nikola serve
```

Now you can browse your site at http://192.168.99.100:8000 (`192.168.99.100` is
the default ip of your docker machine. Change the ip address accordingly , if
your docker machine has a different ip address).

Use `Ctrl+C` to shut down the container process.

## Using docker-compose

You can use docker compose to simplify the startup of the container. Just create
a `docker-compose.yml` in your Nikola sites root directory similar to the following:

``` yml
version: '3'
services:
  nikola-orgmode:
    image: abellmann/nikola-orgmode
    ports:
      - 8000:8000
    volumes:
      - .:/site
    # open interactive shell
    stdin_open: true
    tty: true
    # default directory..
    working_dir: /site
```

You can then start your container in your site directory with

``` sh
docker-compose run --service-ports nikola-orgmode
```

This will start an interactive shell. You can run `nikola serve` in this shell
and see your site at http://192.168.99.100:8000





