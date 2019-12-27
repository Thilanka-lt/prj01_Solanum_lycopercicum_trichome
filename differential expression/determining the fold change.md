## Get the read counts

#### Sorting the .bam files

We can get the raw counts using HTseq.
Usually you can work with .sam files. But now we can also use .bam files as the input. But we need to sort .bam files first in the most cases if the data are paired ended..
Sorting make sure all the alignmnts appire next to each other.
We can use the sorted .bam files used in cufflinks in BWA and have to sort .bam files used in tophat.

I copied all the tophat .bam files to /mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/trimming_with_minlen_36/Htseq/tophat
Then executed the scipt do_all_BAM_sort_python.txt
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
                if f.endswith(".bam"):
                        print("samtools sort  -n -O bam", f,"-o", f.replace(".bam","_sorted.bam"))

```
 then,
 ```
 python qsub_slurm.py -f submit -c do_all_BAM_sort_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'GCC/5.4.0-2.26  OpenMPI/1.10.3 SAMtools/1.5' -wd .
 ```

#### running the HTseq

I used the same procedure for BWA and tophat alignments.
First run **do_all_HTseq.py** script.
```ruby
import os, sys
path = sys.argv[1]
os.chdir(path)
for root, dirs, files in os.walk(path):
        for f in files:
                if f.endswith("sorted.bam"):
                        print("htseq-count --format=bam -m union -s no -t gene -i ID", f,  "/mnt/home/ranawee1/01_Solanum_lycopercicum_trichome/difrential_expression/trimming_with_minlen_36/cufflinks/ITAG4.0_gene_models.gff >", "HT_seq_"+f.replace("_P_sorted.bam", ""))

```

then,
```
python qsub_slurm.py -f submit -c do_all_HTseq_cmd.txt -p 4 -u ranawee1 -w 1200  -m 10 -mo 'GCC/6.4.0-2.28 OpenMPI/2.1.2 ScaLAPACK/2.0.2-OpenBLAS-0.2.20, Boost/1.67.0, Python/3.6.4, ScaLAPACK/2.0.2-OpenBLAS-0.2.20, Boost/1.67.0, Python/3.6.4' -wd ./
```
