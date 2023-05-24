# MSc-Code-Slurm
Code I used for the bioinformatics section of my Masters project. DNA sequencing used Angiosperm353. Coding uses Ubuntu, Slurm, Trimmomatic and Hybpiper.
Sections with stars will need to be replaced with your names/files.
DIR/DIR will need to be replaces with the directory for that file.
You will need 2 tabs open on Ubuntu, one for your computer's directories and one for the slurm's directories 
If pasting script in Notepad ++ remember to remove \r.
Software to download:
Conda
Hybpiper (using conda)
Trimmomatic downloaded later, along with someother softwares

To access slurm, type password when promt
```
ssh *email for log in*
```

To be added to a node
```
srsh --partition=short --cpus-per-task=2 --mem=2G
```

To input the files with the sequences
```
cp -r *location of sequences* .
```

Changing the title of files and directories
```
mv old_title new_title
```

Move script in to the same folder as what I'm working on 
```
scp *script-name*.sh DIR/DIR/apps
```

then run it
```
sbatch *script-name*.sh
```

To copy a file in to the server, this one is the names of the DNA sequence files. This should just have the unique prefixes of all your documents ex. 
Sample_1
Sample_2
Sample_3
NOT
Sample_1.txt
OR
Sample
Sample
```
scp *sample_name_file*.txt DIR/DIR/pwd
```

For putting the fastqc-slurm script file in the server for slurm
```
scp *fastqc-script*.sh DIR/DIR/apps
```

For running the fastqc-slurm code in the terminal
```
sbatch DIR/DIR/apps/*fastqc-slurm*.sh
```

Use the tab with your computer's directories, the html outcomes into your local hard drive(.) 
```
scp DIR/DIR/pwd/*.html .
```
--------------------------------------------------------------------------------------------------------------------------
# Next, move to Trimmomatic

Downloading Paired-ended Format:
```
java -jar DIR/DIR/apps/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 [-trimlog <logFile>] >] [-basein "$name"_R1_001.fastq "$name"_R2_001.fastq] [-baseout "$name"_output_forward_paired.fq.gz "$name"_output_forward_unpaired.fq.gz "$name"_output_reverse_paired.fq.gz "$name"_output_reverse_unpaired.fq.gz ILLUMINACLIP:~software/Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa:1:30:7:2:true MAXINFO:40:0.85 MINLEN:36); done
```
-------------------------------------------------
SETTINGS
ILLUMINACLIP:~/software/Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa:1:30:7:2:true
MAXINFO:40:0.85 
MINLEN:36
TOPHRED 33 

-----------------------
## Trimmomatic code:
```
#!/bin/bash
#
#SBATCH --chdir=/PATH/PATH/directory
#SBATCH --job-name=trimmomatic
#SBATCH --partition=medium
#SBATCH --array=#number of samples ex. 1-7#
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --mail-user=*email*
#SBATCH --mail-type=END,FAIL

echo $SLURM_ARRAY_TASK_ID

name=$(awk -v lineid=$SLURM_ARRAY_TASK_ID 'NR==lineid{print;exit}' /DIR/DIR/*sample_name_file*.txt)

echo $name

java -jar /DIR/DIR/apps/conda/conda-meta/trimmomatic-0.39 PE -phred33 /DIR/DIR/*sample_name_file*.txt "$name"_R1_001.fastq /DIR/DIR/*sample_name_file*.txt "$name"_R2_001.fastq "$name"_R1_001_P.fastq "$name"R1_001_UP.fastq "$name"_R2_001_P.fastq "$name"_R2_001_UP.fastq ILLUMINACLIP:~/apps/Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa:1:30:7:2:true MAXINFO:40:0.85 MINLEN:36)
```

To move the script:
```
sbatch *trimmomatic-script*.sh
```

For putting the trimmomatic-slurm script file in the server for slurm (will be done in your computer's account rather than the slurm tab)
 ```
 scp *trimmomatic-slurm*.sh DIR/DIR/apps
```

For running the trimmomatic script in the terminal
```
sbatch /DIR/DIR/apps/*trimmomatic-script*.sh
```

Checking the quality
```
sbatch /DIR/apps/*fastqc-slurm*.sh
```

Use the tab with your computer's directories, the html outcomes into your local hard drive(.) 
```
scp DIR/DIR/*.html .
```

----------------------------------------------------------
# Hybpiper

```
#!/bin/bash
#
#SBATCH --chdir=/home/DIR/DIR
#SBATCH --job-name=hybpiper
#SBATCH --partition=medium      
#SBATCH --array=*number of samples*   
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=8G
#SBATCH --mail-user=*email*
#SBATCH --mail-type=END,FAIL

echo $SLURM_ARRAY_TASK_ID

name=$(awk -v lineid=$SLURM_ARRAY_TASK_ID 'NR==lineid{print;exit}' /DIR/DIR/*sample_name_file*.txt)

echo $name


cd $TMPDIR

source activate hybpiper

hybpiper assemble -r /home/DIR/"$name"_R*_001_P.fastq -t_aa /home/DIR/*sample_name_file*.txt --prefix "$name" --timeout_assemble 4000 --timeout_exonerate_contigs 4000 --cpu 8 --run_intronerate

# --cov_cutoff 3

mv $name /home/DIR/HP_out

conda deactivate
```
---------------------
Move code to apps
```
scp *hybpiper-script*.sh /home/DIR/apps
```

Running script
```
sbatch /home/DIR/apps/hybpiper-script.sh
```

----------------------------------------
# Post-assembly and extraction pipeline 
Software to install using conda:
Mafft
CIAlign
Optrimal
TAPER
IQTREE
raxml-ng
weighted ASTRAL (or ASTRAL III if you do not figure out ASTRAL-w)

## Supercontig

Stats
```
#!/bin/bash
#
#SBATCH --chdir=/home/DIR/HP_out
#SBATCH --job-name=hybpiper
#SBATCH --partition=medium      
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=8G
#SBATCH --mail-user=*email*
#SBATCH --mail-type=END,FAIL

source activate hybpiper


hybpiper stats -t_aa /home/DIR/*template.fasta* supercontig /home/DIR/*sample_name_file*.txt --stats_filename *supercontig-stats* --seq_lengths_filename *supercontig-lengths*

hybpiper stats -t_aa /home/DIR/*template.fasta* gene /home/DIR/*sample_name_file*.txt --stats_filename gene-stats --seq_lengths_filename *gene-lengths*

hybpiper recovery_heatmap *supercontig-lengths*.tsv

hybpiper recovery_heatmap *gene-lengths*.tsv

hybpiper retrieve_sequences -t_aa /home/DIR/*template.fasta* dna --sample_names /home/DIR/*sample_name_file*.txt --fasta_dir genes

hybpiper retrieve_sequences -t_aa /home/DIR/*template.fasta* dna --sample_names /home/DIR/*sample_name_file*.txt --fasta_dir supercontigs

conda deactivate
```
-------
Sending script to your slurm
```
scp *stats-script*.sh /home/DIR/apps
```

Running script
```
batch /DIR/DIR/apps/stats-slurm.sh
```


