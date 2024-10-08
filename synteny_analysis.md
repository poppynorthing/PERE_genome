# Synteny Analysis of <i>P. recurvata</i>
## 0. Annotate the <i>E. plantagineum</i> reference genome.
Raw transcriptome libraries (SRR4034891, SRR4034890, and SRR7076848) were used to generate a draft gene structural annotation of the <i>E. plantagineum</i> assembly (Tang et al. 2020) using BRAKER3 (Gabriel et al., 2024).
```
singularity exec braker3.sif braker.pl --workingdir=braker_echium --genome=echium_genome.fa --prot_seq=Eudicot_10.faa \
--rnaseq_sets_ids=SRR4034891,SRR4034890,SRR7076848 \
--threads=16 --species=echium --gff3 --AUGUSTUS_CONFIG_PATH=$AUGUSTUS_CONFIG_PATH
```

## 1. Run OrthoFinder
Groups of orthologous genes from the resulting proteomes from the <i>P. recurvata</i> and <i>E. plantagineum</i> genome annotations were identified using OrthoFinder v2.5.4 (Emms and Kelly, 2019).
```
orthofinder -f ./protein_files -t 8 -a 4 -X -o ./output_directory
```
## 2. Make Ks Plot with WGD2
 Evidence for ancient whole genome duplications in the <i>P. recurvata</i> genome was generated by analyzing the frequency distribution of synonymous divergence (Ks) between paralogs. 
 Synonymous divergence was calculated for each pair of duplicate genes (anchored based on synteny) using wgd v2 (Chen et al. 2024).
```
fasta="/path/pere_codingseqs.fasta"
gff="/path/pere_annotation.gff3"

wgd dmd ${fasta} -o wgd_dmd
wgd ksd pere_dmd/${fasta}.tsv ${fasta} -o pere_ksd
wgd syn -f mRNA -a pere_dmd/${fasta}.tsv ${gff} -ks pere_ksd/${fasta}.tsv.ks.tsv -o pere_syn
```
## 3. Detect Synteny with GENESPACE
The groups of shared orthologs identified by OrthoFinder were used to identify blocks of conserved gene order that cluster into regions of synteny using the GENESPACE v1.4 R package with default settings (Lovell et al., 2022). 
```
#Initialize the GENESPACE run & set parameters
gpar <- init_genespace(wd = my_wd, path2mcscanx = local_path2mcscanx)

#Run GENESPACE
out <- run_genespace(gsParam = gpar)
```
Additional formatting of the resulting riparian plot:
```
ggthemes <- ggplot2::theme(
 panel.background = ggplot2::element_rect(fill = "white")
)

ripDat <- plot_riparian(
 gsParam = out,
 braidAlpha = 0.75,
 chFill = "lightgrey",
 addThemes = ggthemes,
 refGenome = "pere",
 genomeIDs = c("echium", "pere"),
 useRegions = TRUE,
 chrLabFontSize = 1,
 useOrder = TRUE,
 reorderBySynteny = TRUE,
 chrExpand = 4
)
```

## 4. Calculate syntenic depth with MCscan
The syntenic depth ratio of each syntenic block (collinear sets of genes > 5) was calculated using the pythonic version of MCscan, which depends on LAST (Tang et al., 2008; Kiełbasa et al., 2011). 
```
#combine bed files for two genomes of interest
cat pere.bed echium.bed > pere-echium.bed #combine bed files

#run mcscan
python3 -m jcvi.compara.catalog ortholog pere_peps echium_peps

#plot histogram of syntenic depths
python3 -m jcvi.compara.synteny depth --histogram pere_peps.echium_peps.anchors
```
