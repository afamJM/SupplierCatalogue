# SupplierCatalogue
The master project for the Hitched Supplier Catalogue application. The master project serves
as a container for submodules.  At the time of writing, the submodules are:

*  SupplierCatlogue.API - The RESTful API to serve data for the catalogue
*  SupplierCatalogue.DataExtract - The application to extract data from Hitched and import it into the SupplierCatalogue API
*  SupplierCatalogue.UI - The web pages for the Supplier Catalogue that consume the data provided by the API

Both the DataExtract and the UI project are dependant on the API project.

![Current Architecture](/Docs/pages/prototype.png?raw=true)

## Docker Configuration
The Supplier Catalogue has been designed to run as a multi-container Docker application. See the  [Docker Compose Overview](https://docs.docker.com/compose/overview/) for more information.

### Pre-Requisites
In order to use the Docker functionality please ensure you first [install Docker](https://docs.docker.com/engine/getstarted/step_one/) in your chosen development environment.

You can check everything is working correctly with the following command:

`
docker-compose version
docker-machine version
`

If not use the following setup notes...

#### On PC

```
choco install docker docker-compose docker-machine

```

#### On Mac

```
brew cask install virtualbox
brew install docker
brew install boot2docker
brew install mongo
```


To upgrade docker use the following:

```
brew update
brew upgrade docker
brew upgrade boot2docker
brew upgrade mongo
```

Then tell OSX to start Mongo as a service (may need to be done on every reboot)

```
brew services start mongodb
```

### Configuration
Configuration of the application is achievied through an environmental variable configuration file stored in the project directory `.env`. At time of writing this contains the following settings:

Environmental Variable | Default Value
--- | ---
API_DEST | http://api:8080
datastore__connection | mongodb://db:27017
**_NB. These settings are required to ensure the Docker application runs correctly. Care should be taken in changing them._**

### Run the Docker application
Navigate to the project folder (containing docker-compose.yml) and issue the following command to build and run the Docker application:

`
docker-machine start default
docker-machine env
eval $(docker-machine env)
docker-compose up
`

Once running simmple issue `ctrl-c` to shut it dowwn.

This has also been added as an npm command which will run the application as a service. This can be initiated using:

`
npm start
`

One more command, check where your localhost is running:

`
docker-machine ip
docker ps
`

First command shows you the local IP address. The second lists out all the ports.

`
➜  SupplierCatalogue git:(master) ✗ docker-machine ip
192.168.99.100

➜  SupplierCatalogue git:(master) ✗ docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                           NAMES
0d21d45c78d7        suppliercatalogue_api      "dotnet run"             23 minutes ago      Up 23 minutes       0.0.0.0:8080->8080/tcp          suppliercatalogue_api
23e6374fd1a4        suppliercatalogue_import   "node"                   23 minutes ago      Up 23 minutes                                       suppliercatalogue_import
18660668855f        mongo:latest               "docker-entrypoint..."   23 minutes ago      Up 23 minutes       0.0.0.0:27018->27017/tcp        suppliercatalogue_db
fb8e3294c043        suppliercatalogue_ui       "npm start"              23 minutes ago      Up 23 minutes       443/tcp, 0.0.0.0:7080->80/tcp   suppliercatalogue_ui
`

This for example means you can go to 192.168.99.100:8080 for the swagger, and 192.168.99.100:80 for the UI.



#### Trigger the import process
Once the application is running the process to import the data to the API can be activated with the following npm command:

`
npm run-script import
`

This process will report the number of successful imports once it completes.


#### Access the MongoDB data store
Once the application is running the MongoDB data store container can be accessed via port 27018 using the command:

`
mongo -port 27018
`

_NB. You will need [MongoDB](https://docs.mongodb.com/manual/installation/) installed and the binaries added to your PATH variable for this to work._

### Stop the Docker application service
Once the application is runnning as a service you can terminate it using the following from the project folder:

`
docker-compose stop
`

Alternatively, the command has also been added to npm. This can be initiated using:

`
npm stop
`

### Destroy the application container
After the application has been run it may be necessary to detroy the pre-built containers in order to make changes. You can achieve this with the following command from the project folder:

`
docker-compose rm
`

This can also be achieved from the same location using the pre-built npm command:

`
npm run-script destroy
`
