<p align="center"><img src="/documentation/images/logo.png" height="25%" width="25%" ></p>

# q.

An exploration into full-stack web engineering and project management, based on a restaurant's queue management system.

**q.** is a restaurant queue management platform catered towards allowing customers to queue up for a restaurant digitally, so that customers do not have to travel to a restaurant to wait in line.

Customers are able to see their current position in line and total wait time, while restaurant managers have the ability to manage the restaurant's queue.

## Team

  - [Johnny Li](https://github.com/Pegaiur)
  - [Tiffany Pham](https://github.com/tiffaphm)
  - [Neha Chaudhary](https://github.com/nehacp)
  - [Eugene Soo](https://github.com/eugenesoo)

## Table of Contents

1. [Usage](#Usage)
1. [Requirements](#requirements)
1. [Development](#development)
    1. [Installing Dependencies](#installing-dependencies)
    1. [Tasks](#tasks)
1. [Roadmap](#roadmap)
1. [Contributing](#contributing)

## Getting Started

### Introduction
q. comes with a customer-facing page, a queue information page and a manager-facing page/

Manager facing page:

```
http://<server_url>/manager
```

Default login information:

|field       |value        |
|------------|-------------|
|username    |johnny       |
|password    |hunter2      |

Currently, the manager page has been developed for a single restaurant.

Customer facing page:

```
http://<server_url>/customer
```

Queue information page:

```
http://<server_url>/customer/queueinfo
```

Visiting `http://<server_url>/` redirects you to either `/customer` or `/customer/queueinfo?queueId=[queueId]`, depending on whether a spot in queue has been reserved at a restaurant.

### Deployment
**How to deploy**

Currently, q. is currently hosted on [Heroku](https://www.heroku.com/) and [Linode](https://www.linode.com/).

The webserver and postgreSQL server is hosted on Heroku, while the session store is hosted on DigitalOcean.

#### Heroku Deployment
This section outlines the steps on how to deploy the webserver and postgreSQL server to [Heroku](https://www.heroku.com/).

1. Follow instructions on [Heroku - Getting Started](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction) to deploy the app on heroku.

2. Create a postgres database for your app. Follow instructions on [Heroku Postgres](https://devcenter.heroku.com/articles/heroku-postgresql).

3. To post dummy data in the database, use postman or curl to make a POST request to http://<server_url>/dummydata
  
  Note: Post request for dummy data will delete all tables in the database and re-create them.

#### DigitalOcean Deployment
This section outlines the steps on how to deploy the session store database to [Linode](https://www.linode.com/).

Linode provides a similar service to [DigitalOcean](https://www.digitalocean.com/), allowing you to deploy a linux server with a static IP address.

The session store for q. is currently configured to use a Redis Server, and the simplest way to deploy a Redis Server is to use a pre-made docker image.

##### Setup
1. Create and setup an 64-bit Ubuntu-based linux server on Linode or DigitalOcean.

1. Using the IP address and password for your cloud server, log in by running the ssh command with username@ipaddress as the argument. For example ssh root@'your cloud server ip address'

1. Install Docker using the repository.

Refer to [Docker installation instructions](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository) for more information. **At Step 4 of the instructions, use `amd64` for the stable apt repository.** `amd64` refers to 64-bit operating systems, if you're interested to know :)

1. Pull the Redis docker image from dockerhub.

```
sudo docker pull redis //clones the redis image from dockerhub to your ubuntu server.
```

Refer to the [official repository for redis](https://hub.docker.com/_/redis/) for more information.

1. Start the redis image with the following script:

```
sudo docker run --name q-sessions -p 6379:6379 -d redis redis-server --appendonly yes
```

This creates a new instance/container of the redis server, WITH persistant storage so that when you restart the docker container, the session information will not be wiped. 

This also exposes port 6379 on the docker container to port 6379 of your linode/digitalocean server, so that it is possible to access the redis server via `http://<linode_ip_address>:6379/`

It would be helpful to also install redis-cli, to be able to access the redis server like how you would access a mysql server.

To install redis-cli, you can install npm, then install redis-cli:

```
sudo apt-get install npm
sudo apt-get install redis-cli
```

Here are some useful redis commnds:
```
keys * //shows you all the key-value pairs
get [key name] //shows you the information in the key
```

Here are some useful docker commands:

```
sudo docker ps -a //shows all containers, including the not active containers
sudo docker attach [containername, for e.g. qsessions] //attach the container so that you can get inside of it. use ctrl-c to exit, but you have to start the container again once you have exited the container.
```

1. add the following config variables in Heroku:

```
REDISURL: points to the linode ip address.
REDISPORT: point this to 6379, or whichever port you exposed the docker container port to.
REDISSECRET: set a random string here to generate session id hashes with.
```

#### Initialisation

NPM Scripts:

|script name   |action                            |
|--------------|----------------------------------|
|start         |starts server with nodemon        |
|client        |starts webpack to bundle js files |

To get started, run

```
npm install
npm run client
npm start
```

**Setup Postgres Database**

1. [Install](https://www.postgresql.org/download) postgres on your local computer. 

Note: At this point you will not have a database called 'q-dot'. You will have to create it manually on your local computer.

2. Go to Postgres in your applications folder and run SQL Shell. The default database is called 'postgres'. Login in to the database.

3. Run command 

```
create database qdot;
```

4. Make a post request for dummy data. Details below.

**Add Dummy Data**

To add dummy data to the database, use postman or curl to make a POST request to `http://<server_url>/dummydata`

  Note: Post request for dummy data will delete all tables in the database and re-create them.

### File Structure

```
.
├── API-DOCUMENTATION.md //documentation for API
├── CONTRIBUTING.md //documentation for Contribution Guidelines
├── LICENSE //license file
├── Procfile //file that heroku uses to run initial commands after deployment
├── README.md //this file
├── STYLE-GUIDE.md //documentation for style-guides
├── app.json //heroku requires this file
├── client //where are client files get stored
│   ├── dist //where client files are served
│   │   ├── customer //routes to http://<server_url>/customer comes here
│   │   │   ├── css
│   │   │   │   ├── customermain.css
│   │   │   │   ├── materialize.css
│   │   │   │   └── materialize.min.css
│   │   │   ├── fonts
│   │   │   │   └── roboto
│   │   │   ├── index.html
│   │   │   ├── js
│   │   │   │   ├── materialize.js
│   │   │   │   └── materialize.min.js
│   │   │   └── queueinfo
│   │   │       └── index.html
│   │   ├── images
│   │   ├── js
│   │   │   ├── customerApp-bundle.js
│   │   │   ├── managerApp-bundle.js
│   │   │   └── queueinfo-bundle.js
│   │   ├── manager //routes to http://<server_url>/manager comes here
│   │   │   ├── index.html
│   │   │   └── style.css
│   │   └── managerlogin //routes to http://<server_url>/managerlogin comes here
│   │       ├── index.html
│   │       └── styles.css
│   └── src
│       ├── components
│       │   ├── customer //customer components
│       │   │   ├── CustomerApp.jsx //renders main components
│       │   │   ├── CustomerHome.jsx //renders customer home page
│       │   │   ├── CustomerInfoForm.jsx //nested in SelectedRestaurant, renders input form
│       │   │   ├── CustomerMain.jsx //component that renders other component based on url
│       │   │   ├── CustomerNav.jsx //renders navigation bar
│       │   │   ├── GroupSizeSelector.jsx //nested in CustomerInfoForm, renders group size selector
│       │   │   ├── QueueInfo.jsx //renders /customer/queueinfo
│       │   │   ├── RestaurantCard.jsx //nested in CustomerHome, renders each restaurant in list
│       │   │   ├── RestaurantInformation.jsx //nested in SelectedRestaurant, displays queue info
│       │   │   ├── RestaurantLogoBanner.jsx //nested in Selected Restaurant, renders restaurant image
│       │   │   └── SelectedRestaurant.jsx //renders main page after selecting restaurant
│       │   ├── manager //manager components
│       │   │   ├── AddToQueue.jsx //renders add to queue button on manager page
│       │   │   ├── CustomerList.jsx //renders the list of customer queuing up and remove customer modal
│       │   │   ├── CustomerListEntry.jsx //renders each customer queue entry
│       │   │   ├── ManagerApp.jsx //renders main page for manager
│       │   │   ├── ManagerAudit.jsx //renders login history for manager
│       │   │   ├── Nav.jsx //renders navigation bar
│       │   │   └── StatusSwitch.jsx //renders open/close queue on navigation bar
│       │   └── managerlogin //manager login components
│       │       └── ManagerLogin.jsx //renders login page
│       ├── customerIndex.jsx //renders /customer page
│       ├── managerIndex.jsx //renders /manager page
│       ├── managerLoginIndex.jsx //renders /managerlogin page
│       └── queueinfoIndex.jsx //renders /customer/queueinfo page
├── controller
│   ├── index.js //contains functions that does a query to the database
│   └── manager.js //contains all manager-related queries to the database
├── database
│   ├── dummydata.js //contains all functions that creates dummy data
│   └── index.js //contains schema information and associations
├── documentation //contains documentation files
│   └── images
│       └── logo.png
├── helpers
│   └── helpers.js //contains any functions not related to database queries or server
├── package-lock.json
├── package.json
├── server
│   ├── components
│   ├── index.js //contains express, sockets and all routings
│   └── passport.js //contains settings for passport.js
├── test
│   └── test.js //contains test suite
└── webpack.config.js
```

## Requirements

- babel-cli 6.7.5
- babel-core 6.26.0
- babel-loader 7.1.2
- babel-preset-es2015 6.6.0
- babel-preset-react 6.24.1
- babel-register 6.7.2
- body-parser 1.18.2
- bootstrap 3.3.7
- connect-redis 3.3.2
- css-loader 0.28.7
- express 4.15.5
- express-session 1.15.6
- file-loader 1.1.4
- jquery 3.2.1
- passport 0.4.0
- passport-local 1.0.0
- pg 6.4.2
- pg-hstore 2.3.2
- react 16.0.0
- react-dom 16.0.0
- redis 2.8.0
- request 2.83.0
- sequelize 4.13.2
- socket.io 2.0.3
- style-loader 0.18.2
- url-loader 0.5.9
- webpack 3.6.0
- react-router-dom 4.2.2

## Development

### Installing Dependencies

From within the root directory:

```sh
npm install
```

### API Documentation

Please refer to [API Documentation](API-DOCUMENTATION.md).

### Roadmap

View the project roadmap [here](https://docs.google.com/document/d/1nZWyFHd6PuJd0sl97Ik7XCR5y3FO7S336ywUhs0l6vQ/edit)

Features in black have not been implemented.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.
