# cp_genome_assembly_and_variant_calling

## background
It is unclear to what extent chloroplast genome sequence varies among and within two sampling localities. We are currently considering lcWGS of co-cultures and would like to know how many polymorphic sites there are among closely related isolates, so that we can estimate how much polymorphic site information we might expect and rely on to deconvolute competing co-cultures in our lineage sorting experiment.

## methods
In this pipeline, i applied getOrganelle, Bandage and CHLOROBOX GeSeq to assembly, circularize and annotate a draft plastid genome, respectively. This circularized and annotated reference genome was then used to align filtered short reads from 40 different <i>T. rotula</i> isolates and to call SNVs and short indels among individuals (applying bwa-mem, samtools, bamaggr, picardTools, and bayescan). Raw SNVs were filtered for depth, call rate, quality and no individuals were removed for excess missing genotype data. Finally, the filtered vcf file, reference fasta and annotation file were manually combined in IGV for easy viewing of the distribution of SNVs across and within two sampling localities (NB:11-14,41-56; SA:19,21-30,32-40).

## results
Overall, we found approximately 50 variant sites among the 40 isolates. Given that we're working with plastid genomes, all variants are assumed to be in full LD. Initial viewing suggests that there are many variants that span both populations or are unique to either population.

![plot](manual_combination_vcf_gb_fasta.png)
#### <b>Figure 1.</b> Test

## conclusions
Initial conclusions are that there is plenty of genetic diversity upon which deconvolution analyses can be based, even among single cell isolates collected from the same water sample. If lcWGS genotyping only gave us sufficient coverage at high-copy number regions like rRNA, plastid and mitochondiral genomes, these results suggest that potentially hundreds of high depth segregating markers could be available for use (they are segregating in this case because we assume no mutation during the experiment). It is even possible that knowing <i>a priori</i> which variants belong to which strain (by sequencing them separately at higher coverage) is not strictly necessary, if our primary goal is to identify when one of the two genotypes goes extinct in each co-culture. If we have monocultures in the evolution plate as well, it is even more likely that we do not need any separate founder strain sequencing at higher coverage.

