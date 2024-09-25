# Directories

Team working directory
```
/lustre/scratch126/tol/teams/jaron
```
Team software directory
```
/software/team360/
```
DTOL genome directory
```
/lustre/scratch124/tol/projects/darwin/data
```

# Farm commands

Print cluster you are on
```
lsid 
```
Show lsf groups
```
bugroup | grep "\b${USER}\b" | cut -d" " -f1

# Alias
show_my_lsf_groups
```
Submit a job
```
bsub /path/to/script
	-o path/to/output.%J File to write standard output (J appends job ID number)
	-e path/to/error.%J File to write standard error (J appends job ID number)
	-E pre-execution job
	-I interactive job
	-J Set jobname
	-n number of cores
	-M memory
	-q select queue
	-R reserve resource
	    - Memory: '-R "select[mem>20000] rusage[mem=20000] span[hosts=1]"'
	-G user group
        	- Add to .bashrc: 'export LSB_DEFAULTGROUP=your-group'
        
# Interactive example
bsub -Is bash
bsub -G team360 -Is -n 16 -M 10240 -R "select[mem>10240] rusage[mem=10240]" bash -l 
	
# Job array example
bsub -J “jobname[1-100]%10” -o output.%J.%I /my/script.sh
```
List jobs
```
bjobs
	-d see finished jobs within past 1h
	-uall See all jobs (default own jobs)
	
# Runtime estimation
bjobs -l 131481 | grep -C2 ESTIMATION:
```
List job history
```
bhist
```
Kill job 
```
bkill <ID>
	<0> kills all jobs
```
List queues and number of jobs in system
```
bqueues
	-l includes details and priority
```
Show status of worker nodes
```
bhosts
	-w show stats
	-l show FULL host 
```

# Queues
- Normal 
	- For most jobs
	- Maximum time 12h

- Long
	- Maximum time 48h

- Basement
	- Maximum time 30 days
	- 300 job limit per user

- Hugemem etc.
	- 512GB/1TB memory machines
	- Locked to certain users who need it

- Yesterday
	- Very fast
	- 7 per user

- Small
	- Short jobs
	- Batches 10 together and runs sequentially

- Parallel
	- Spans multiple machines


# Example wrapper script
```
#! /bin/bash
#BSUB -o cellbender-%J-output.log
#BSUB -e cellbender-%J-error.log 
#BSUB -q gpu-basement
#BSUB -n 4  
#BSUB -M20000
#BSUB -R "select[mem>20000] rusage[mem=20000] span[hosts=1]"
#BSUB -gpu "mode=shared:j_exclusive=no:gmem=6000:num=1"

#input file (samples will be read from this csv)
SAMPLES_CSV="/nfs/cellgeni/prete/cellbender/samples.csv"

# add singularity to the path
PATH="/software/singularity-v3.5.3/bin:$PATH"

# set the path to the image we want to use
CELLBENDER_IMAGE="/nfs/cellgeni/singularity/images/cellbender-0.2.0.sif"

# path to output folder (samples will have a folder inside this one)
OUTPUT_FOLDER="/lustre/scratch117/cellgen/team205/kk18/HeartAtlas/CellBender/outs"

# create output folder if it does not exist
if [[ ! -d "${OUTPUT_FOLDER}" ]]; then
    mkdir -p "${OUTPUT_FOLDER}"
fi

# 'sed 1d' skips header after that read using IFS for each row and store columns as vars
sed 1d $SAMPLES_CSV | while IFS=, read -r SAMPLE_ID SANGER_HARVARD CELL_NUCLEI VER_10X EXPECTED_CELLS INPUT_FILE
do
    echo "================================"
    echo "- Sample: $SAMPLE_ID"
    echo "- Sample Source: $SANGER_HARVARD"
    echo "- Sample type: $CELL_NUCLEI"
    echo "- 10x version: $VER_10X"
    echo "- Target cells: $TGT_NUMBER_CELLS"
    echo "- Input path: $INPUT_FILE"

    # -------
    TOTAL_DROPLETS=15000
    EPOCS=150

    # create folder for this sample output
    mkdir -p "${OUTPUT_FOLDER}/${SAMPLE_ID}"

    # run remove-background
    singularity run --nv --bind /nfs,/lustre $CELLBENDER_IMAGE cellbender remove-background \
                --input "$INPUT_FILE" \
                --output "${OUTPUT_FOLDER}/${SAMPLE_ID}/${SAMPLE_ID}_cellbender_out.h5" \
                --epochs $EPOCS \
		        --cuda \
                --expected-cells $EXPECTED_CELLS \
                --total-droplets-included $TOTAL_DROPLETS
done
```

# Useful links
- [Farm FAQ](https://ssg-confluence.internal.sanger.ac.uk/display/FARM)
- [LSF command reference](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=reference-command)
- [LSF quick start guide](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=started-quick-start-guid)
- [Ganglia](https://ganglia.internal.sanger.ac.uk/ganglia/)
- [Metrics](https://metrics.internal.sanger.ac.uk/)
- [Analytics](http://analytics01.internal.sanger.ac.uk)
- [ToLID](https://id.tol.sanger.ac.uk/search)
- [ToLQC](https://tolqc.cog.sanger.ac.uk)
