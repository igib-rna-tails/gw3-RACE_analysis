# Pipeline to analyze Genome-wide 3'RACE data (gw-3'RACE)
Script to analysis of gw-3'RACE data from fastq files.

### Description of protocol:
Sequencing data were filtered for read pairs where R2 contained a 3'-adapter sequence ligated at the first steps of the library preparation. Low-quality read pairs were filtered out using fastp [36]. Reads R1 and R2 were aligned separately to the genome using STAR aligner [37], in the case of R2 settings allowed soft-clipping and keeping non-aligned reads in the final output (.sam). Next using samtools [38] and bedtools [39]  utilities uniquely aligned reads were extracted from the R1 alignment file and matched to genomic features forming a .bed type file. In the next step, R1 read names from created  .bed file were matched with R2 read mates from the .sam file, custom table was created containing in each row R2 CIGAR string, read name and sequence as well as R1 mate matched feature and R1 and R2 alignment coordinates. These raw data tables were a base for tails analysis, information about the tail existence, its length and type was than extracted from the tables using a custom Python script (https://github.com/igib-rna-tails/tailseq_complete_analysisgithub). The script extracted tail sequence from R2 read based on CIGAR string and categorised tails as poly(A), poly(A)U, oligo(U) or other using text search (grep regular expressions). If R2 was not aligned and therefore CIGAR was not available R2 sequence was scanned using text search to categorise the tail into one of the aforementioned four categories. The script produced as output a .csv type table with the information necessary for downstream data analysis and visualisation. The final table contained one row per uniquely aligned R1 read with information onof tail type, 3'-end coordinate (where applicable), tail length, number of Us (where applicable), gene name, and distance of detected 3'-end from annotated TES.

### 0. PrepareSTAR index
STAR --runMode genomeGenerate --genomeDir genome/ --genomeFastaFiles genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa


### 1. test.sh
bash test_script.sh -i Wild_type_clone2_R1_001.fastq -I Wild_type_clone2_R2_001.fastq -o output_Wild_type_clone2_20220830


### 2. Joining 
  *run the script in the directory with the output)
bash ../joining_R1R2.sh -i R1_Aligned.sortedByCoord.out.bam -I R2_Aligned.out.bam -a ../genome/annotation_6k_clean.bed


### 3.bigwigs 
  * run the script in the directory with the output
  * R1 input: bed, R2 input bam!
bash ../R1_bed_bigWig.sh -i R1_Aligned.sortedByCoord.out.bam_sorted_unique.bed -g ../genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa.fai 

bash ../R2_bed_bigWig.sh -i R2_Aligned.out.bam  -g ../genome/Schizosaccharomyces_pombe.ASM294v2.dna.toplevel.fa.fai 

### 4. Analysis of tails using Python script
