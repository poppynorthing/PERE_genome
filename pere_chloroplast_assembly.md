# Chloroplast Genome Assembly for <i>Pectocarya recurvata</i>
## 1. Align PERE reads to reference
Potential chloroplast reads were identified by aligning the raw HiFi reads to the <i>Echium plantagineum</i> (Boraginaceae) reference plastome (Carvalho Leonardo et al., 2022; Genbank accession: OL335188.1) using Minimap2 v2.28 with default settings (Li, 2018).
```
minimap2 -a ecplan_plastome/ecplan_plastome.fa PERE.hifi_reads.fasta.gz -o alignment.sam
```

## 2. Filter and subsample the reads
Reads that did not align to the reference plastome were filtered and removed using Samtools v1.10 (Li et al., 2009). 
```
samtools view -b -F 4 alignment.sam > chloroplast_mapped_reads.bam
samtools sort chloroplast_mapped_reads.bam > sorted_chloroplast_mapped_reads.bam
```
The remaining mapped reads had an exceptionally high estimated coverage; accordingly, reads were randomly subsampled with SeqKit v2.8.1 (Shen et al., 2016) to approximately 150x coverage to mitigate assembly errors.
```
seqkit sample -p 0.001 sorted_chloroplast_mapped_reads.fa -o downsampled_reads_150x.fa
```
## 3. Assemble the Chloroplast Genome
These subsampled reads were then used to generate a circular <i>P. recurvata</i> chloroplast genome assembly using Hifiasm v0.16.1 with default settings (Cheng et al., 2021).
```
hifiasm -o cp150_asm/PERE_cp_desampled150.asm -t 94 -l0 downsampled_reads_150x.fa
```
