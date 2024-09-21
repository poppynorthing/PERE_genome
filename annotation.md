# Annotation of the <i>P. recurvata</i> Genome Assembly
## 1. Masking Repetitive Elements
A custom species-specific library of identified and classified repetitive element families was generated with RepeatModeler v2.0.3 (Flynn et al., 2020).
```
BuildDatabase -name pere pere_assembly.fa

RepeatModeler -database pere -pa 16 > pere_out.log
```

Next, distinct repeats were identified and soft masked (--xsmall) in the assembly iteratively with RepeatMasker v4.1.3 
(https://www.repeatmasker.org/RepeatMasker/), starting with simple repeats (--noint), then repeats identified from the nrTEplants2020 curated repetitive sequence library (Contreras-Moreira et al., 2021), 
followed by known and unknown repeats from the <i>P. recurvata</i> custom repeat library from RepeatModeler.

Iteratively mask repeats:
```
mkdir -p logs 01_simple_out 02_nrTEplants_out 03_known_out 04_unknown_out

#Round 1: Mask simple repeats
RepeatMasker -pa 16 -a -e rmblast -gff -dir 01_simple_out -noint -xsmall pere_final.fa 2>&1 | tee logs/01_simplemask.log

mv 01_simple_out/pere_final.fa.masked 01_simple_out/pere_simple_mask.masked.fasta
mv 01_simple_out/pere_final.fa.align 01_simple_out/pere_simple_mask.align
mv 01_simple_out/pere_final.fa.cat.gz 01_simple_out/pere_simple_mask.cat.gz
mv 01_simple_out/pere_final.fa.out 01_simple_out/pere_simple_mask.out

#Round 2: Mask repeats from the nrTEplantsJune2020 repeat library
RepeatMasker -pa 16 -a -e rmblast -gff -lib nrTEplantsJune2020.fna -xsmall -dir 02_nrTEplants_out \
01_simple_out/pere_simple_mask.masked.fasta 2>&1 | tee logs/02_nrTEplants-mask.log

mv 02_nrTEplants_out/pere_simple_mask.masked.fasta.masked 02_nrTEplants_out/pere_nrTEplants_mask.masked.fasta
mv 02_nrTEplants_out/pere_simple_mask.masked.fasta.align 02_nrTEplants_out/pere_nrTEplants_mask.align
mv 02_nrTEplants_out/pere_simple_mask.masked.fasta.cat.gz 02_nrTEplants_out/pere_nrTEplants_mask.cat.gz
mv 02_nrTEplants_out/pere_simple_mask.masked.fasta.out 02_nrTEplants_out/pere_nrTEplants_mask.out

#Round 3: mask known elements from de novo library
RepeatMasker -pa 16 -xsmall -a -e rmblast -gff -lib customlib_curation/reference-genome-families.pere1.fa.known \
-dir 03_known_out 02_nrTEplants_out/pere_nrTEplants_mask.masked.fasta 2>&1 | tee logs/03_knownmask.

mv 03_known_out/pere_nrTEplants_mask.masked.fasta.masked 03_known_out/pere_known_mask.masked.fasta
mv 03_known_out/pere_nrTEplants_mask.masked.fasta.align 03_known_out/pere_known_mask.align
mv 03_known_out/pere_nrTEplants_mask.masked.fasta.cat.gz 03_known_out/pere_known_mask.cat.gz
mv 03_known_out/pere_nrTEplants_mask.masked.fasta.out 03_known_out/pere_known_mask.out

#Round 4: mask unknown elements from de novo library
RepeatMasker -pa 16 -xsmall -a -e rmblast -gff -lib customlib_curation/reference-genome-families.pere1.fa.unknown \
-dir 04_unknown_out 03_known_out/pere_known_mask.masked.fasta 2>&1 | tee logs/04_unknownmask.log

mv 04_unknown_out/pere_known_mask.masked.fasta.masked 04_unknown_out/pere_unknown_mask.masked.fasta
mv 04_unknown_out/pere_known_mask.masked.fasta.align 04_unknown_out/pere_unknown_mask.align
mv 04_unknown_out/pere_known_mask.masked.fasta.cat.gz 04_unknown_out/pere_unknown_mask.cat.gz
mv 04_unknown_out/pere_known_mask.masked.fasta.out 04_unknown_out/pere_unknown_mask.out
```
Combine all of the results:
```
mkdir -p 05_full_out

# Combine full RepeatMasker results - .cat.gz
cat 01_simple_out/pere_simple_mask.cat.gz \
02_nrTEplants_out/pere_nrTEplants_mask.cat.gz \
03_known_out/pere_known_mask.cat.gz \
04_unknown_out/pere_unknown_mask.cat.gz \
> 05_full_out/pere_final.full_mask.cat.gz

# Combine RepeatMasker tabular files for all repeats - .out
cat 01_simple_out/pere_simple_mask.out \
<(cat 02_nrTEplants_out/pere_nrTEplants_mask.out | tail -n +4) \
<(cat 03_known_out/pere_known_mask.out | tail -n +4) \
<(cat 04_unknown_out/pere_unknown_mask.out | tail -n +4) \
> 05_full_out/pere_final.full_mask.out

# Copy RepeatMasker tabular files for simple repeats - .out
cat 01_simple_out/pere_simple_mask.out > 05_full_out/pere_final.fa.out

# Combine RepeatMasker tabular files for complex, interspersed repeats - .out
cat 02_nrTEplants_out/pere_nrTEplants_mask.out \
<(cat 03_known_out/pere_known_mask.out | tail -n +4) \
<(cat 04_unknown_out/pere_unknown_mask.out | tail -n +4) \
> 05_full_out/pere_final.complex_mask.out

# Combine RepeatMasker repeat alignments for all repeats - .align
cat 01_simple_out/pere_simple_mask.align \
02_nrTEplants_out/pere_nrTEplants_mask.align \
03_known_out/pere_known_mask.align \
04_unknown_out/pere_unknown_mask.align \
> 05_full_out/pere_final.full_mask.align

# Combine all of the libraries used as input
cat customlib_curation/reference-genome-families.pere1.fa \
nrTEplantsJune2020.fna > full_lib.fa
```
Generate a summary table with everything using ProcessRepeats (RepeatMasker):
```
ProcessRepeats -a -lib full_lib.fa 05_full_out/pere_final.full_mask.cat.gz 2>&1 | tee logs/05_fullmask.log
```

## 2. Gene Structure Annotation
Gene structural annotation of the P. recurvata genome assembly was performed on the soft-masked assembly using the BRAKER3 annotation pipeline  (Hoff et al., 2016, 2019; Brůna et al., 2021; Gabriel et al., 2021, 2024), which uses evidence from homologous proteins and transcripts to train AUGUSTUS (Stanke et al., 2006) and GeneMark-EP+ (Lomsadze et al., 2005; Brůna et al., 2020) to produce a final set of high-confidence gene models. 

First, align Iso-Seq reads to the reference genome:
```
T=16 # number of threads

minimap2 -t${T} -ax splice:hq -uf pere_assembly_te_masked.fa transcripts.fasta > transcript_alignment.sam

samtools view -bS --threads ${T} transcript_alignment.sam -o transcript_alignment.bam
```
Then, run BRAKER3 for long read transcript evidence:
```
singularity exec braker3_lr.sif braker.pl --workingdir=braker_isoseq_ch_vdp_31aug2024 \
--genome=pere_assembly_te_masked.fa --prot_seq=Eudicot10.fa –-bam=transcript_alignment.bam \
--threads=16 --species=pere --gff3 --AUGUSTUS_CONFIG_PATH=$AUGUSTUS_CONFIG_PATH
```

## 3. Stuctural Annotation Evaluation
The resulting set of predicted proteins was assessed for completeness against the eudicots_odb10 (08 jan 2024) benchmarking ortholog database with BUSCO v5.6.1 in euk_tran mode (-m tran) (Manni et al., 2021).
```
busco -i braker_pere/pere_codingseqs.fa -m tran -l eudicots_odb10 -c 94 -o eudicot_pere_busco
```
Summary statistics describing the annotation were generated using agat_sp_statistics.pl from AGAT v1.0.0 (Dainat, J. https://www.doi.org/10.5281/zenodo.3552717)
```
singularity run agat_1.0.0--pl5321hdfd78af_0.sif agat_sp_statistics.pl \
--gff /path/pere_annotation.gff3 \
> AGAT_stats_output
```

## 3. Gene Function Annotation
The final protein set was functionally annotated using InterProScan v5.66-98.0 with the --goterms to include GO terms in the annotation (Jones et al., 2014).
```
interproscan -i pere_proteins.aa -d output_go --goterms
```
