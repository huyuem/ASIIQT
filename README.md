# ASIIQT v1.0
ASIIQT:  an Allele-Specific Isoform Identification and Quantification Tool
Protocols for ASIIQT analysis

i) Identfication of ASIs:

0.	Given the CCS reads of the single-molecule RNA-Seq: sample_raw.longReads.fasta; 
          the high-quality isoform assembled using IsoSeq: sample_hq_isoform.fastq;
          NGS RNA-Seq reads: sample_NGS.reads.fasta;
          Reference genome: ref.fa
          
1.	Mapping the CCS reads to reference genome with GMAP and minIntronLength = 25 nt, generating SAM file: sample_raw.longReads.sam

2.	./transcriptIdentify     sample_raw.longReads.sam     >sample_transcripts.txt

3.	./pseudoReadRefCoord     sample_transcripts.txt     >sample_pseudo.longReads.coord.txt

4.	./pseudoReadsSeqExt     ref.fa	    sample_pseudo.longReads.coord.txt     >sample_pseudo.longReads.fasta

5.	./rawReadSeqModify     sample_transcripts.txt     sample_raw.longReads.fasta     >sample_modified.longReads.fasta

6.	Mapping 'sample_NGS.reads.fasta' against 'sample_pseudo.longReads.fasta';
    Recalling SNPs with SAMtools, generating BCF-derived VCF file: sample_bcf.txt;
    Low-quality filtering of 'sample_bcf.txt', generating: sample_filtered.vcf.txt
    
7.	./bcfCorrect     sample_bcf.txt     sample_pseudo.longReads.fasta     >sample_NGS.corrected.pseudo.longReads.fasta

8.	./lqMark     sample_vcf.txt      sample_NGS.corrected.pseudo.longReads.fasta	>sample_lq_marked.NGS_corrected.pseudo.longReads.fasta

9.	./snpMark    sample_vcf.txt     sample_lq_marked.NGS_corrected.pseudo.longReads.fasta	>sample_SNP_marked.NGS_corrected.pseudo.longReads.fasta

10.	./modifiedLongReadsCorrect   sample_pseudo.longReads.coord.txt   sample_modified.longReads.fasta   sample_filtered.vcf.txt   sample_SNP_marked.NGS_corrected.pseudo.longReads.fasta
         #Automatically generating two files: SNP_marked.NGS_corrected.modified.longReads.fasta; modified.longReads.SNP.annotation.txt;
    Renaming the files with 'sample_SNP_marked.NGS_corrected.modified.longReads.fasta' and 'sample_modified.longReads.SNP.annotation.txt'

11.	./snpSet    sample_pseudo.longReads.coord.txt     sample_filtered.vcf.txt  
          #Automatically generating a file: chr_index_SNP.txt;
    Renaming the file with 'sample_chr_index_SNP.txt'
    
12.	./phasing     sample_modified.longReads.SNP.annotation.txt     sample_chr_index_SNP    sample_SNP_marked.NGS_corrected.modified.longReads.fasta
         #Automatically generating 9 files:
           (1) to_be_phased.txt ; 
           (2) to_be_phased_2.txt;
           (3) to_be_phased_2_ann.txt;
           (4) to_be_phased_2_stat.txt;
           (5) phased_raw.txt;
           (6) phased_correction.txt 
           (7) read_phased.txt;
           (8) corrected.modified.longReads.fasta;
           (9) phased.modified.longReads.fasta
     Renaming the files with a prefix 'sample_' added.


ii) Quantification of ASIs:

13.	Mapping 'sample_hq_isoform.fastq' to reference genome with GMAP, generating SAM file: sample_hq_isoform.sam

14. ./transcriptIdentify     sample_hq_isoform.sam     >sample_hq_isoform_transcripts.txt

15. ./phasedIso      sample_chr_index_SNP.txt      sample_hq_isoform.fastq      sample_phased_raw.txt      sample_hq_isoform_transcripts.txt
        #Automatically generating 2 files:，isoform_phased.txt; phased.hq_isoform.fasta;
    Renaming the files with 'sample_isoform_phased.txt' and 'sample_phased.hq_isoform.fasta' resepctively

16. Mapping 'sample_NGS.reads.fasta' against 'sample_hq_isoform.fastq' with bowtie or bwa, reporting at most 2 best matches, generating SAM file: sample_ngs.hq_isoform.sam

17. ./asiqFilt	sample_ngs.hq_isoform.sam	  >sample_ngs.hq_isoform.filtered.sam
18. ./asiq	sample_isoform_phased.txt  sample_ngs.hq_isoform.filtered.sam  >sample_asiq.out.txt	
