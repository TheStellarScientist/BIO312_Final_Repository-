# BIO312 Final Repository
This document contains all the commands used to create my project. There are two parts. The commands used in Labs 03 - 08 which were not used for my project and then the actual commands I used for my project. 

## Part 1: The Labs 

### Lab 03: Finding homologs with BLAST KEY
The purpose of this lab is to conduct a BLAST search on a starting protein to find homologs and get a general picture of size of the gene family. 

```commands
List of commands used for PONPON (excluding less commands used to view files)
```
The command below is used to create a directory in Lab 03 called PONPON where all the relevant files and operations for this lab will be stored and conducted.
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
In this lab we aligned the genes we found in our BlAST search from the previous lab. 
