# MyGenome
Analyses for ABT480/CS485G genome assembly

## 1. Analysis of sequence quality
The F1 and R1 sequence datasets were analyzed using FASTQC:
```bash
ssh -Y jcca274@jcca274.cs.uky.edu
cd MyGenome
fastqc &
```
Load F1 and R1 datasets into GUI interface. 
Take screenshots of output files
Forward Paired:
![F1screenshot.png](/data/forward_paired.png)
Reverse Paired:
![R1screenshot.png](/data/reverse_paired.png)
## 2. Ran trimmomatic
```bash
java -jar ./trimmomatic-0.38.jar PE -threads 16 -phred33 -trimlog file.txt UFVPY166_1.fq UFVPY166_2.fq UFVPY166_1_paired.fastq UFVPY166_1unpaired.fastq UFVPY166_2_paired.fastq UFVPY166_2_unpaired.fastq ILLUMINACLIP:adaptors.fasta:2:30:10 SLIDINGWINDOW:20:20 MINLEN:100
```

## 3. Count the number of forward reads remaining
```bash
grep -c @A00261 forward_paired.fastq 
awk 'NR%4==2 {total += length($0)} END {print total}' forward_paired.fastq 
awk 'NR%4==2 {total += length($0)} END {print total}' reverse_paired.fastq 
```

On MCC ran
```bash
sbatch velvetoptimiser_noclean.sh UFVPY166 61 131 10
```
Gave hash value of 111, so ran
```bash
sbatch velvetoptimiser_noclean.sh UFVPY166 101 121 2
```
Gave hash value of 107

Renamed the sequence headers using
```bash
perl /project/farman_s24cs485g/SCRIPTS/SimpleFastaHeaders.pl UFVPY166.fasta UFVPY166
```


Then ran
```bash
sbatch /project/farman_s24cs485g/SLURM_SCRIPTS/BuscoSingularity.sh UFVPY166_nh.fasta
```
*File is too large to upload.

## Removed Contigs shorter than 200 base pairs long
```bash
perl CullShortContigs.pl UFVPY166_nh.fasta
```

## Run blast against the MoMitochondrion file.
```bash
blastn -query MoMitochondrion.fasta -subject UFVPY166_nh.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.UFVPY166.BLAST
```
[MoMitochondrion.UFVPY166.BLAST](/data/MoMitochondrion.UFVPY166.BLAST)

## Save output to .csv file
```bash
awk '$4/$3 > 0.9 {print $2 ",mitochondrion"}' MoMitochondrion.UFVPY166.BLAST > UFVPY166_mitochondrion.csv
```
[UFVPY166_mitochondrion.csv](/data/UFVPY166_mitochondrion.csv)

## Blast against B71v2sh_masked
```bash
blastn -query B71v2sh_masked.fasta -subject UFVPY166_Final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid qstart qend sstart send btop' -out B71v2sh.UFVPY166.BLAST
```
[B71v2sh.UFVPY166.BLAST](/data/B71v2sh.UFVPY166.BLAST)

## Identify variants
```bash
sbatch CallVariants.sh UFVPY166_BLASTS
```

## Run Snap
```bash
snap-hmm Moryzae.hmm UFVPY166_final.fastaa UFVPY166-snap.zff
```
[UFVPY166-snap.zff](data/UFVPY166-snap.zff)

## Run Augustus
```bash
augustus --species=magnaporthe_grisea --gff3=on --singlestrand=true --progress=true ../snap/UFVPY166_final.fasta > UFVPY166-augustus.gff3
```
* File too large to upload.

## Run Maker
```bash
maker 2>&1 | tee maker.log
```

## Combine maker results
```bash
gff3_merge -d UFVPY166_final.maker.output/UFVPY166_final_master_datastore_index.log -o UFVPY166-annotations.gff
```
* File too large to upload.

## Count number of unique genes: Results in 12,750 genes.
```bash
grep 'UFVPY166_contig' UFVPY166-annotations.gff | awk '{print $3}' | grep 'gene' | wc -l
```

## Maker output file
[UFVPY166-genes.fasta.all.maker.proteins.fasta](data/UFVPY166-genes.fasta.all.maker.proteins.fasta)
