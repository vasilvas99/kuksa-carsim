# Carsim example

This example has two communicating components communicating by publishing and reading data from the Kuksa Databroker.

## Carsim

### Simulator

Carsim is a basic kinematic model "bicycle vehicle model". Ref: [VEHICLE DYNAMICS: THE KINEMATIC BICYCLE MODEL](https://thef1clan.com/2020/09/21/vehicle-dynamics-the-kinematic-bicycle-model/).

It takes three external inputs: acceleration, brake and steering wheel angle and propagates the model system in time with a step defined through variable `simulation_step` (default: _0.001 **s**_).

The simulator itself can be found in the `carsim.py` file.

### Runner

`carsim_runner.py` is the "bridge" between the kuksa databroker python API and the simulator. It reads the following datapoints from the databroker:

```python
DP_BRAKE_POS = "Vehicle.Chassis.Brake.PedalPosition"  # uint8
DP_ACCELR_POS = "Vehicle.Chassis.Accelerator.PedalPosition"  # uint8
DP_STEER_ANGLE = "Vehicle.Chassis.SteeringWheel.Angle"  # int16
```

updates the simulator controls with them and steps through the simulator update.

After the new values for the acceleration components, speed and position are calculated, `carsim_runner.py` publishes the following datapoints to the databroker:

```python
DP_SPEED = "Vehicle.Speed"  # float
DP_ACCEL_LAT = "Vehicle.Acceleration.Lateral"  # float
DP_ACCEL_LONG = "Vehicle.Acceleration.Longitudinal"  # float
DP_ACCEL_VERT = "Vehicle.Acceleration.Vertical"  # float
```

This runner can server as an example of how 

## Driversim


### Simulator

This is an open-loop data feeder. It generates data for accelerator, brake and steering angle without reading back the state of the system of the controlled system.

The simulation step is still a parameter and has to match the simulation step for carsim.

The feeder has roughly two general states:
- "Good driver" - slowly accelerates and moves the steering angle accordingly and then slowly brakes.

- "Bad driver" - accelerates hard (full throttle), moves the steering wheel randomly in both directions, brakes hard.

After a set amount of time (based on simulation_step) it switches back and forth between the two states.

Roughly the fed data looks like so:

Acceleration/Brake %:

![Acceleration](driversim/data_plots/driversim_acceleration_brake.png)


Steering angle (degrees):
![Steering angle](driversim/data_plots/driversim_steer_angle.png)



### Runner

The runner is similar to the carsim_runner but only publishes to the databroker the system controls and steps through the simulation after each update.

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
$ carsim/docker-build.sh --local <arch>
$ driversim/docker-build.sh --local <arch>
```

Note: The `docker-build.sh` script can bootstrap an amd64/arm64/armv6 build by substituting the required architecture as an argument. The host architecture has to match <arch> or you should use QEMU. Otherwise staticx might fail to statically link the final binary.