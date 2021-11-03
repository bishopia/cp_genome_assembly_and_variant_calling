# cp_genome_assembly_and_variant_calling

### intro
in this repo, i describe the process by which I used PE150 short reads and assembled a set of chloroplast genome assemblies for various strains of the same species. the overall procedure could be outlined as follows:

- trim reads for adaptors and quality
- assemble circular genome via getOrganelle
- examine assembled path via bandage to check circularity
- check dot plots and reverse complement for downstream alignment
- annotate one (or all) via Chlorobox
- align with mauve

