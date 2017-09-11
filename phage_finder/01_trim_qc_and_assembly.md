### trim and remove short reads

#### using sickle pe mode
```
/mnt/raid1/wchen/bin/sickle pe \
	-f ../../Data_Zheng/Lasiopodomys_brandtii/350bp/muscle_2_1_clean.fq  \
	-r ../../Data_Zheng/Lasiopodomys_brandtii/350bp/muscle_2_2_clean.fq \
	-o lbr_muscle_clean_trimmed_1.fq \
	-p lbr_muscle_clean_trimmed_2.fq \
	-s trimmed_singles_file.fastq \
	-t sanger -q 30 -l 15
```

#### using trimmomatic
```
java -jar /home/wchen/01Trimmomatic/trimmomatic-0.35.jar PE  \
	-threads 10 \
	-trimlog ${infile}.trim.log \
	muscle_${infile}_clean_trimmed_1.fq \
	muscle_${infile}_clean_trimmed_2.fq \
	muscle_${infile}_clean_trimmed_clean_paired_1.fq \
	muscle_${infile}_clean_trimmed_clean_unpaired_1.fq \
	muscle_${infile}_clean_trimmed_clean_paired_2.fq \
	muscle_${infile}_clean_trimmed_clean_unpaired_2.fq \
	ILLUMINACLIP:/home/wchen/01Trimmomatic/adapters/TruSeq2-PE.fa:2:30:10 \
	SLIDINGWINDOW:120:30 \
	MINLEN:100 \
	TRAILING:30 \
	AVGQUAL:30
```

#### use SLURM to do batch jobs
**1. prepare template file**
file name: trimmomatic_jobs_template_slurm.sh
```
#!/bin/bash
#SBATCH --cpus-per-task=10        ## note: please change this accordingly --
#SBATCH -o slurm.%N.%j.out        # STDOUT
#SBATCH -e slurm.%N.%j.err        # STDERR

## -- do sickle --
java -jar /home/wchen/01Trimmomatic/trimmomatic-0.35.jar PE  \
	-threads 10 \
	-trimlog ${infile}.trim.log \
	muscle_${infile}_clean_trimmed_1.fq \
	muscle_${infile}_clean_trimmed_2.fq \
	muscle_${infile}_clean_trimmed_clean_paired_1.fq \
	muscle_${infile}_clean_trimmed_clean_unpaired_1.fq \
	muscle_${infile}_clean_trimmed_clean_paired_2.fq \
	muscle_${infile}_clean_trimmed_clean_unpaired_2.fq \
	ILLUMINACLIP:/home/wchen/01Trimmomatic/adapters/TruSeq2-PE.fa:2:30:10 \
	SLIDINGWINDOW:120:30 \
	MINLEN:100 \
	TRAILING:30 \
	AVGQUAL:30
```

**2. prepare a list files to be trimmed**
file name: fastq_list_file.lst
```
DES00209_L3_1_clean.fq.gz
DES00209_L3_2_clean.fq.gz
...
DES00212_L5_1_clean.fq.gz
DES00212_L5_2_clean.fq.gz
DES00212_L6_1_clean.fq.gz
DES00212_L6_2_clean.fq.gz
```

**3. prepare another script file to submit batch jobs**
file name: submitter.sh
```
#!/usr/bin/bash
filename="$1"
while read -r line
do
    sbatch -D `pwd` --job-name=$line  --export=indir="/mnt/raid1/Data_Zheng/rawdata/data/small/cleandata/",infile=$line,outdir="small" trimmomatic_jobs_template_slurm.sh
done < "$filename"
```

**4. finally submit batch jobs**
```
sh submitter.sh fastq_list_file.lst
```
