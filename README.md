# Unifi Controller Docker Container

This is Yet Another UniFi Controller.  It draws UniFi from the vendor-provided apt source and updates frequently.


## Update Policy

The image is updated on the following policy:

* Weekly, Sundays at 00:00: The `master` branch will be re-built and tagged `latest`.  This allows for upstream packages to be updated.  It will always use the latest version of UniFi.
* On UniFi update: When I notice the package has updated, I'll rebuild `master` and push to docker hub, tagged with the UniFi version.  I'll also update `latest` for good measure.
* Ad-hoc: As and when I make changes, I'll push to `dev`.  It'll probably work but it also might break.

For stability, choose `latest`.  For a specific UniFi revision, choose the tag but beware the platform will be as per the UniFi release and is highly likely to need an update.  Some of the builds are quite old.  For whatever I'm poking, choose `dev`.


## Image Installation

From Docker Hub:

```sh
docker pull lumel/unifi-controller
```
From Source:

```sh
git clone https://github.com/HenryJS/docker-unifi-controller.git
cd docker-unifi-controller
docker build -t "unifi-controller:latest" --rm .
```


## Running the Container

Create a volume to store the UniFi persistence data, then launch the 
container using the previously created volumes.

```sh
docker volume create --name unifi
docker run -d -p 8080:8080 \
              -p 8443:8443 \
			  -p 3478:3478/udp \
			  -p 10001:10001/udp
			  -v unifi:/usr/lib/unifi/data \
			  --name unifi \
			  lumel/unifi-controller
```

If, like me, you'd rather maintain state in a specific place in the local 
filesystem, do this instead:

```sh
DATA_PATH=/wherever/unifi-controller
mkdir -p $DATA_PATH
docker run -d -p 8080:8080 \
              -p 8443:8443 \
			  -p 3478:3478/udp \
			  -p 10001:10001/udp
			  -v ${DATA_PATH}:/usr/lib/unifi/data \
			  --name unifi \
			  lumel/unifi-controller
```


## Manual Update

If you'd like to update the package / distro manually, use the following:

```sh
docker exec -it unifi sh -c 'apt update && apt dist-upgrade'
```

Note this will also update the UniFi controller, so if you're pinning that for stability reasons, you'll have to hold the package first:

```sh
docker exec -it unifi sh -c 'apt-mark hold unifi && apt update && apt dist-upgrade'
```

## Troubleshooting

**Q: Adoption fails, reporting that my devices can't connect to the controller.**

**A:** When adopting a device, the controller needs to tell it where to talk 
back to; i.e. where the controller is.  By default, this is autodetected; as 
this controller runs in a container, the address it finds is the container's 
private IP - which on my system is in 172.16.0.0/12.   

To resolve, go to your Unifi Settings, hit Controller, then enter your 
Controller Hostname / IP and select *Override inform host with controller 
hostname/IP*.


## Author
- Henry Southgate - [Github](https://github.com/lumel-uk/)

Distributed under the GPL 3 license. See ``LICENSE`` for more information.

