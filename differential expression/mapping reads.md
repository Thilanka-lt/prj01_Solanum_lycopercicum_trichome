# Mapping reads
   
   
### 1. Create a genome index file
 Indices allow the bowtie aligner to narrow down the potential origin of a query sequence within the genome, saving both time and memory.

```ruby
module load GCC/7.3.0-2.30  OpenMPI/3.1.1
module load Bowtie2/2.3.4.2
bowtie2-build S_lycopersicum_chromosomes.fa S_lycopersicum_chromosomes

```

### 2. Read qulity check before trimming
It is essential to see the qulity of RNA seq files. 
```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/do_all_fast_QC_nontrimmed.py
```
This script requries the working directory and print the output to file name you like by doing **<- file name** 
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
                if f.endswith(".fastq"): 
                    print("fastqc -o /mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/fastQC_before_trimming/ -f fastq %s" %(f))
                    
```
Then,
```
python qsub_slurm.py -f submit -c do_all_fast_QC_nontrimmed_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'FastQC' -wd ./
```

### 3. Trimming the reads
Data trimming is an essential step in analysing RNA seq. 
We need to remove any adapter sequences that might be present in the RNA sequence reads.
Also, we are pooling out the low quality bases from the sequnce reads.
```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/do_all_trimming.py 
```
This script requries the working directory and print the output to file name you like by doing **<- file name** 
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
    for f in files:
        if f.endswith("_1.fastq"): 
            print("java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.38.jar PE -threads 4", f, f.replace("_1.fastq", "_2.fastq"), f+".trimP", f+".trimU", f.replace("_1.fastq", "_2.fastq")+".trimP", f.replace("_1.fastq", "_2.fastq")+".trimU", "ILLUMINACLIP:all_PE_adapters.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20")
```
* This creates set of outputs like following
```
java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.38.jar PE -threads 4 SRR2239886_1.fastq SRR2239886_2.fastq SRR2239886_1.fastq.trimP SRR2239886_1.fastq.trimU SRR2239886_2.fastq.trimP SRR2239886_2.fastq.trimU ILLUMINACLIP:all_PE_adapters.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MNILEN:36
```
```diff
- platform and the path - <java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.38.jar>
+ specify the inputs and outputs - <input file_forfard><input_file_reverse><Output_file_forward_paired><Output_file_forward_unpaired><Output_file_reverse_paired><Output_file_reverse_unpaired> 
! ILLUMINACLIP:<fasta_WithAdapters>:<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>
! ILLUMINACLIP:all_PE_adapters.fa:2:30:10   
#   ILLUMINACLIP -  This step is used to find and remove Illumina adapters.
#   all_PE_adapters.fa -  The file containing the adapter sequnces
#   Seed_Mismatches - specifies the maximum mismatch count which will still allow a full match to be performed
#   Palindrome_ClipThreshold: specifies how accurate the match between the two 'adapter ligated' reads must be for PE             palindrome read alignment.
!   Simple_Clip_Threshold: specifies how accurate the match between any adapter etc. sequence must be against a read.
! SLIDINGWINDOW:<windowSize>:<requiredQuality>
! SLIDINGWINDOW:4:20
#   Window_Size: specifies the number of bases to average across
#   Required_Quality: specifies the average quality required.
! MNILEN : Specify the minimum length of the squnce after trimming. 
```

Then,
```
python qsub_slurm.py -f submit -c do_all_trimming_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'Trimmomatic/0.38-Java-1.8.0_162' -wd ./
```
* The trimming results in four output files:
  * paired forward and revers file - Both reads survived the processing
  * unpaired forward and revers - Both reads did not survive the filtering

### 4. Read quality check after trimming

We need to check the read quality of all the paired and non-paired files
```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/do_all_fast_QC_trimmedP.py
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/do_all_fast_QC_trimmedU.py

```
This script requries the working directory and print the output to file name you like by doing **<- file name** 
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
                if f.endswith(".trimP"): 
                    print("fastqc -o /mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/fastQC_after_trimming/ -f fastq %s" %(f))
```

Then,
```
python qsub_slurm.py -f submit -c fastQC_trimmed_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'FastQC' -wd ./
```
We did this for both paired and non-paired files

### Mapping reads to the genome

We can do using two ways.
   * using TopHat
   * using BWA

#### TopHat

Aligns the short reads to the genome using Bowtie. 
The paired ends and non-paired end reads are aligned separately.
You can use the Bowtie indexes
   * when you run the tophat2 you need to run this is either in intel16|intel18
   * Or you can upload each job submission shell script using
```rubi
sbatch --constraint="intel16|intel18" <your slurm script>

```

##### For paired end seqnces

Run the script on;

```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/map_reads_trimU

```

```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
    for f in files:
        if f.endswith("_1.fastq.trimU"): 
            print("tophat2 -p 4 -I 5000 -g 1 -o", "tophat_"+f.replace(".fastq.trimU", ""), "S_lycopersicum_chromosomes", f, f.replace("_1.fastq.trimU", "_2.fastq.trimU"))
```
Then,
```
python qsub_slurm.py -f submit -c do_all_tophat2_cmd.txt -p 4 -nnode 2  -u ranawee1 -w 1200  -m 10 -mo 'GCC/5.4.0-2.26 OpenMPI/1.10.3 TopHat/2.1.1 Bowtie2/2.3.2 SAMtools/1.5' -wd ./
```

   * The results can be found in the same folder

##### For non-paired end seqnces

The pipeline is similar to the paired-end once.


#### BWA

***We are performing BWA as we can compaire our tophat2 results with BWA***

Files can be found in,
```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/BWA_alignments
```


* **First have to do the indexing**
      * Copy genome to this folder (Once you create the index you can delete it)
      * then,
```
module load GCC/5.4.0-2.26  OpenMPI/1.10.3 BWA/0.7.17

bwa index S_lycopersicum_chromosomes.fa 
```
* Then you can map paired end reads using "do_all_BWA_alignment.py" in 
```
/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/BWA_alignments/paired_reads
```
```ruby

import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
            if f.endswith("_1.fastq.trimP"): 
                print("bwa mem /mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/BWA_alignments/index_files/S_lycopersicum_chromosomes.fa", f, f.replace("_1.fastq.trimP", "_2.fastq.trimP"), ">", f.replace("_1.fastq.trimP", ".sam"))

```

      * once you map you can purge the paired-end reads.

Then,

```
python qsub_slurm.py -f submit -c do_all_BWA_alignment_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'GCC/5.4.0-2.26  OpenMPI/1.10.3 BWA/0.7.17' -wd ./
```

* **once you have sam files we need to covert it in to .bam file where .bam file allocate less memory and easy to work with**


   * we can run this script to creat a script that converts sam > bam (do_all_sam_to_bam_cmd.txt)
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
                if f.endswith(".sam"):
                        print("samtools view -S -b", f, ">", f.replace(".sam", ".bam"))

```

Then,

```
 python qsub_slurm.py -f submit -c do_all_sam_to_bam_cmd.txt -p 4 -nnode 2  -u ranawee1 -w 1200  -m 10 -mo 'SAMtools/1.5' -wd ./
```
