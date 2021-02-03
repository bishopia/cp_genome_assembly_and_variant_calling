## assembly and annotate diatom plastid genome, map individuals to reference, call SNPs/indels
### ian bishop
### 2021-02-02

## prepare materials

```
#install getOrganelle
interact
module load anaconda/2020.02
conda create --name getorganelle
source activate getorganelle
conda install -c bioconda getorganelle

#gather reads and references
ln -s ~/filtered_reads_R1_P.fq.gz $PROJ_DIR
ln -s ~/filtered_reads_R2_UP.fq.gz $PROJ_DIR
ln -s ~/filtered_reads_R1_P.fq.gz $PROJ_DIR
ln -s ~/filtered_reads_R2_UP.fq.gz $PROJ_DIR

cp Tpse_mito_complete_genome.fasta $PROJ_DIR
```

## run getOrganelle, batch script

#### run getOrganelle using batch function in Oscar. be sure to adjust word_size parameter (-w), to include a mitogenome reference. SPAdes is pretty memory intensive here so make sure you have at least 84G allocated

```
#SBATCH -n 8
#SBATCH -t 2:30:00
#SBATCH --mem=84G
#SBATCH --account=epscor-condo
#SBATCH -J batch_getOrganelle
##SBATCH --array=11-13,34,36,48-50,52-53,55-56
#SBATCH --array=19
ID=$SLURM_ARRAY_TASK_ID

module load anaconda/2020.02
source activate getorganelle

LIB=lib${ID}

get_organelle_from_reads.py -1 ${LIB}_R1_P.fq.gz -2 ${LIB}_R2_P.fq.gz \
	-u ${LIB}_R1_UP.fq.gz,${LIB}_R2_UP.fq.gz \
	-o ${LIB}_mt_output \
	-R 50 -k 21,45,65,85,105 -P 1000000 \
	-F embplant_mt \
	-t 8 \
	-w 40 \
	-s Tpse_mt_complete.fasta
```

## confirm/circularize genome with bandage

#### secure copy (scp) output plastid graph and fasta to personal laptop

```
#scp ibishop@transfer.ccv.brown.edu:${PROJ_DIR}/${LIB}_mt_output/filtered_K105.assembly_graph.fastg.extend-embplant_mt-embplant_pt.fastg ~/$TARGET_DIR

ibishop@transfer.ccv.brown.edu:${PROJ_DIR}/${LIB}_mt_output/filtered_K105.assembly_graph.fastg.extend-embplant_mt-embplant_pt.csv ~/$TARGET_DIR
```

#### now open bandage, load .fastg and .csv, delete extraneous contigs. For a diatom plastid genome, you're looking for two loops connected by a contig, representing two genic regions flanked by inverted repeats. command-select one strand from each of the two loops and both strains of the repeat tie in the middle, then go to Output/Save selected path sequence to fasta to export. The resulting fasta should have "(circular)" at the end of the header indicating that you've extracted the full circular plastid genome.

## annotate with CHLOROBOX GeSeq

Now that you have a circularized draft reference plastid genome, you can import it into the web-based annotator tool GeSeq (CHLOROBOX). Use mostly default options, but be sure to load many RefSeq plastid genomes to help with the annotation effort. Save gbk output

## map sample reads to reference, keep only unique mappers

#### secure copy the circularized genome and the gbk annotation file and the OGDRAW jpg back to a mapping directory

```
scp ~/Downloads/mt_output_getorganelle/lib19_mt_output/path_sequence_config1_cp.fasta ibishop@transfer.ccv.brown.edu:~/scratch/2getorg/lib19_ref_mapping
```

#### rename reference
```
mv path_sequence_config1_cp.fasta lib19_cp_ref.fasta
```

#### pull in symbolic links of reads to align to reference
```
mkdir reads
cd reads
ln -s $FILTERED_READS/ .
cd ..
```

#### use bwa-mem and samtools to align reads to cp genome, keep only unique alignments, sort, then conversion to bam

```
#!/bin/bash

#SBATCH --time=00:12:00
#SBATCH --mem=32G
#SBATCH -J mapping-sorting-dedup-rgtag_array
#SBATCH -n 32
##SBATCH --array=11-14,19,21-30,32-56
#SBATCH --array=19
#SBATCH --account=epscor-condo

ID=$SLURM_ARRAY_TASK_ID


#load modules
module load bwa/0.7.15
module load samtools/1.9
#module load picard-tools/2.9.2 
module load bamaddrg/20180928

REF=lib19_cp_ref.fasta

#index ref.fasta
bwa index $REF

#align with bwa mem, convert to uncompressed BAM, removing unpaired and unmapped reads, then sorted and saved
#add '-f 0x02' as view argument if you want to exclude unpaired reads
#add '-F 0x04' as view argument if you want to exclude unmapped reads
bwa mem $REF reads/lib${ID}_R1_P.fq.gz reads/lib${ID}_R2_P.fq.gz -t 32 | samtools view -@ 32 -Su - | samtools sort - > lib${ID}.sorted.allreads.bam


#remove unmapped reads
samtools view -b -F 4 lib${ID}.sorted.allreads.bam > lib${ID}.sorted.bam


#mark and delete duplicates with Picard;
java -jar ~/data/ibishop/scripts/picard.jar MarkDuplicates \
	INPUT=lib${ID}.sorted.bam \
	OUTPUT=lib${ID}.sorted.dedup.bam \
	REMOVE_DUPLICATES=true \
	METRICS_FILE=metrics_lib${ID}.txt


# add read group ids
bamaddrg -b lib${ID}.sorted.dedup.bam > lib${ID}.sorted.dedup.rg.bam;

#remove intermediate temp files
#rm lib${ID}.sorted.allreads.bam
#rm lib${ID}.sorted.dedup.bam
#rm lib${ID}.sorted.bam
```

## call variants

#### make list of bamfiles
```
ls *.sorted.dedup.rg.bam > bamlist.txt
```

#### call variants

```
#!/bin/bash

#SBATCH --time=48:00:00
#SBATCH --mem=96G
#SBATCH -J freebayes
#SBATCH -n 4
#SBATCH --account=epscor-condo

module load freebayes/1.1.0 

REF=lib19_cp_ref.fasta

freebayes -f $REF -L bamlist.txt > total_snps.vcf
```

#### filter variants

## using ddocent variant filter tutorial, remove variants when call rate is low, depth is low, qual is low, etc.

```
vcftools --vcf total_snps.vcf --max-missing 0.5 --mac 3 --minQ 30 --recode --recode-INFO-all --out raw.g5mac3

vcftools --vcf raw.g5mac3.recode.vcf --minDP 3 --recode --recode-INFO-all --out raw.g5mac3dp3

#check for individuals with low call rates, remove if necessary
vcftools --vcf raw.g5mac3dp3.recode.vcf --missing-indv
cat out.imiss

vcftools --vcf raw.g5mac3dp3.recode.vcf --max-missing 0.95 --maf 0.05 --recode --recode-INFO-all --out DP3g95maf05 --min-meanDP 20
```

#### export filtered variants, annotation and reference for IGV


































