Welcome to Campr On Premise builds!
===================================

Table of contents:
- [General Information](#general-information)
- [Domains](#domains)
- [Setup](#setup)
- [Customizations](#customizations)

General Information
===================

This repo provides the basic for setting up *Campr On Premise* on your local network or own servers.

The basics consist of:
- a `docker-compose.yml` file with everything preconfigured
- a few configuration files required by the applications and the services that they relly on

`Campr On Premise` is the on premise version of [Campr](https://campr.biz). It is made out of 2 applications, `portal` and `workspaces` and uses a mysql database and redis cache server to deliver the application.

The 2 apps are behind an nginx proxy which routes requests to the proper application depending on the domain.

This guide assumes you are familiar with `docker` and `docker-compose`. Upon request and negotiation, assistance can be provided for setting the project up.

For convenience, a `mailcatcher` service is preconfigured so that you can set your installation up easily. Please see the [Customizations](#customizations) section to find out how you can change the mailer configuration.

Domains
=======

The default domains used are:
- `campr.local`
  - this is the `portal` app where you can login and create workspaces
- `*.campr.local`
  - this is the `workspaces` as, where each workspace can be worked on as a separate environment

As you can see we're using `campr.local` as a base domain, which means you will have to configure these domains in your `/etc/hosts` file manually. An example would look like this:

```
127.0.0.1   campr.local
127.0.0.1   workspace1.campr.local
```

Unfortunately, wildcard domains are not supported out of the box so you wil have to add each created workspace as an entry there if you plan on keeping the domains as they are.

Customizing the domains will be detailed in the [Customizations](#customizations) section.

Setup
=====

The campr on premise repository is located here: http://lab.trisoft.ro/campr/on-premise

To get access to this repository, contact `radu.topala@trisoft.ro`

Once you've acquired access to the repository, use the same username and password to login to the `docker` container repository as well.

Login to the docker repository:

```
docker login lab.trisoft.ro:4567
```

Once you have logged in, you can simply run `docker-compose up -d` to start the project.

*Note: You do not need to worry about database migrations, they are automatically ran by the applications on start up.*

Customizations
==============

Changing the domain from `campr.local`
--------------------------------------

Change every occurence of `campr.local` with your new domain in the following files:

- `config/workspaces/backend/app/config/parameters.yml`
- `docmer-compose.yml`
- `config/portal/.env`

Ensure that the `VIRTUAL_HOST` environment variables for the `workspaces` service is set to be a wildcard for the new domain.

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

In the `config/workspaces/backend/app/config/parameters.yml` file change this:

```
MAILER_URL=smtp://mailcatcher:1025?auth_mode=login&username=&password=
```
