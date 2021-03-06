Due to the increasing file size of observations coming from Jira’s API,
the size of the dataset can grow precipitously as a function of the
amount of projects the user is interested in querying. Right now, a
years worth of changes can grow to about 8 gigs.

It’s important to note, strictly speaking, not all the changes are
necessary to create estimates for the time in development for a
story/projects

Additional development is needed to filter our the dataset based on only
the changes needed. If done in the student cluster, a compute job will
need to be queued to correctly allocate the amount of memory needed to
run the tasks on the large dataset.

Currently, the architecture of the data extraction functions load one
dataframe into memory and add the new observations accordingly; however,
for redundancy, it may be better to write each dataset as a seperate
file in the directory of interest. Later, a function will need to be
created to append all these files into one dataframe.

The R script which is needed to filter these observations and needs to
be automated is given:

``` r
# adjust variable names as needed

# we need to filter on the swimlane of interest:

library(data.table)

original_observations <- fread("myobservations.csv")

original_observations[ fromString %in% c("In Progress", "Peer Review", "QA Review") ]

# these swimlanes focus specifically on calculating development time

fwrite(original_observations, "filtered_observations")
```

This rscript will need to be automated using the compute job *sbatch*
function in the clsuter. An example bash script for submission is given
below. You must allocate enough memory to load the file size into
memory.

``` bash
#!/bin/bash

#SBATCH --nodes=1

#SBATCH --ntasks-per-node=10

#SBATCH --mem=32G

#SBATCH --time=1:32:15

#SBATCH --job-name=this_is_my_job

module purge

module add apps/R/3.5.2

Rscript myscript.R

module purge
```
