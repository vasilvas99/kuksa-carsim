## Basic how to start

Clone the repo and run in 3 separate terminals:

TERMINAL 1: 

```bash
$ docker run -it --rm --net=host ghcr.io/eclipse/kuksa.val/databroker:master
```

TERMINAL 2: 

```bash
$ pip3 install -r carsim/requirements.txt 
$ python3 carsim/carsim_runner.py
```

TERMINAL 3: 

```bash
$ pip3 install -r driversim/requirements.txt 
$ python3 driversim/driver_runner.py
```

## Using containers from workflows:

### Using host networking

This would be somewhat equivalent to running the scripts directly on the host, but without needing to 
install the python dependencies (or having python installed on the host).



TERMINAL 1: 

```bash
$ docker run -it --rm --net=host ghcr.io/eclipse/kuksa.val/databroker:master
```

TERMINAL 2: 

```bash
docker run --net=host -e "LOG_LEVEL=INFO" --rm -it ghcr.io/vasilvas99/carsim:main
```

TERMINAL 3: 

```bash
docker run --net=host -e "LOG_LEVEL=INFO" --rm -it ghcr.io/vasilvas99/driversim:main
```


### Using container-to-container networking:

1) Create a custom docker bridge network :

```bash
$ docker network create --driver bridge sim-net
```

In 3 separate terminals:


2) Run the kuksa container

```bash
$ docker run --name databroker_cont --network sim-net --rm -it ghcr.io/eclipse/kuksa.val/databroker:master
```

3) Run the car simulator

```bash
$ docker run --name simulator  --network sim-net -e "LOG_LEVEL=INFO" -e "DATABROKER_ADDRESS=databroker_cont:55555" --rm -it ghcr.io/vasilvas99/carsim:main
```

4) Run the driver simulator 

```bash
$ docker run --name driver  --network sim-net -e "LOG_LEVEL=INFO" -e "DATABROKER_ADDRESS=databroker_cont:55555" --rm -it ghcr.io/vasilvas99/driversim:main
```

## Building the containers locally

```bash
$ carsim/docker-build.sh --local amd64
$ driversim/docker-build.sh --local amd64
```