# cp_genome_assembly_and_variant_calling

## background
it is unclear how much chloroplast genome sequence varies among and within two sampling localities. we are currently considering lcWGS and would like to know how many polymorphic sites there are among closely related isolates, so that we can estimate how much segregating information we can rely on to deconvolute cryptic strains in our lineage sorting experiment.

## methods
in this pipeline, i used getOrganelle, bandance and CHLOROBOX GeSeq to assembly, circularize and annotate a draft plastid genome, respectively. This circularized and annotated reference genome was used to align short reads from 40 different T. tumida isolates and to call SNVs and short indels (applying bwa-mem, samtools, bamaggr, picardTools, and bayescan). raw variants were filtered for depth, call rate, quality and no individuals were removed for excess missing genotype data. Finally, the filtered vcf file, reference fasta and annotation file were manually combined in IGV for easy viewing of distribution of SNPs across two sampling localities (NB:11-14,41-56; SA:19,21-30,32-40).

## results
Overall, we found approximately 50 variant sites among the 40 isolates. Given that we're working with plastid genomes, all variants are assumed to be in full LD. Initial viewing suggests that there are many variants that span both populations or are unique to either population.

![plot](manual_combination_vcf_gb_fasta.png)

## conclusions
Initial conclusions are that there is plenty of genetic diversity upon which deconvolution analyses can be based, even among single cell isolates collected from the same sample. If lcWGS genotypes only gave us reasonable coverage at high-copy number regions like rRNA, plastid and mito, if this plastid is any indication, potential hundreds of high depth segregating markers could be available for use. It is even possible that knowing a priori which variants belong to which strain is not strictly necessary, if our goal is to identify one of the two genotypes goes extinct in each co-culture. If we have monocultures in the evolution plate as well, this analysis suggests that we might have enough information using variant calls in those samples without having to separately sequence them at higher coverage.

