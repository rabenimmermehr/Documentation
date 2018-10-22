# Documentation

This is the documentation of the Personal Health Train.

## Architecture diagram
- [PHT.archimate](PHT.archimate) is the current view on the architecture, available online at [personalhealthtrain.github.io](https://personalhealthtrain.github.io/Documentation/).
- [PHT-original.archimate](PHT-original.archimate) is the collection of original PHT architecture diagrams, as discussed during the technical meeting in Leiden in April 2018.

## Live view of the architecture
The latest export of the architecture diagram is available at [personalhealthtrain.github.io](https://personalhealthtrain.github.io/Documentation/).

## General Information

- [PHT Tech Google Group](https://groups.google.com/a/go-fair.org/forum/#!forum/pht-tech)

### Working groups

- [PHT Architecture Working Group](https://groups.google.com/a/go-fair.org/forum/#!forum/pht-tech-architecture)
- [PHT Use Cases Working Group](https://groups.google.com/a/go-fair.org/forum/#!forum/pht-tech-usecases)
- [PHT Legal Working Group](https://groups.google.com/a/go-fair.org/forum/#!forum/pht-tech-legal)

## Contents

- [Use cases](https://github.com/PersonalHealthTrain/Documentation/wiki/Use-Cases)
    - A set of use cases highlighting key situations envisioned for the Personal Health Train



## Library
This is an overview of the PHT library components that were developed for the JVM.
This section describes all the repositories that are prefixed with `lib-`. as
these comprise all the components of the library.

### Overview
![PHT Library](https://github.com/PersonalHealthTrain/Documentation/blob/master/figures/lib.png?raw=true "PHT Library")

Each node in this image describes one Git repository.
With the version dependencies.
Note that the version describe the actual dependencies (see respective `build.gradle`), not necessarily the most
recent version of the artifact.

The green repository are part of the PHT Library
and can be found in the [Personal Health Train GitHub Organization](https://github.com/PersonalHealthTrain).

The orange repositories belong to the [JDregistry](https://github.com/jdregistry)
project, which aims to provide APIs for the [Docker Registry](https://docs.docker.com/registry/) for the JVM. The JDregistry project was retrospectively extracted from the PHT library,
as it can be developed on a larger scope.

The following table gives an overview of the green nodes for the PHT lib project:

Repository Name and URL                                     | Description
------------------------------------------------------------|-------------
[lib-data](https://github.com/PersonalHealthTrain/lib-data) | This is the root project of the library. It contains the data classes that form the basis of class interactions.
[lib-runtime](https://github.com/PersonalHealthTrain/lib-runtime) | The runtime interfaces. Currently, the `DockerRuntimeClient` interface specifies how a Docker client needs to be implemented to be used with the PHT library.
[lib-runtime-docker-spotify](https://github.com/PersonalHealthTrain/lib-runtime-docker-spotify) | This repo **implements** the `DockerRuntimeClient` interface using the JVM Docker client from [Spotify](https://github.com/spotify/docker-client).   |
[lib-api](https://github.com/PersonalHealthTrain/lib-api) | Contains the actual Train API classes. It provides the specification of the concepts **TrainArrival** and **TrainDeparture** and also the specification and default implementation of the default **TrainRegistry**.  |  

Note that the library talks about two different kind of clients that have
something to do with Docker. It is important to understand the distinction:

* The **Docker Registry client** (which is developed in the
  [JDregistry](https://github.com/jdregistry) project) is a consumer of the
  [Docker Registry V2 API](https://docs.docker.com/registry/spec/api).
  Such clients do not require a Docker daemon installed on the host system,
  as they only rely on HTTP networking.
* The client interface defined in the
  [lib-runtime](https://github.com/PersonalHealthTrain/lib-runtime) project
  defines **Docker Runtime clients**, which require a Docker host installed on the host
  system. These clients can run containers, pull images, commit new images, and
  push images to registries.

The clients overlap in their requirements. For instance, both need access to
authentication credentials. The Docker Registry clients needs those to list
repositories from certain namespaces. The Runtime Client need those to pull
images from these very namespaces.

## Installation (October 2018)

The following sections describes how to set up all necessary components to run the Personal Health Train Infrastructure.
These installation steps have been performed and tested on a fresh Ubuntu 18.04.
Obviously, the stations and the registry are able to run on different systems.

### The registry / Portus

This [git repository](https://github.com/PersonalHealthTrain/Portus) contains the source code of the Portus-Project, adapted to the
needs of the Personal Health Train Registry.

#### Prerequisites

- Install Docker Community Edition (CE): <https://docs.docker.com/install/linux/docker-ce/ubuntu/>
- Install Docker Compose to launch multiple containers at once: <https://docs.docker.com/compose/install/>

#### Setup

```shell
git clone https://github.com/PersonalHealthTrain/Portus
cd Portus
git checkout feature/station_support
nano .env # Edit MACHINE_FQDN to match your IP
sudo docker-compose up # Takes a while to pull/build all containers & launch
```

- You can now access the Portus-Webinterface under <yourMachineIp:3000>, where you'll be prompted to generate an admin account.
- Afterwards, the interface asks for the location of the registry. The docker registry is a separate container and 
  has been launched via the `docker-compose` statement and therefore runs on the same host.
  - *Name*: DefaultRegistry
  - *Hostname*: <yourMachineIp:5000>
  - The *Create*-Button will become active after the connection check in the background was successful. Click it
- In the menu bar on the left, you should see an entry *Stations*. If not, you have launched the wrong branch of Portus, make sure  
  you checked out the correct branch
  checked out the correct branch
- Under the menu *Users* create a user for each station
  - **TODO** what's with the bot feature? Should/Could a station user be a bot?
  - **TODO** Ask Lukas whether this is necessary checked out the correct branch
- Under *Teams* create a team for the stations
  - **TODO** Is this necessary? A namespace just asks for a team, therefore I'm assuming it is
  - Select the newly created team by clicking on the name
  - Under *Members* add the station users to the team as *Contributor*s
- In the menu on the left navigate to *Namespaces* and create a namespace for the trains (e.g. *trains*). Provide the team you created
  a couple steps above as *owner*

- **TODO**
  - Create user for each station?
  - when do the stations show up in the user interface?
  - How do I upload a train?

### The station

This [git repository](https://github.com/PersonalHealthTrain/station) contains the source code for a simple Station, a docker client
configured to check the registry for new Trains/Docker Images, pulling & executing them.

#### Using the binary

- **TODO** Lukas mentionend it's possible to configure the station via environment params. Check source code to see how
- Describe use of binary

#### Prerequisites

- A Java Development Environment (See [documentation](https://github.com/PersonalHealthTrain/station#prerequisites))

  ``` shell
  sudo apt install openjdk-8-jdk
  ```

- [Gradle](https://gradle.org/install/) (Tested with 4.9 & higher, doesn't work with version 3):

  ```shell
  wget https://services.gradle.org/distributions/gradle-4.10.2-bin.zip
  sudo mkdir /opt/gradle
  sudo unzip -d /opt/gradle/ gradle-4.10.2-bin.zip
  export PATH=$PATH:/opt/gradle/gradle-4.10.2/bin # If necessary, additionally add to PATH permanently
  gradle -v # Should print welcome message
  rm gradle-4.10.2-bin.zip
  ```

  Note: `sudo apt install gradle` installs version 3, which isn't compatible

- **TODO** Docker client / runtime?

#### Configure & Build from source

```shell
git clone https://github.com/PersonalHealthTrain/station
git checkout develop # Currently only the develop branch is building
cd station
nano src/main/resources/application.yml # Configure the station to your settings
gradle assemble
java -jar build/libs/station-0.0.2.jar # Actually run the station
```

### Train

 **TODO** How to create a train and how to push it to the registry
