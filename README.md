# Mouse-HOXA-4C-Seq
The file Cotney_Lab_4C-seq.txt contains 
1) scripts to further demultiplex 4C data by viewpoint using cutadapt and check the 
quality of the cutadapt output
2) steps to align the fully demultiplexed 4C data with the genome and generate bedGraphs
of the raw data
3) scripts to prepare the data for analysis with r3Cseq, preparation steps include reducing
the number of reads within 1.5kb to either side of the viewpoint and choosing a number of
reads to subsample to ensure that an equal number of reads from each bam file are being
used in comparative analysis by r3Cseq.
4) scripts to write a large number of R scripts for r3Cseq, running the analysis over many windows
 
The initial input is fastq.gz files demultiplexed by biological sample/replicate 
(by standard illumina barcode demultiplexing).
The scripts in some cases prepare PBS scripts for use on an hpc cluster running TORC, 
which can be modified as your environment requires.

To run the scripts as written, you need the following programs:
cutadapt
fastqc
multiqc
bowtie2
samtools
bedtools
R and r3Cseq (modified version found at https://github.com/cotneylab/r3Cseq), and all dependencies required for r3Cseq (http://r3cseq.genereg.net/Site/index.html)

The user must supply files at certain steps:

Demultiplex using cutadapt: primer_list.txt 
(two column file with the name of the viewpoint and the genomic sequence of the viewpoint 
to be trimmed by cutadapt)
<viewpoint_name>	<adapter_sequence>

Fastqc and cutadapt quality check: tissue_list.txt
(single column file with descriptions of the biological samples, i.e. tissue names or 
replicate names, the names should match the unique names given to the *trimmed.fastq.gz
files generated during cutadapt processing)
<Tissue_rep>

Split 4C data bam files: set of "remove_1.5kb.bed" files, one per viewpoint
columns are vp#, viewpoint_name, -1.5kb from left side of NlaIII/DpnII fragment, +1.5kb from right side of NlaIII/DpnII fragment
echo vp1	viewpoint1	50888274	50891713 > vp1_remove_1.5kb.bed
echo vp2	viewpoint2	50889686	50893035 > vp2_remove_1.5kb.bed
echo vp3	viewpoint3	51119551	51122986 > vp3_remove_1.5kb.bed
echo vp4	viewpoint4	51122174	51125613 > vp4_remove_1.5kb.bed
echo vp5	viewpoint5	51957704	51961137 > vp5_remove_1.5kb.bed
echo vp6	viewpoint6	52152952	52156310 > vp6_remove_1.5kb.bed

Add back a small number of reads near the viewpoint: <vp#>_<mid/min>_downsampling.txt
This is a file for the tissues and replicates you will be using in one specific r3Cseq analysis, it needs to be in the directory with the bam files
The format is: <samtools_-o_file>	<percent(=percent x 0.001)>	<new_name_for_downsampled.bam>	<samtools_-U_file>	<Merged_file_name>
For the example analysis, mine looked like this:
	Brain_1_viewpoint2_trimmed_sorted.bam_reads_1.5k_near_vp.bam	10.0001	Brain_1_viewpoint2_near_vp_min_downsampled.bam	Brain_1_viewpoint2_trimmed_sorted.bam_omit_1.5k_near_vp.bam	Brain_1_viewpoint2_ts_min_merged.bam
 	Brain_2_viewpoint2_trimmed_sorted.bam_reads_1.5k_near_vp.bam	10.0001	Brain_2_viewpoint2_near_vp_min_downsampled.bam	Brain_2_viewpoint2_trimmed_sorted.bam_omit_1.5k_near_vp.bam	Brain_2_viewpoint2_ts_min_merged.bam
	Face_1_viewpoint2_trimmed_sorted.bam_reads_1.5k_near_vp.bam	10.0001	Face_1_viewpoint2_near_vp_min_downsampled.bam	Face_1_viewpoint2_trimmed_sorted.bam_omit_1.5k_near_vp.bam	Face_1_viewpoint2_ts_min_merged.bam	
	Face_2_viewpoint2_trimmed_sorted.bam_reads_1.5k_near_vp.bam	10.0001	Face_2_viewpoint2_near_vp_min_downsampled.bam	Face_2_viewpoint2_trimmed_sorted.bam_omit_1.5k_near_vp.bam	Face_2_viewpoint2_ts_min_merged.bam




 
