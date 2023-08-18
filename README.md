# TimingExerciseTriggerHATS2023
Short timing exercise for Trigger HATS@LPC 2023


## Prerequisites

A CERN account with access to lxplus - that's it!

## Instructions

### Exercise 1: CPU/GPU Timing measurements without creation of a CMSSW environment

1. Log in to lxplus and clone [the timing repository](https://gitlab.cern.ch/cms-tsg/steam/timing) somewhere (e.g. in your EOS space)

```bash
git clone https://gitlab.cern.ch/cms-tsg/steam/timing.git
cd timing
```

2. Submit a timing job to the timing machine using CMSSW_13_2_0, the GRun menu V152 and the default dataset on the timing machine. Additionally, tell the program to merge in pull request [#42534](https://github.com/cms-sw/cmssw/pull/42534) Also use a user-specified tag to better identify your job later in the queue.


```bash
python3 submit.py /dev/CMSSW_13_0_0/GRun/V152 --cmssw CMSSW_13_2_0 --pull-requests 42534 --tag YOUR_TAG_HERE
```

3. Check the status of your job using the `job_manager.py` script.

```bash
python3 job_manager.py
```

4. Re-submit your job using the --rerun option, followed by the job ID of the first submitted job. This will re-submit the first job with the exact same parameters and can be useful if you want to re-run a job multiple times to get an idea of the variance of the timing measurements. This also leads to the program re-using the same CMSSW area as before on the timing machine, so it saves up some disk space there. Also make sure to add a new `--tag` to your job so you can distinguish the two in the job queue

```bash 
python3 submit.py --rerun JOB_ID --tag YOUR_NEW_TAG_HERE
```

5. Remove the recently added job from the queue using the `job_manager.py` script and the `--rm` option.

```bash
python3 job_manager.py --rm JOB_ID_OF_RESUBMITTED_JOB
```

NOTE: It is currently not possible to cancel an already running job. Only queued jobs can be cancelled.

6. Submit another job with the same settings as before, but now only using the CPUs by adding the `--cpu-only` option. This will run the same job, but only on the CPUs of the timing machine. This is useful to compare the performance of the CPUs and GPUs.

```bash
python3 submit.py /dev/CMSSW_13_0_0/GRun/V152 --cmssw CMSSW_13_2_0 --cpu-only --pull-requests 42534 --tag YOUR_CPU_JOB_TAG_HERE
```

7. Once your jobs have finished, get the reports for one of the jobs using the `job_manager.py` script and the `--report` option. This will download a `tar.gz` file containing output and error files for :

- the creation of the CMSSW environment on the timing machine (i.e. the `scram proj`/`cmsrel` step)
- the building of the CMSSW environment including the merging of provided pull requests etc. (i.e. the `scram b` step)
- the benchmarking script (found in the `run_benchmark.py` file) which initializes and controls the settings of the actual timing measurements on multiple CPUs/GPUs using the [`patatrack-scripts`` repo](https://github.com/cms-patatrack/patatrack-scripts)
- the final `cmsRun` of the timing job

**NOTE:** All of these steps are taken care of by the server for you as a user and the reports are just helpful tools when occasionally some measurement is crashing.

```bash
python3 job_manager.py --report JOB_ID
```

8. Investigate the results using the timing GUI

Use the timing GUI to compare your CPU-only job with your GPU job. The timing GUI can be accessed from [this Link](https://timing-gui-tsg-steam.app.cern.ch/). Once you arrive at the GUI, klick the "open" button and find your CERN username, then klick the little arrow/triangle next to the box to get a drop-down list which contains all your finished measurements. Click the boxes on the GPU and the CPU ones and hit the "OK" button.

The results take some time to load. Once you see your results, klick on the "TIMING" tab to see the different timing distributions. You should be able to identify the fast peaks and tracking/PF tails that were discussed in the slides.

In the lower panel, you see the timing of the individual paths of the menu. Klick on one of the job ID names to sort the path timing in ascending/descending order for that respective measurement. Investigate which paths are the fastest and the most time-consuming ones and if it checks out with what we discussed in the slides. You can also klick on a path/row to see the timing distribution of that path (left panel) and the average timing of each module inside that path (right panel). For the right panel, it's sometimes difficult to see which module you're looking at, so you can also hover over the "bins" and some information for that bin will pop up, including the module name.

Finally, let's find the path with the largest timing *difference* between CPU and GPU measurements. To do this, switch on the "Display Diff" button in the top right of the lower panel. This will cause the column of the second measurement to show the timing difference with the column of the first measurement. Note that this works also when more than two measurements are selected for comparison: The first selected measurement column is always the "reference" and other columns show the difference relative to it. Now you can sort ascending/descending by timing difference by clicking the job name of the second measurement. Which paths have the largest timing difference? Is this what you would expect? You can also look at the timing distributions of the paths again by clicking on the path name again.

### Exercise 2 (Bonus): CPU/GPU Timing measurements *with* creation of CMSSW area

It is also possible to submit a timing job with a previously created CMSSW area. When you do this, your area will get "cloned" to the timing machine and it will run a measurement using the cloned area. Since the arealess submission already allows for a lot of flexibility, this is not really necessary and **the arealess submission is the recommended way to submit jobs**. However, if you run some very specialized expert workflows that for some reason require additional tinkering with the CMSSW area that is not covered by the timing code, it can sometimes still be useful.

1. Create a CMSSW area on lxplus and build it using the GRun menu V152 and the default dataset on the timing machine. Additionally, tell the program to merge in pull request [#42534](https://github.com/cms-sw/cmssw/pull/42534)

```bash
cmsrel CMSSW_13_2_0
cd CMSSW_13_2_0/src
cmsenv
git cms-init
git cms-merge-topic 42534
scram build -j 4
```

2. Download the same GRun menu as in Exercise 1 using the `hltGetConfiguration` command. When you do this outside this exercise, make sure you choose the correct globaltag and l1 menu for your use case. For this exercise you can copy/paste the command below.

```bash
hltGetConfiguration /dev/CMSSW_13_0_0/GRun/V152 --globaltag 132X_dataRun3_HLT_v2 --data --process TIMING --full --offline --output minimal --type GRun --max-events 20000 --era Run3 --timing --l1 L1Menu_Collisions2023_v1_2_0-d1_xml > hlt.py
```
3. Clone the timing repository.

```bash
git clone https://gitlab.cern.ch/cms-tsg/steam/timing.git
```

4. Submit your config to the timing server. Many of the options that we needed in the arealess submission are not needed anymore, since they are already specified in the config.

```bash
python3 ./timing/submit.py hlt.py --tag YOUR_NEW_TAG_HERE
```

5. Again you can investigate the job using the `job_manger.py` script.

```bash
python3 ./timing/job_manager.py
```

6. Once the job is done, you can investigate the results in the timing GUI. For example, you can check if your result of the arealess submission coincides with your result of the submission with an area (a variance of about 1-3% is normal).


## Useful links

- [Timing GUI](https://timing-gui-tsg-steam.app.cern.ch/)
- [Timing repository](https://gitlab.cern.ch/cms-tsg/steam/timing)
- [Timing studies twiki](https://twiki.cern.ch/twiki/bin/viewauth/CMS/TriggerStudiesTiming)
- [Timing reports twiki](https://twiki.cern.ch/twiki/bin/viewauth/CMS/HLTCpuTimingReports2023)