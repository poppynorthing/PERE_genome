# Genome Assembly of <i>Pectocarya recurvata</i>
## 1. K-mer Counting & Analysis
K-mer analysis of the raw reads was used to estimate genome size, abundance of repetitive elements, heterozygosity, and ploidy using KMC v3.1.0 (Kokot et al., 2017), 
GenomeScope v1.0 (Vurture et al., 2017), and SmudgePlot v1.0 (Ranallo-Benavidez et al., 2020) with a maximum kmer coverage of 10,000 to count and analyze 17-mers, respectively.

First, we ran KMC to count and make a histogram of 17-mers.
```
#First, use KMC to count 17mers, counting kmer coverages between 1 and 10000x
kmc -k17 -t16 -ci1 -cs10000 PERE1.hifi_reads.fastq.gz 17mers .

#Generate k-mer histogram
kmc_tools transform 17mers histogram 17mers.hist -cx10000
```
Next, we ran SmudgePlot to estimate the ploidy of the individual sequenced using the 17-mers.
```
L=$(smudgeplot.py cutoff 17mers.hist L)
U=$(smudgeplot.py cutoff 17mers.hist U)
echo $L $U

kmc_tools transform 17mers -ci"$L" -cx"$U" reduce 17mers_L"$L"_U"$U"
smudge_pairs 17mers_L"$L"_U"$U" 17mers_L"$L"_U"$U"_coverages.tsv 17mers_L"$L"_U"$U"_pairs.tsv > 17mers_L"$L"_U"$U"_familysizes.tsv

kmc_tools transform 17mers -ci"$L" -cx"$U" dump -s 17mers_L"$L"_U"$U".dump
smudgeplot.py hetkmers -o 17mers_L"$L"_U"$U" < 17mers_L"$L"_U"$U".dump

smudgeplot.py plot -k 17 -n 132 17mers_L"$L"_U"$U"_coverages.tsv
```
## 2. De novo Assembly
The initial genome assembly was constructed de novo from the PacBio HiFi reads using Hifiasm v0.16.1 (Cheng et al., 2021). Based on low estimated heterozygosity from the K-mer analysis 
(see Results) and the selfing mating system of <i>P. recurvata</i>, purging of haplotypic duplications was skipped in the assembly (using the -l0 flag).

```
hifiasm -o PERE_01_homo.asm -t 94 -l0 PERE.hifi_reads.fasta.gz
```
## 3. Error-correction and Quality Assessment
The resulting assembly graph was visualized using Bandage v0.8.1 (Wick et al., 2015) and subsequently evaluated and error-corrected using Inspector (Chen et al., 2021).
```
inspector.py -c PERE_01_homo.asm.fasta -r PERE.hifi_reads.fasta.gz -o inspector_output –- datatype pacbio-hifi
```
The initial assembly was assessed for completeness using 2326 Benchmarking Universal Single-Copy Orthologs (BUSCOs) from the Eudicots dataset <i>eudicots_odb10</i> (01 Aug 2024) using BUSCO version 5.6.1 (Manni et al., 2021).
```
busco -i assembly.fasta -m genome -c 15 -l eudicots_odb10 -o pere_eudicot_busco
```
