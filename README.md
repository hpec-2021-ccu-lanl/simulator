# simulator
Our work makes use of Inria's Batsim (https://batsim.readthedocs.io/) simulator. We have added a node fault model and simulated job checkpoint / restart in order to more easily explore the trade-offs between performance and reliability. This repo is associated with our IEEE HPEC 2021 submitted article entitled "Exploring the Tradeoff Between Reliability and Performance in HPC Systems." <br/>

Scripts are provided to apply patches to the original Batsim source and run experiments congruent with those presented in our article. They have been packaged here to be applied, built, and executed in a dockerized format for ease of use and replication of our experimental data presented in the HPEC article.  Those that wish to learn more about the native Batsim are encouraged to visit the Batsim homepage directly.



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




## Test the batsim_docker

Test the batsim_docker to see if it gives you the correct results.  This will make sure the docker is running properly, but will also give you a chance to see how the process
of running simulations goes.  We will use a config file "test_docker.config".<br/>
```
{  "test":{
            "input": {
                        "node-sweep":{
                                "range":[1490]
                        },
                        "SMTBF-sweep":{
                                "compute-SMTBF-from-NMTBF":true,
                                "formula":"128736000 * (1/i)",
                                "range":[1,8]
                        },
                        "checkpoint-sweep":{
                                "range":["optimal"]
                        },
                        "performance-sweep":{
                                "range":[1.0]
                        },
                        "checkpointing-on":true,
                        "synthetic-workload":{
                                "number-of-jobs":30000,
                                "number-of-resources":"/home/sim/basefiles/workload_types/wl2.csv:0:csv",
                                "duration-time":"/home/sim/basefiles/workload_types/wl2.csv:1:h:csv",
                                "submission-time":"0:fixed",
                                "wallclock-limit":-1,
                                "dump-time":"3%",
                                "read-time":"2%"
                        }
          },
          "output": {
                        "AAE":true,
                        "avg-makespan":1
          }
   }
}
```

### Just a real quick intro to this config file...
- We are sweeping over nodes, but really there is no sweep, as we used a fixed "range" and in that list of nodes there
is only one value [1490].  There are tools available to do a real sweep, but we won't get into that just yet.<br/><br/>
- We use a formula for (**S**)ystem (**M**)ean (**T**)ime (**B**)etween (**F**)ailure.  
  - 128736000 * 1/i where "i" is replaced with a "range": [1,8]
  - So we will have 128736000 / **1** and 128736000 / **8**
  - For clarification: 128,736,000 seconds = **24hrs** * **3600seconds/hr** * **1490nodes/system** * **1 system**
    - so this is a system failure rate of 1 failure every 24 hours for baseline ( "range" : [1] ) and for 8x worse ( "range" : [8] )<br/><br/>
- We let the simulator compute the optimal checkpointing for each job
- We set the speed of the system to 1.0 (normal speed) where higher is slower ( a 30% faster and 30% slower system would be 0.70 and 1.30 respectively )
- We give an option for all jobs, checkpointing-on.
- We set up a workload
- We tell it what kind of output we need
  - Average Application Efficiency
  - makespan with only 1 run   (replace "1" with "200" for 200 runs)
    - we did not pass the seed-failures option to all jobs.  This means our results will be deterministic.  Therefore, we only need 1 run.  Normally we want to take an average of at least 200 runs when it is not deterministic. However, we need to confirm exact numbers for our test so we will keep it deterministic by leaving the seed-failures option off.





### Ok, let's run this test

1. you should already be in the /home/sim/basefiles directory.   If not, head there. `cd /home/sim/basefiles`
2. set two variables to make things easier:<br/>
    ```file1=./configs/test_docker.config```
    ```folder1=test_docker```
3. Now we get our workloads made, input/output folders made for each run, underlying config files made, and then the simulations begin.
    ```python3 run_simulation --config $file1 --output ~/experiments/$folder1```
  - since the docker is not a cluster and simulations run sequentially, there is a handy counter that is flushed to output before every simulation.  For example:
```
    Experiment 1/1
    Job 1/2
    Run 1/1

    ...

    Experiment 1/1
    Job 2/2
    Run 1/1
```
4. We have results but they need to be aggregated: ```python3 aggregate_makespan.py -i ~/experiments/$folder1```
  - After running this you should have a file called "total_makespan.csv" under the ~/experiments/$folder1 folder

5. Now to make determining whether we have correct results easier, run the following:<br />
  ```
  cat ~/experiments/$folder1/total_makespan.csv | \
  awk -F, BEGIN'{printf "\n"}''(NR>1)''{printf "%f\t%s,%s\n",$6,$7,$8}'END'{printf "\n"}'
  ```

  you should get
  ```

  4896980.836070  "56 days, 16:16:20"            <----fyi baseline makespan (displayed as seconds, then days,h:m:s)
  7861976.738434  "90 days, 23:52:56"            <----fyi 8x worse failures makespan (displayed as seconds, then days,h:m:s)

  ```








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
