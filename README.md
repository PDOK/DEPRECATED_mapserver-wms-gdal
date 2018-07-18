# Mapserver WMS GDAL

## Introduction
This project aims to fulfill two needs:
1. create a [OGC services](http://www.opengeospatial.org/standards) that are deployable on a scalable infrastructure.
2. create a useable [Docker](https://www.docker.com) base image.

Fulfilling the first need the many purpose is to create an Docker base image that eventually can be run on a platform like [Kubernetes](https://kubernetes.io/).

Regarding the second need, finding a usable Mapserver Docker image is a challenge. Most images expose the &map=... QUERY_STRING in the getcapabilities, don't run in fastcgi and are based on Apache.

## What will it do
It will create an Mapserver application run with a modern web application NGINX in which the map=.. QUERY_STRING issue is fixed. The application is configured for service raster files (GeoTIFF)

## Components
This stack is composed of the following:
* [Mapserver](http://mapserver.org/)
* [GDAL](http://www.gdal.org/)
* [NGINX](https://www.nginx.com/)
* [Supervisor](http://supervisord.org/)

### Mapserver
Mapserver is the platform that will provide the WFS, WMS of WCS services based on a vector datasource (Geopackage, SHAPE, Postgis).

### GDAL
Is used for the rasterfiles

### NGINX
NGINX is the web server we use to run Mapserver as a fastcgi web application. 

### Supervisor
Because we are running 2 processes (Mapserver CGI & NGINX) in a single Docker image we use Supervisor as a controller.

## Docker image

The Docker image contains 2 stages:
1. builder
2. Service

### builder
The builder stage compiles Mapserver. The Dockerfile contains all the available Mapserver build option explicitly, so it is clear which options are enabled and disabled.

### service
The service stage copies the Mapserver, build in the first stage, and configures NGINX and Supervisor.

## Usage

### Build
```
docker build -t pdok/mapserver-wms-gdal .
```

### Run
This image can be run straight from the commandline. A volumn needs to be mounted on the container directory /srv/data. The mounted volumn needs to contain at least one mapserver *.map file. The name of the mapfile will determine the URL path for the service.
```
docker run -d -p 80:80 --name mapserver-run-example -v /path/on/host:/srv/data pdok/mapserver-wms-gdal
```

The prefered way to use it is as a Docker base image for an other Dockerfile, in which the necessay files are copied into the right directory (/srv/data)
```
FROM pdok/mapserver-wms-gdal

COPY /etc/example.map /srv/data/example.map
```
Running the example above will create a service on the url: http:/localhost/example/wms? An working example can be found: https://github.com/PDOK/mapserver/tree/natura2000-example

## Misc
### Why NGINX
We would like to run this on a scalable infrastructure like Kubernetes that has it's Ingress based on NGINX. By keeping both the same we hope to have less differentiation in our application stack.

### Used examples
* https://github.com/srounet/docker-mapserver
* https://github.com/Amsterdam/mapserver
