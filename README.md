# AlphaFold Singularity Container

This repo provides definition files to build a singularity container of AlphaFold v2 
(https://github.com/deepmind/alphafold) that will run and be easy to invoke in the NMRbox
environment.

Build instructions from [non-docker setting](https://github.com/kalininalab/alphafold_non_docker) by kalininalab were used.

## Setup

Clone repository into your home directory and `cd` into the cloned folder. 

## Build container

If in CHTC, we strongly recommend using an interactive job to build. To start the interactive job, 
use the `build.sub` file in this repository: 

```
condor_submit -i build.sub
```

Once you are ready to build, run the following commands: 
```
export APPTAINER_CACHEDIR=$PWD
export TMPDIR=$PWD
# build base container
apptainer build base.sif base.def
# build alphafold container
apptainer build alphafold.sif alphafold.def
# if running in an interactive job in CHTC:
rm base.sif
mv alphafold.sif /staging/$USER
```
After the build is done, exit the interactive job if you are in one. 

## Run AlphaFold Job

To actually run alphafold, upload your protein files and submit jobs 
with a submit file that looks something like this. 

```
universe = container
container_image = file:///staging/YOUR_USERNAME/alphafold.sif
transfer_executable = false

executable = /opt/monomer.sh
# replace with multimer.sh if applicable
arguments = /gpulab_data/alphafold FASTA_file (optional) (optional)

transfer_input_files = FASTA_file

request_gpus = 1
+WantGPULab = true
requirements = (HasGpulabData == true)
# use if you want a certain amount of GPU memory (in MB)
# require_gpus = (GlobalMemoryMB > 45000)
# change length of job as needed to "medium" or "long"
+GPUJobLength = "short"

# change these as needed
request_cpus = 1
request_memory = 64GB
request_disk = 32GB

queue 1
```


### Notes

* The alphafold.py run script has no requirements and should run in vanilla python 3.8.
* The run script allows customizing the database location and max_template_date. Call with `-h` to see usage information.
* By default, this uses the `monomer` model for monomers and the `multimer` model for multimers,
  and uses the `full_dbs` option for better quality results. For more details, see https://github.com/deepmind/alphafold#running-alphafold
