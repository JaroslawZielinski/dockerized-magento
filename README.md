# Dockerized - Magento Community Edition 1.9.x

A dockerized version of "Magento Community Edition 1.9.x"

## Requirements

If you are on Linux you should install

- [docker](http://docs.docker.com/compose/install/#install-docker) and
- [docker-compose (formerly known as fig)](http://docs.docker.com/compose/install/#install-compose)

If you are running on [Mac OS](https://docs.docker.com/engine/installation/mac/) or [Windows](https://docs.docker.com/engine/installation/windows/) you can install the [Docker Toolbox](https://docs.docker.com/engine/installation/mac/) which contains docker, docker-compose and docker-machine.

## Preparations

The web-server will be bound to your local ports 80 and 443. In order to access the shop you must add a hosts file entry for `workbench-magento.local`.

### For Linux Users

In order to access the shop you must add the domain name "workbench-magento.local" to your hosts file (`/etc/hosts`).
If you are using docker **natively** you can use this command:

```bash
sudo su
echo "127.0.0.1    workbench-magento.local" >> /etc/hosts
```

### For Mac Users

If you are using [docker-machine](https://github.com/docker/machine) you need to use the IP address of your virtual machine running docker:

```bash
docker-machine ls
docker-machine ip <name-of-your-docker-machine>
```

```bash
echo "192.168.99.100    workbench-magento.local" >> /etc/hosts
```

**docker-machine performance**

For faster sync between your host system and the docker machine image I recommend that you activate NFS support for virtualbox: [github.com/adlogix/docker-machine-nfs](https://github.com/adlogix/docker-machine-nfs)

```bash
docker-machine create --driver virtualbox nfsbox
docker-machine-nfs nfsbox
```

And to fix issues with filesystem permissions you can modify your NFS exports configuration:

1. open `/etc/exports` and replace `-mapall=$uid:$gid` with `-maproot=0`
2. `sudo nfsd restart`

Thanks to [René Penner](https://github.com/renepenner/magento-docker-boilerplate) for figuring that out.

### For Windows Users

I suppose it will work on Windows, but I have not tested it. And I suspect that the performance will not be great due to the slow file-sharing protocol between the Windows hosts and the VirtualBox VM.

## Installation

1. Make sure you have docker and docker-compose on your system
2. Clone the repository
3. Run this command to build php7.2 image `docker build -t dockerized-image-php docker-images/php/`
4. Start the projects using `./magento start` or `docker-compose up`

```bash
git clone https://github.com/JaroslawZielinski/WorkbenchMagento.git && cd WorkbenchMagento
docker build -t dockerized-image-php docker-images/php/
./magento start
```

During the first start of the project **docker-compose** will

1. first **build** all docker-images referenced in the [docker-compose.yml](docker-compose.yml)
2. then **start** the containers
3. and **trigger the installer** which will
	- [install magento](docker-images/installer/bin/install.sh) and all modules that are referenced in the [composer.json](composer.json) using `composer` into the web folder
	- download the [Magento Demo Store Sample Data](http://www.magentocommerce.com/knowledge-base/entry/installing-the-sample-data-for-magento)
	- copy the files to the magento-root
	- import the sample database
	- and finally reindex all indices

Once the installation is finished the installer will print the URL and the credentials for the backend to the installer log:

```
...
installer_1      | phpMyAdmin: http://workbench-magento.local:8080
installer_1      |  - Username: root
installer_1      |  - Password: pw
installer_1      |
installer_1      | Backend: http://workbench-magento.local/admin
installer_1      |  - Username: admin
installer_1      |  - Password: password123
installer_1      |
installer_1      | Frontend: http://workbench-magento.local/

```

[![Animation: Installation and first project start](documentation/installation-and-first-start-animation.gif)](https://s3.amazonaws.com/andreaskoch/dockerized-magento/installation/Dockerized-Magento-Installation-Linux-no-sound.mp4)

**Note**: The build process and the installation process will take a while if you start the project for the first time. After that, starting and stoping the project will be a matter of seconds.

## Usage

You can control the project using the built-in `magento`-script which is basically just a **wrapper for docker and docker-compose** that offers some **convenience features**:

```bash
./magento <action>
```

**Available Actons**

- **start**: Starts the docker containers (and triggers the installation if magento is not yet installed)
- **stop**: Stops all docker containers
- **restart**: Restarts all docker containers and flushes the cache
- **status**: Prints the status of all docker containers
- **stats**: Displays live resource usage statistics of all containers
- **magerun**: Executes magerun in the magento root directory
- **composer**: Executes composer in the magento root directory
- **enter**: Enters the bash of a given container type (e.g. php, mysql, ...)
- **destroy**: Stops all containers and removes all data

**Note**: The `magento`-script is just a small wrapper around `docker-compose`. You can just use [docker-compose](https://docs.docker.com/compose/) directly.

## If you want to configure your Magento using "Workbench_PlainTable" module do following steps:
- Create your own admin user
```bash
./magento magerun admin:user:create
``` 
- Remove Demo notice
```bash
./magento magerun design:demo-notice --global
```
- Remove notices in admin panel
    - System->Notifications (with mass action select all, Mark as read action and submit)
    - Enable Formkey validation on checkout (System->Configuration->ADVANCED->Admin [Security]: Enable Form Key Validation On Checkout - Yes)
    - ReindexAll (System->Index Management Select All Reindex Data action and submit)
- Setup Website Workbench as default (System->Manage Stores, Find Workbench click on it and set as default, Save Website)
    - Alternative way:
        ```bash
      ./magento enter php
      ``` 
      ```bash
      php web/shell/workbench/setup-website-workbench-as-default/shell.php --start
      ``` 
- Configure and Save
    - System->Configuration->ADVANCED->Admin [Startup Page]: Configuration
    - System->Configuration->ADVANCED->System [Cron]: Enable "run now": Yes
    - System->Configuration->ADVANCED->Developer [Log Settings]: Enabled: Yes
    - System->Configuration->GENERAL->General [Locale Options]: Timezone: Central European Standard Time (Europe/Warsaw)
    - System->Configuration->GENERAL->General [Locale Options]: Locale: English (United Kingdom)
    - System->Configuration->GENERAL->Web->[Default Pages]: CMS HOME Page : Workbench
    - System->Configuration->GENERAL->Design [Package]:	Current Package Name: workbench
    - System->Configuration->GENERAL->Design [HTML Head]: Default Title: Magento Workbench
    - System->Configuration->GENERAL->Design [Footer]: Copyright:
```html
<div class="pt-2 lgrey">
<div class="text-center py-2">Github page:
                        <a href="https://github.com/JaroslawZielinski" target="_blank"> JaroslawZielinski</a>
                    </div>
                    <div class="text-center py-2">LinkedIn page:
                        <a href="https://linkedin.com/in/jzielinski82" target="_blank"> JaroslawZielinski</a>
                    </div>
</div>
```

- Switch off cache (developer mode) and clear
    - System->Cache Management 
        - Select all Disable action and Submit
        - Select all Refresh action and Submit
        - Flush Magento Cache - click button
        - Flush JavaScript/Css Cache - click button
- Refresh Front Page of the project
 
## Components

### Overview

The dockerized Magento project consists of the following components:

- **[docker images](docker-images)**
  1. a [PHP](docker-images/php/Dockerfile) image
  2. a [Nginx](docker-images/nginx/Dockerfile) web server image
  3. a [Solr](docker-images/solr/Dockerfile) search server
  4. a standard [MySQL](https://hub.docker.com/_/mysql/) database server image
  5. multiple instances of the standard [Redis](https://hub.docker.com/_/redis/) docker image
	6. a standard [phpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) image that allows you to access the database on port 8080
  7. and a [Installer](docker-images/installer/Dockerfile) image which contains all tools for installing the project from scratch using an [install script](docker-images/installer/bin/install.sh)
- a **[shell script](magento)** for controlling the project: [`./magento <action>`](magento)
- a [composer-file](composer.json) for managing the **Magento modules**
- and the [docker-compose.yml](docker-compose.yml)-file which connects all components

The component-diagram should give you a general idea* how all components of the "dockerized Magento" project are connected:

[![Dockerized Magento: Component Diagram](documentation/dockerized-magento-component-diagram.png)](documentation/dockerized-magento-component-diagram.svg)

`*` The diagram is just an attempt to visualize the dependencies between the different components. You can get the complete picture by studying the docker-compose file:  [docker-compose.yml](docker-compose.yml)

Even though the setup might seem complex, the usage is thanks to docker really easy.

If you are interested in a **guide on how to dockerize a Magento** shop yourself you can have a look at a blog-post of mine: [Dockerizing  Magento](https://andykdocs.de/development/Docker/Dockerize-Magento) on [AndyK Docs](https://andykdocs.de)

## Custom Configuration

All parameters of the Magento installation are defined via environment variables that are set in the [docker-compose.yml](docker-compose.yml) file - if you want to tailor the Magento Shop installation to your needs you can do so **by modifying the environment variables** before you start the project.

If you have started the shop before you must **repeat the installation process** in order to apply changes:

1. Modify the parameters in the `docker-compose.yml`
2. Shutdown the containers and remove all data (`./magento destroy`)
3. Start the containers again (`./magento start`)

### Changing the domain name

I set the default domain name to `workbench-magento.local`. To change the domain name replace `workbench-magento.local` with `your-domain.tld` in the `docker-compose.yml` file:

```yaml
installer:
  environment:
    DOMAIN: workbench-magento.local
```

### Using a different SSL certificate

By default I chose a dummy certificate ([config/ssl/cert.pem](config/ssl/cert.pem)).
If you want to use a different certificate you can just override the key and cert with your own certificates.

### Adapt Magento Installation Parameters

If you want to install Magento using your own admin-user or change the password, email-adreess or name you can change the environment variable of the **installer** that begin with `ADMIN_`:

- `ADMIN_USERNAME`: The username of the admin user
- `ADMIN_FIRSTNAME`: The first name of the admin user
- `ADMIN_LASTNAME`: The last name of the admin user
- `ADMIN_PASSWORD`: The password for the admin user
- `ADMIN_EMAIL`: The email address of the admin user (**Note**: Make sure it has a valid syntax. Otherwise Magento will not install.)
- `ADMIN_FRONTNAME`: The name of the backend route (e.g. `http://workbench-magento.local/admin`)

```yaml
installer:
  build: docker-images/installer
  environment:
		ADMIN_USERNAME: admin
		ADMIN_FIRSTNAME: Admin
		ADMIN_LASTNAME: Inistrator
		ADMIN_PASSWORD: password123
		ADMIN_FRONTNAME: admin
		ADMIN_EMAIL: admin@example.com
```

### Change the MySQL Root User Password

I chose a very weak passwords for the MySQL root-user. You can change it by modifying the respective environment variables for the **mysql-container** ... and **installer** because otherwise the installation will fail:

```yaml
installer:
  environment:
    MYSQL_PASSWORD: <your-mysql-root-user-password>
mysql:
  environment:
    MYSQL_ROOT_PASSWORD: <your-mysql-root-user-password>
```
