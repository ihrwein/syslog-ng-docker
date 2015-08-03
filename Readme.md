# syslog-ng Docker images
This repository contains the source of the following Docker images:

* `balabit/syslog-ng`: syslog-ng is installed. It is intended to be used as a system logging service.
* `balabit/syslog-ng-dev`: development image for [`syslog-ng`](https://github.com/balabit/syslog-ng). It contains all packages to build and debug syslog-ng within the container.
* `balabit/syslog-ng-incubator-dev`: development image for [syslog-ng-incubator](https://github.com/balabit/syslog-ng-incubator). It contains all packages to build and debug syslog-ng-incubator within the container.

Later we will describe how to use each of these images in a various cases.


## `balabit/syslog-ng` image
  * Syslog-ng is installed with all of its modules (incubator modules included)
  * Within the container syslog-ng will start in foreground. This is useful because if there is some error with syslog-ng we can easily check the output console log through the `docker logs [containerID]` command
  * You can use your own `syslog-ng.conf` or fall back to use the default one

The following ports are exposed:
    * Syslog UDP: 514,
    * Syslog TCP: 601,
    * Syslog TLS: 6514

Please check the syslog-ng image tags at the official docker repository to know what image versions exist  [https://registry.hub.docker.com/u/balabit]

### Using default configuration
Assume that the following ports are not used on host machine, because they can conflict: `514`, `601`:

```bash
sudo docker run --rm -p 514:514 -p 601:601 --name syslog-ng balabit/syslog-ng:latest
```
By default syslog-ng will not print any debug messages to the console. If you want to see more debug messages you need to start the containers in this way:

```bash
sudo docker run --rm -p 514:514 -p 601:601 --name syslog-ng balabit/syslog-ng:latest -edv
```

### Using custom syslog-ng configuration
You can override the default configuration by mounting a configuration file under `/etc/syslog-ng/syslog-ng.conf`:

```bash
sudo docker run --rm -v "$PWD/syslog-ng.conf":/etc/syslog-ng/syslog-ng.conf balabit/syslog-ng:latest
```

### Reading logs from other containers
An example is used to describe how syslog-ng can read logs from other containers.

Assume that you have already running an `apache2` container which exposes its logs as a mounted volume under "/var/log/apache2/". We will read the apache logs and send them to a remote host (`1.2.3.4:514`). The example syslog-ng configuration file is stored in the current directory as `syslog-ng.conf`.

```
@version: 3.6

source s_apache {
  file("/var/log/apache2/access.log");
};

destination d_remote {
  tcp("1.2.3.4" port(514));
};

log {
  source(s_apache);
  destination(d_remote);
};
```

Now we can start syslog-ng:

```bash
sudo docker run --rm --volumes-from [containerID for apache2] -v "$PWD/syslog-ng.conf":/etc/syslog-ng/syslog-ng.conf balabit/syslog-ng:latest
```

## Entering into a container
Assume that your running container has a name "syslog-ng". In this case we can enter into this container by executing the following command:

```bash
sudo docker exec -it syslog-ng /bin/bash
```

## `balabit/syslog-ng-dev` image
This image provides a development environment to build and install syslog-ng from source. You have to clone the source
code of [syslog-ng ](https://github.com/balabit/syslog-ng.git) into a directory on your host machine then you can mount it
into the container (under `/source`).

### Building syslog-ng from source

Assume that we have cloned syslog-ng's source into the `$HOME/syslog-ng` directory. The following commands starts a container mounted with the source:

```bash
sudo docker run --rm -it -v $HOME/syslog-ng:/source balabit/syslog-ng-dev:latest /bin/bash
```

Within the image you can build and install syslog-ng:

```bash
cd /source/
./autogen.sh
mkdir build
cd build/
../configure --enable-debug --prefix=/install/syslog-ng
make
make check
make install
```

If the compilation and installation was successful you can run syslog-ng with the following command:

```bash
/install/syslog-ng -Fedv
```

## `balabit/syslog-ng-incubator-dev` image
This image provides a development environment to build and install syslog-incubator-ng from source. You have to clone the source
code of [syslog-ng-incubator](https://github.com/balabit/syslog-ng-incubator.git) into a directory on your host machine then you can mount it
into the container (under `/source`).

### Building syslog-ng-incubator from source
Assume that we have cloned syslog-ng-incubator's source into the `$HOME/syslog-ng-incubator` directory. The following commands starts a container mounted with the source:


```bash
sudo docker run --rm -it -v $HOME/syslog-ng-incubator:/source balabit/syslog-ng-incubator-dev:latest /bin/bash
```

Within the image you can build and install syslog-ng-incubator:

```bash
cd /source/
autoreconf -i
mkdir build
cd build/
../configure
make
make check
make install
```

If the compilation and installation was successful you can run syslog-ng with the following command:

```bash
/usr/sbin/syslog-ng -Fedv
```
