## Maker on Eve


* [maker2 publication](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-12-491)

* [maker2 official homepage](http://www.yandell-lab.org/software/maker.html)


# Running maker2 on Eve using Singularity and mpi support

* this Readme is based on the maker2 image from docker://stephanholgerdrukewitz/maker2:latest
* other maker2 images can be used but errors can appear caused by different path setting etc.


### Prepare your maker run

run

```
singularity run docker://stephanholgerdrukewitz/maker2:latest maker -CTL

```
* this creates the maker control files in your current working directory
* don't do it in your home directory
* creating a projectfolder in your workdirectory makes sense, maker creates a lot of output and you don need everything
* after the analyses you can just use mv and move the filoes of intrest in your data directory

* edit the maker_bopts.ctl file to point to your data
* singularity mounts several directories per default (home, tmp and current working directory) but not the your data directory
* thats why we need to mount the data directory in the submit script using :
```
-B /gpfs1/data/YourWorkingroupName:/data

```
* this way you mount your data drive to the "/data" within the singularity container




### Submit script
```
#!/bin/bash

#$ -S /bin/bash

#$ -o /work/$USER/$JOB_NAME-$JOB_ID.log
#$ -j y

cd ${1}
export SINGULARITYENV_AUGUSTUS_CONFIG_PATH=${1}/config # if your work with non model organisms you need to re define were the augustus config folder is located, this can not be included in the image because the parameters change after training rounds
singularity run -B /gpfs1/data/YourWorkingroupName:/data docker://stephanholgerdrukewitz/maker2:latest mpiexec -np ${NSLOTS:-1} maker

```

* submit the job with

```
qsub -N maker -l h_vmem=8G,h_rt=100:00:00 -pe smp 20-28 maker.sh ${the folder in your working directory where your maker files are located}

```
