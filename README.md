![](https://user-images.githubusercontent.com/63307779/81802868-240f7280-9517-11ea-92a6-546ac97988fe.png)
General Information
===================

This repo provides the basic for setting up **CAMPR On-Premise** on your local network or own servers.

Campr On Premise is the on premise version of Campr. It consists of a single Symfony application, a mysql server and a redis cache.

The Symfony application handles each requested domain as a separate environment with a separate database.


The basics consist of:
- a `docker-compose.yml` file with everything preconfigured
- a few configuration files required by the applications and the services that they relly on

This guide assumes you are a bit familiar with `docker` and `docker-compose`. Upon request and negotiation, assistance can be provided for setting the project up.

For convenience, a `mailcatcher` service is preconfigured so that you can set your installation up easily. Please see the [Customizations](#customizations) section to find out how you can change the mailer configuration.

# Contributors

<a href="https://github.com/CamprGmbH/campr/graphs/contributors">
  <img src="https://contributors-img.web.app/image?repo=CamprGmbH/campr" />
</a>

Installation Requirement
========================
CAMPR can be installed on any Unix, Linux, Windows or MacOS server.


The separate installation of `MySQL`, `apache`, `php` and `redis` is not necessary, as all these installations are done via the installation script, CAMPR runs on your server in a docker container.

Adjustments to your local domain, setting up the workspaces, the printing function and using your own mail server are explained in the following.


Setup Git
  ====
The installation of git is exemplary under ubuntu as follows:
```
sudo apt update
sudo apt install git
```
confirm that you have installed Git correctly by running the following command:
```
git --version
```
```
Output
git version 2.17.1
```

Navigate to the directory where you want to create the repository. Next clone the campr on-premise repository, it's located here: https://github.com/CamprGmbH/on-premise.git.
```
git clone https://github.com/CamprGmbH/on-premise.git
```

Setup Docker
=====
Docker can be downloaded and installed from the appropriate download page for your system. https://www.docker.com/products/docker-desktop. For the installation under e.g. ubuntu execute the following command:

```
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update -y
sudo apt-get install docker-ce -y
```

Test the installation and run the following sample image:
```
sudo docker run hello-world
```
The following message indicates that Docker is working correctly:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:3e1764d0f546ceac4565547df2ac4907fe46f007ea229fd7ef2718514bcec35d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

To start Docker automatically at boot time, execute the following command:
```
sudo systemctl status docker
```

Create a user/password then, this user will be used for the first login to CAMPR.
```
docker-compose exec workspaces bash
bin/console tss:app:user-create yourname@youremail.xxx admin admin --role=ROLE_ADMIN
```

Go to `docker-compose.yml` in your local directory and run
```
docker-compose up -d
```
If your installation was successful, you should see the following output on the console:
```
Starting on-premise_workspace_1   ... done
Starting on-premise_redis_1       ... done
Starting on-premise_mailcatcher_1 ... done
Starting on-premise_mysql_1       ... done  
```

Configure domains
=======

The default domain we are using is `campr.local`:
- `campr.local` is the part of the app where you can login and create workspaces

- `*.campr.local` is the workspaces part of the application, where each workspace can be worked on as a separate environment

As you can see we're using `campr.local` as a base domain, which means you will have to configure these domains in your `/etc/hosts` file manually. An example would look like this:

```
127.0.0.1   campr.local
127.0.0.1   workspace1.campr.local
```

Unfortunately, wildcard domains are not supported out of the box so you wil have to add each created workspace as an entry there if you plan on keeping the domains as they are.

Configure pdf-printing
======================

In order for PDF printing to work, the PDF printing service makes requests to the API to retrieve data. Normally, this isn't an issue as we relly on external DNS services, however, in the on-premise environment, the PDF service will try to access the workspace's API vie it's domain (e.g.: `workspace1.campr.local`) and this will result in an error.

To make the pdf printing work, you will have to add the current workspace hostname as an extra host inside your local
`docker-compose.yml` file:

e.g.:
```
# ...
services:
  workspaces:
    # ...
    extra_hosts:
    - "workspace1.dev.campr.biz:127.0.0.1"
# ...
```

Customizing the domains will be detailed in the [Customizations](#customizations) section.



Customizations
==============

Changing the domain from `campr.local`
--------------------------------------

Change every occurence of `campr.local` with your new domain in the following files:

- `config/workspaces/backend/app/config/parameters.yml`
- `docker-compose.yml`

Changing mail service
---------------------

In the `config/workspaces/backend/app/config/parameters.yml` file change this:

```
    mailer_transport: smtp
    mailer_host: mailcatcher
    mailer_port: 1025
    mailer_user: postmaster@mg1.campr.biz
    mailer_password: null
```

Once you have changed the mail configuration, the mailcatcher service is no longer necessary and can therefore be removed from the configuration.

Login
=====
1. Call up the URL `campr.local`in your web browser.
2. Log in with your previously created user/ password.
3. After successful login you will see your created workspace e.g. `workspace1`.
Log in and create your first project!
