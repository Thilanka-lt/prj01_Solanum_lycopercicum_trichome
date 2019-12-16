
## 1. Download the raw data

The expression data are dowloaded from the NCBI SRA database.

Species | Tissue | Accession
-------- |-------|---------
*Solanum lycopersicum*| Trichome only |SRX1182190  
*Solanum lycopersicum*| Trichome only |SRX1182189
*Solanum lycopersicum*| Shaved stem |SRX1182188 
*Solanum lycopersicum*| Shaved stem |SRX1182187
*Solanum lycopersicum*| Stem + Trichome |SRX1182186 
*Solanum lycopersicum*| Stem + Trichome |SRX1182185
 

* Use the **qsub_slurm.py** to submit this as job.
	fastq-dump SRX3562827 ---> all the accesions in a get_data.sh
* Also can use the script below and submit it through the qsub_slurm.py
~~~
# this code uses two arguments. The first on is to acess the file with all the SRR accession and the secon one is the name of your output file.

/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/fastq_dump.py

#generate_dump_seq.py code

import sys
data_file = open(sys.argv[1], "r").readlines()
write_file = open(sys.argv[2], "w")
for i in data_file:
    write_file.write("fastq-dump --split-3 %s" %(i))
write_file.close

# when submitting the looks like we have to load noyt only SRA-Toolkit but also Perl/5.28.1.

python qsub_slurm.py -f submit -c full_dump_file.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'SRA-Toolkit Perl/5.28.1' -wd ./
~~~

* Download **1- genome dataset**,  **2- Protien datasets** and **3- Annotatin** from ftp://ftp.solgenomics.net/genomes/Solanum_lycopersicum/Heinz1706/
```
#Genome
wget ftp://ftp.solgenomics.net/genomes/Solanum_lycopersicum/Heinz1706/assembly/build_4.00/S_lycopersicum_chromosomes.4.00.fa

#proteome
wget ftp://ftp.solgenomics.net/genomes/Solanum_lycopersicum/Heinz1706/annotation/ITAG4.0_release/ITAG4.0_proteins.fasta

#Annotation
wget 
```







