# simulator
Our work extends Batsim (https://batsim.readthedocs.io/) by applying patches to the original source. It has been packaged here to be applied and built in a dockerized format to ease of use and replication of our experimental data.  Those that wish to learn more about the native Batsim are encouraged to visit the Batsim homepage.

## How to build the docker
clone this repo:<br/>
```git clone https://github.com/hpec-2021-ccu-lanl/simulator.git``` <br/>
enter directory:<br/>
```cd simulator```<br/>
build the docker and name the image "simulator": <br/>
```docker build . -t simulator```<br/>

## How to run the docker

create the docker container based off the "simulator" image and name it "batsim_docker":<br/>
```docker create --name batsim_docker -t simulator```<br/>
start the docker container: <br/>
```docker start batsim_docker```<br/>
start an interactive shell:<br/>
```docker exec -it batsim_docker /bin/bash```

## How to run some simulations

1.  Start the docker container: `docker start batsim_docker`
2.  Start an interactive shell: `docker exec -it batsim_docker /bin/bash`
3.  Change directory to "basefiles": `cd /home/sim/basefiles`
4.  Choose a config file and optionally edit it (*below): `nano ./configs/1_simulation.config`
5.  Set the config file you wish to run:`file1=./configs/1_simulation.config`
6.  Set the output folder you wish the output to go to ( a new folder ): `folder1=/home/sim/experiments/1_sim`
5.  Run the simulation: `python3 run_simulation.py --config $file1 --output $folder1`

\* Instructions for editing config files can be seen by running the following commands:
 - view general info on config files: `python3 generate_config.py  --config-info general`
 - view general info on sweeps: `python3 generate_config.py --config-info sweeps`
 - All --config-info options can be seen by running: `python3 generate_config.py --help`
    - Under `Required Options 1 -> --config-info <type>` you can see the various types of info that is offered
    - generate_config.py will not generate your config file for you.  It is called that because it takes a config file that you will need to write and generates the underlying config files the simulator needs.

### For example:
  - Run a modified Figure 4, left-hand subfigure for workload 4:
    - `file1=./configs/figure4_left_wl4.config`
    - `folder1=/home/sim/experiments/fig4_left_wl4`
    - `python3 run_simulation.py --config $file1 --output $folder1`
    - `python3 aggregate_makespan.py -i $folder1`
    - `python3 process_aggregation.py -i $folder1/total_makespan.csv`
