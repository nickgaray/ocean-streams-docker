# OpenSensorHub Build and Deployment Docker for Ocean Streams Project

![](/logo/ocean-streams-docker-nobg.png)

This Docker template will work with OpenSensorHub projects based on
[Ocean Streams](https://github.com/nickgaray/ocean-streams)
for creating OpenSensorHub nodes. The image is using the OSGi support built
into [version 2.0 of OpenSensorHub](https://github.com/opensensorhub/osh-core/tree/v2). OSGi allows for ease of
extensibility by managing addons as bundles that can be downloaded and configure while the OpenSensorHub node remains
running. Additional bundles can be created by using the [Ocean Streams](https://github.com/nickgaray/ocean-streams)
OpenSensorHub project and writing new modules for sensor, processes, and service. See
the [README.md](https://github.com/nickgaray/ocean-streams/blob/main/README.md) file for instructions on building.

The [Ocean Streams](https://github.com/nickgaray/ocean-streams)
allows you to write new modules for [OpenSensorHub](www.opensensorhub.org) and build a
deployable distribution.

This docker template allows you to point to a specific branch and repository URL in order
to clone, build, and deploy your own nodes in a containerized environment.

This project also includes, via a docker.io, an [Apache](https://httpd.apache.org/download.cgi) file server as a
location to host any additional OSGi bundles not packaged by default.
See [configuring the bundles repo](#configuring-the-bundles-repo) section
of [Executing via Docker Compose](#executing-via-docker-compose)

## Cloning this Repository

    git clone https://github.com/nickgaray/ocean-streams-docker.git

## Setting Up Default Configuration

This project contains a default ```config.json``` file that is used by OpenSensorHub to maintain the configuration
parameters of the various modules (driver, processes, services) managed by OSH. Either prior to building or executing
the image there are two optional properties that are recommended to be changed:

1. **bundleRepoUrls** - an array of URLs identifying file servers from which new OSGi bundles may be downloaded for use
   in
   OpenSensorHub. The locations are of the form ```["URL | ADDRESS:PORT/index.xml"]```. If you are either not setting up
   a bundle repository or do now currently know where one exists to be pointed to, then leave the field unmodified.
2. **deploymentName** - This helps identify an OSH Node and by default contains "Ocean Streams"

The properties can be found by editing the ```config/config.json``` file and navigating to the config block for the
object class ```org.sensorhub.ui.AdminUIConfig``` as seen below.

```
{
  "objClass": "org.sensorhub.ui.AdminUIConfig",
  "widgetSet": "org.sensorhub.ui.SensorHubWidgetSet",
  "bundleRepoUrls": ["192.168.1.81:9090/index.xml"],
  "customPanels": [],
  "customForms": [],
  "id": "5cb05c9c-9123-4fa1-8731-ffaa51489678",
  "moduleClass": "org.sensorhub.ui.AdminUIModule",
  "name": "Admin UI",
  "deploymentName": "Ocean Streams",
  "autoStart": true
},
```

If your mounted volumes are not located where this project is cloned, on first time running, the image will copy
the default ```config/config.json``` into the directory for the corresponding __config
__ [mounted volume](#declare-volume(s)-for-mount-point-directories). You can edit the config, relaunch the image, and
the changes will be seen by OpenSensorHUb.
Optionally, you may want to copy and modify the provided ```config.json``` prior to starting the image for
OpenSensorHub.

## Building the Project

Building is as simple as running the following command replacing the placeholders in ```[]```
with appropriate values. See the placeholders description below the build command template.

    docker build -t [tag] . -f dockerfile --build-arg BRANCH=[branch] --build-arg REPO_URL=[url]

- **tag**: The tag to assign to the image being built
- **branch**: The branch to check out
- **url**: The URL of the repository to an OpenSensorHub project based
  on [OSH Node Dev Template](https://github.com/opensensorhub/osh-node-dev-template.git)

## Understanding Docker Commands

It is highly recommended that the user be or become familiar with docker and the following commands

● To run Docker image detached exposing ports

     sudo docker run -d -p443:443 [tag]

- **-p port_visible_to_world:port_exposed_docker_container**:
  Expose additional ports by including more -p switches, one for each port to be mapped

● To list Docker images:

     sudo docker images

● To see which Docker image is/are running:

     sudo docker ps

● To see which Docker all Docker images running or stopped:

     sudo docker ps -a

● To kill a Docker image:

     sudo docker kill <container id>

● To gracefully start a stopped a Docker image:

     sudo docker start <container id | friendly name>

● To gracefully stop a Docker image:

     sudo docker stop <container id | friendly name>

● To build and tag an image:

     sudo docker build -t <repository>:<tag> . -f <dockerfile>

● To export an image:

     sudo docker save --output [filename].tar <docker_registry>/<repository>:<tag> . -f <dockerfile>

● To import an image:

     sudo docker load --input [filename].tar

## Executing via Docker Compose

This option will require the directories for mounted volumes to be created and the ownership set to the osh uid and gid
prescribed in the Dockerfile. In addition, the ```docker_compose.yml``` file must be edited to the image to use from
the docker repository as well as the ports to expose where in the mapping for ports operates on the same principle as
the **-p** command line switch discussed in the previous section.

    image: [DOCKER REPO URL]/[PATH]/[PROJECT]:[TAG]
    ports:
      - [EXTERNAL]:[INTERNAL]/tcp
      - [EXTERNAL]:[INTERNAL]/tcp

- **DOCKER REPO URL**:
  The URL of the docker repo to pull the image from
- **PATH**:
  The path within the docker repo to the hosted project images
- **PROJECT**:
  The project or image name to pull
- **TAG**:
  The tag of the image to pull, such as a version identifier

### Declare volume(s) for mount point directories

● **config**: Location of config.json and logback.xml.

● **data**: Suggested location to save H2 database files. Can be referenced as "./data" in node configuration.

● **db**: Used by OpenSensorHub to manage node specific data. Can be referenced as "./db" in node configuration.

● **bundles**: Any additional OSGi Bundles that the user may want to include after install.

The command to execute is:

     docker compose -f docker_compose.yml up -d

- **-d**
  Executes the image in detached mode, so it is safe to close the terminal window

### Configuring the Bundles Repo

The ```docker_compose.yml``` file contains an entry for the Apache file server under the __services__ block. This file
server provides the ability to host additional OSGi bundles and is preconfigured to download the latest image of
the Apache file server image. This server is reachable by clients via port 9090, but you can change this value but be
sure to not change the internal port or to set the external port to a port already configured for another application or
service. See the [Ocean Streams](https://github.com/nickgaray/ocean-streams) OpenSensorHub project for optional or
additional bundle creation and the generation of the bundles ```index.xml``` document.

The file server service also requires the specification of the mounted volume to use as the location from which the
available bundles should be retrieved.

    apache:
      image: docker.io/library/httpd:latest
      container_name: osgi-bundles-server
      ports:
        - 9090:80/tcp
      volumes:
        - [BUNDLES REPO LOCATION]:/usr/local/apache2/htdocs

- **BUNDLES REPO LOCATION**:
  The location in the file system where the bundle jar files and the index.xml document are hosted enabling additional
  bundles to be downloaded and installed onto an existing OSH node.  **DO NOT MAP THIS TO THE SAME PATH AS THE
  NODE'S ```bundles```** directory. That path is reserved for default bundles and bundles installed on the node to which
  they are mapped via the ```docker_compose.yml``` file.

If not deploying with a bundle repo do not uncomment the block from the ```docker_compose.yml``` file. Otherwise, make
sure to set your  [config](#setting-up-default-configuration) ```bundleRepoUrls``` in the ```config/config.json``` to
point to the address of this bundle repository.

### Shutting Down via Docker Compose

     docker compose down

## Executing or Running a Docker Container Manually

● To run docker image detached with mounted file system & name, using present working directory for filesystem source

Important: Make sure to create osh user and group on the host system for the volume to be mounted and set owner and
group to osh:osh for the volume being mounted

     docker run -d \
     -it \
     --name [container-friendly-name] \
     --mount type=bind,source=[mount-path],target=/opt/[osh-node-path]/[target-dir] \
     [tag]

- **container-friendly-name**: a friendly name for the docker container
- **mount-path**: the absolute path to the directory to be mounted as a volume
- **osh-node-path**: the absolute path to the directory where OpenSensorHub lives within the container
- **target-dir**: the name of the directory where the source should be mounted within the target
- **tag**: The tag to assign to the image being built, usually of the form ```repo_url/image:tag``` where
  the ```repo_url/``` may be omitted

It is recommended to start the image using the following command if you want to mount a host filesystem path directory
where data is typically stored in OSH, making this data accessible outside the docker instance and persisting across
executions of the instance. The config.json can be stored in this path to persist configuration. The launch script needs
to be updated, if doing this, so that config.json and db are correctly referenced.

     docker run -d -p 443:443 -p80:80 \
     -it --name [container-friendly-name] \
     --mount type=bind,source=[mount-path]/data,target=/opt/[osh-node-path]/data \
     [tag]

- **container-friendly-name**: a friendly name for the docker container
- **mount-path**: the absolute path to the directory to be mounted as a volume
- **osh-node-path**: the absolute path to the directory where OpenSensorHub lives within the container
- **target-dir**: in this example it has been set to data and will contain the config and recorded data
- **tag**: The tag to assign to the image being built, usually of the form ```repo_url/image:tag``` where
  the ```repo_url/``` may be omitted

## Testing Deployment

On your favorite browser go to: ```[protocol]://[address]:[port]/sensorhub/admin```
When prompted, unless changed utilize the default node admin username and password.
If you have deployed to another server, make sure to change ```localhost``` to the correct IP or base url.

- **protocol**: Either ```http``` or ```https``` according to how the node is configured in the
  nodes ```config/config.json```
- **address**: The address of the system hosting the node
- **port**: The port number configured in the nodes ```config/config.json```
