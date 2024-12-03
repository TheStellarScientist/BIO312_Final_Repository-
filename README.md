# BIO312 Final Repository
This document contains all the commands used to create my project. There are two parts. The commands used in Labs 03 - 08 (excluding 07) which were not used for my project and then the actual commands I used for my project. 

## Part 1: The Labs 
The following is just the commands used. For the actual labs and detiled information please use the following links:
[LAB03](https://github.com/Bio312/lab03-TheStellarScientist)
[LAB04](https://github.com/Bio312/lab04-TheStellarScientist)
[LAB05](https://github.com/Bio312/lab05-TheStellarScientist)
[LAB06](https://github.com/Bio312/lab06-TheStellarScientist)
[LAB08](https://github.com/Bio312/lab08-TheStellarScientist)


### Lab 03: Finding homologs with BLAST KEY
The purpose of this lab is to conduct a BLAST search on a starting protein to find homologs and get a general picture of size of the gene family. 

```commands
List of commands used for PONPON (excluding less commands used to view files)
```
The command below is used to create a directory in Lab 03 called PONPON where all the relevant files and operations for this lab will be stored and conducted. We will use this for all future labs.
```1
1. mkdir ~/lab03-$MYGIT/PONPON
```
Next we downloaded all the files in NCBI partaining to NP_000437.3.
```2
2. ncbi-acc-download -F fasta -m protein "NP_000437.3"
```
With the information we just downloaded were able to run a BLAST search. This command specifies the query's fasta file NP_000437.3.fa and the output of the blast results which we called PONPON.blastp.typical.out.
```3
3. blastp -db ../allprotein.fas -query NP_000437.3.fa -outfmt 0 -max_hsps 1 -out PONPON.blastp.typical.out
```
We then requested a different format of the output for easier processing with human eyes. This step is completely optional but we reccomend it. [Here](http://www.metagenomics.wiki/tools/blast/blastn-output-format-6) is a link for further reading about output formats.
```4
4. blastp -db ../allprotein.fas -query NP_000437.3.fa  -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident
 stitle"  -max_hsps 1 -out PONPON.blastp.detail.out
```
In this next step we used awk to filter the BLAST results for e-value below 1e-30. You can of course choose whatever e-value you believe is best but we used this one to ensure a small pool. 
```5
5. awk '{if ($6< 1e-30)print $1 }' PONPON.blastp.detail.out > PONPON.blastp.detail.filtered.out
```
After this you can sort the results in a way that makes sense to you but we chose to use the following command to sort our results alphabetically. 

```6
6. grep -o -E "^[A-Z]\.[a-z]+" PONPON.blastp.detail.filtered.out  | sort | uniq -c
```

### Lab 04: Gene family sequence alignment
In this lab we aligned the genes we found in our BlAST search from the previous lab. Once again we created a diretory to create a place for all the files.

```commands
List of commands used for PONPON (excluding less commands used to view files and other non-critical commands)
```
Using seqkit we grabbed the the filtered results from Lab 03 and transfered them to Lab 04 in preperation for alignment.
```1
1.  seqkit grep --pattern-file ~/lab03-$MYGIT/PONPON/PONPON.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit gr
ep -v -p "carpio" > ~/lab04-$MYGIT/PONPON/PONPON.homologs.fas
```
Next we used MUSCLE to align all the sequences retrieved from last lab.
```2
2. muscle -align ~/lab04-$MYGIT/PONPON/PONPON.homologs.fas -output ~/lab04-$MYGIT/PONPON/PONPON.homologs.al.fas
```
We then viewed the alignment and converted it into a pdf for easy viewing. There are many ways you can do this but we provided the commands we used below.

View:
```flow
A. alv -kli  ~/lab04-$MYGIT/globins/globins.homologs.al.fas | less -RS
B. alv -kli --majority ~/lab04-$MYGIT/globins/globins.homologs.al.fas | less -RS
```
Convert to pdf using an R-script
```glow
 Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R  ~/lab04-$MYGIT/globins/globins.homologs.al.fas
```
Next we looked at the alignment statistics (percent identity, alignment length etc) using AlignBuddy and Tcoffee. We used general commands but here are some examples pasted below:
```commands
length of the alignment:
alignbuddy  -al  ~/lab04-$MYGIT/globins/globins.homologs.al.fas

length after removing gaps:
alignbuddy -trm all  ~/lab04-$MYGIT/globins/globins.homologs.al.fas | alignbuddy  -al

length after removing conserved position:
alignbuddy -dinv 'ambig' ~/lab04-$MYGIT/globins/globins.homologs.al.fas | alignbuddy  -al

percent identity:
 alignbuddy -pi ~/lab04-$MYGIT/globins/globins.homologs.al.fas | awk ' (NR>2)  { for (i=2;i<=NF  ;i++){ sum+=$i;num++} }
     END{ print(100*sum/num) } '

percent identity but with tcoffee:
t_coffee -other_pg seq_reformat -in ~/lab04-$MYGIT/globins/globins.homologs.al.fas -output sim
```

### Lab 05: Gene Family Phylogeny using IQ-TREE
This lab we used the alignments from LAB04 and IQ-TREE to construct phylogenetic trees. 
We as always started by creating a new directory and relocating there.

 First we transffered the relevant information from Lab 04 to Lab 05. This command just removes duplicate tags before copying the data.
```commands
1. sed 's/ /_/g'  ~/lab04-$MYGIT/PONPON/PONPON.homologs.al.fas | seqkit grep -v -r -p "dupelabel" >  ~/lab05-$MYGIT/PONPON/PONPON.ho
mologsf.al.fas
```
Next we used the IQ-TREE command to run the program on our alignments.
```command
2. iqtree -s ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.fas -bb 1000 -nt 2
```
We used the following commands to view the unrooted tree and convert it into a pdf.
```commands
A. nw_display ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.fas.treefile
B. Rscript --vanilla ~/lab05-$MYGIT/plotUnrooted.R  ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.fas.treefile ~/lab05-$MYGIT/PONPON/PON
PON.homologsf.al.fas.treefile.pdf 0.4 15
```
Next we rooted the tree using midpoint rooting.
```commands
3. gotree reroot midpoint -i ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid
.treefile
```
The following commands are once again optional and were jsut so we could view the tree.
```commands
C. nw_order -c n ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile  | nw_display -
D. nw_order -c n ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile | nw_display -w 1000 -b 'opacity:0' -s  >  ~/lab05-$MYGIT/PO
NPON/PONPON.homologsf.al.mid.treefile.svg -
E. convert  ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile.svg  ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile.pdf
F. nw_order -c n ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile | nw_topology - | nw_display -s  -w 1000 > ~/lab05-$MYGIT/PO
NPON/PONPON.homologsf.al.midCl.treefile.svg -
convert ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.midCl.treefile.svg ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.midCl.treefile.pdf
```

### Lab 06: Reconciling a Gene and Species Tree
In this lab we reconciled the trees created in the previosu section with a species tree. 

As usual we created a directory and copied the relevant information there.
```command
A. mkdir ~/lab06-$MYGIT/PONPON
B. cp ~/lab05-$MYGIT/PONPON/PONPON.homologsf.al.mid.treefile ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile
```
We used Notung to reconcile the trees.
```commands
1. java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s ~/lab05-$MYGIT/species.tre -g ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/PONPON/
```
We had to use the following commands to view and convert the generated files. Activating an environment will not be necessary for everyone of course but we had to so we could use thirdkind.
```
C. conda activate my_python27_env
D. python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile.rec.ntg --include.species
E. thirdkind -Iie -D 40 -f ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile.rec.ntg.xml -o  ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile.rec.svg
F. convert  -density 150 ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile.rec.svg ~/lab06-$MYGIT/PONPON/PONPON.homologs.al.mid.treefile.rec.pdf
```

### Lab 08: Protein Domain Prediction
In this lab we predicted protien domains based on the unaligned sequences. We transffered them from Lab 04 and removed the stop codons for processing. 
Starting off as usual:
```commands
A. mkdir PONPON
B. cd PONPON
C. sed 's/*//' ~/lab04-$MYGIT/PONPON/PONPON.homologs.fas > ~/lab08-$MYGIT/PONPON/PONPON.homologs.fas
```
Pfam was already in our instance so we had no need to install it and just ran the following command.
```command
1. rpsblast -query ~/lab08-$MYGIT/PONPON/PONPON.homologs.fas -db ~/data/Pfam/Pfam -out ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out  -outfmt "6 qseqid qlen qstart qend evalue stitle" -evalue .0000000001
```
We copied the gene tree from Lab 05.
```command
D. cp ~/lab05-$MYGIT/PONPON/PONPON.homologsf.outgroupbeta.treefile ~/lab08-$MYGIT/PONPON
```
We used an Rscript to create a pdf for viewing and then examined the results using the following optional commands.
```commands
E. Rscript  --vanilla ~/lab08-$MYGIT/plotTreeAndDomains.r ~/lab08-$MYGIT/PONPON/PONPON.homologsf.outgroupbeta.treefile ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out ~/lab08-$MYGIT/PONPON/PONPON.tree.rps.pdf
F. mlr --inidx --ifs "\t" --opprint  cat ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out | tail -n +2 | less -S
G. cut -f 6 ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out | sort | uniq -c
H. awk '{a=$4-$3;print $1,'\t',a;}' ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out |  sort  -k2nr
I. cut -f 1,5 -d $'\t' ~/lab08-$MYGIT/PONPON/PONPON.rps-blast.out
```

## Part 2: The Actual Project
The entire project was run with one script using Orthofinder. The script is below. 

```script
#!/usr/bin/env bash

# Load necessary modules
module load slurm
module load phylo/1.0        # Load the phylo module

# Submit this script using sbatch: sbatch orthofinder_job.slurm
 
#SBATCH --job-name=orthofinder_job           # Job name
#SBATCH --output=/gpfs/scratch/guwechue/BIO312_Project/orthofinder.log  # Output log file
#SBATCH --ntasks-per-node=96                 # Number of cores per node
#SBATCH --nodes=1                            # Number of nodes
#SBATCH --time=48:00:00                      # Max walltime (48 hours)
#SBATCH -p long-96core                       # Specify the partition
#SBATCH --qos=long-96-core-qos               # Set the QoS for this partition

# Run OrthoFinder with 96 threads, using MSA (Multiple Sequence Alignment) method
orthofinder -f /gpfs/scratch/guwechue/BIO312_Project/Proteomes -M msa -t 96
```
