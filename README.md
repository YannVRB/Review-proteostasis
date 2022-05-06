# Review-proteostasis

Previously, we identified 149 transcripts differentially regulated by sleep deprivation in the translatome but not in the whole transcriptome (Lyons et al., 2020). Out of those 149 transcripts, 94 were significantly upregulated and 55 were significantly downregulated (adjusted p-value <0.05). We further investigated the mechanism of post-transcriptional regulation by  identifying common regulatory sequences in the RNA  using extracted sequence features to search for conserved binding sites for micro-RNA (miRNA) in the 3’ UTR. The following pipeline was applied to our list of transcripts down-regulated after sleep deprivation in the translatome (**Fig. 1**). This pipeline can be applied to any sequence feature from a list of genes or transcripts however results highly depend on the databases used for comparison.

![alt text](Figure 1.png)
**Figure 1. Overview of the pipeline analysis for the investigation in silico of miRNAs binding sites from a list of differentially regulated transcripts.**

## 3’UTR retrieval from transcripts IDs with BioMart
BioMart enables advanced querying of biological data sources through a single web interface. Full sets of ENSEMBL 3’UTR sequences were obtained using the BioMart utility (Smedley et al., 2009) (www.ensembl.org/biomart/martview) with the following filters and parameters (**Fig. 2**). Briefly, transcript names in the format “mt-Tf-201” were pasted into the filter. The attribute outputs required were the transcript stable identifications and their respective 3’UTR sequence. Results were exported in FASTA format (with .txt extension) as DNA sequence because BioMart only provides genomic sequences. This is not an issue here because we are specifying specific transcript isoforms therefore only the 3’UTR from the genomic annotation of the specific transcript is provided. In addition, tools in downstream analysis allow for alphabet expansion to compare RNA motifs to DNA motifs. Finally, as some transcripts may not have a published or characterized 3’UTR, one must remove IDs and sequences that display “sequence unavailable” in the FASTA file, otherwise the next step will fail.



**Figure 2. Filters and parameters used for 3’UTR sequence retrieval from a list of transcript names.**

## Unbiased motif discovery with MEME Suite
The MEME Suite is available through a web server or command line. It provides tools for discovery and analysis of enriched sequence motifs representing features such as DNA/RNA binding sites. Primary sequences were provided as input from the FASTA file generated by BioMart containing our 3’UTR sequences from translationally down-regulated transcripts. The site distribution was setup in “zoops” mode meaning that zero or one occurrence of the motif site is expected and distributed in the input sequences. This parameter is consistent with the fact that a 3’UTR of a given mRNA might include or not a binding site with the highest affinity for a miRNA to silence it. The number of motifs MEME Suite should find is flexible, the more motifs the longer the analysis will be. In advanced options, the discovered motif should be between 4 and 9 nucleotides to find matches for seed sequences. Finally, motif sites cannot be on both strands since our input sequences come from 3’UTR of single stranded RNA, therefore, one must allow MEME to search motifs in the given strand only.
Below is a summary of the parameters used by MEME Suite with the command line:

`meme 3UTR_DEGtranscripts55TRAP_down.txt -dna -oc . -nostatus -time 18000 -mod zoops -nmotifs 10 -minw 4 -maxw 9 -objfun classic -markov_order 0`

MEME Suite output contains DNA sequence LOGOS for each discovered motif significantly enriched in the input sequences (**Fig. 3**). MEME identified 4 motifs that were significantly enriched in our query of sequences. Most of them were related with transcription termination such as polyadenylation sites, however, motif number 2 seemed promising and was kept for downstream analyses.


**Figure 3. MEME Suite output from BioMart query containing 3’UTR sequences.** The discovered motifs are ranked based on their statistical significance (E-value). Each motif has several sites or occurrences in the query. In the context of the motif number 2, since here we performed the analysis in zoops mode, this means that 32 transcripts have this enriched motif in their 3’UTR. Each computed motif has a width of 9 nucleotides. The “More” column investigates the details of the motif, such as sequence position. The “Submit/Download” column facilitates using the MEME output as input for motif comparison tools including Tomtom.

## Comparison of motif number 2 with miRBase using Tomtom
Significantly enriched motifs can be conveniently submitted to the sequence and motif database scanning algorithm Tomtom (Gupta et al., 2007), for comparison against a database of known motifs. Tomtom ranks the motifs in the database and produces an alignment for each significant match. Motif number 2 from the previous analysis was used for comparison with the database “miRbase Single Species RNA” along with its “Mus_musculus_mmu” subcategory database. Since our query motif is displayed as DNA and we are searching for miRNA binding site, one must allow the alphabet expansion before running Tomtom. Default significance threshold (E-value <10) was used.
Below is a summary of the parameters used by Tomtom with the command line:

`tomtom -no-ssc -oc . -verbosity 1 -min-overlap 5 -mi 1 -dist pearson -evalue -thresh 10.0 -xalph -time 300 query_motifs db/MIRBASE/Mus_musculus_mmu.meme`

Tomtom identified 49 matches in this database that have a significant match the query motif number 2 from MEME Suite. The two most significant alignment for enriched motif number 2 included miRNA binding site for mmu-miR-7045-3p and mmu-miR-6971-5p (**Fig. 4**). Very little is known about the role of those two miRNAs and their regulatory mechanism, even less in the context of sleep deprivation. Only one study previously found down regulation of mmu-miR-7045-3p after local treatment for rheumatoid arthritis in bone marrow-derived dendritic cells (Wang et al., 2020). Nevertheless, it is possible to support the targets of those miRNA in silico even though experimental validation would be ideal. To this end, one can go back on the MEME Suite output and retrieve transcripts identification as well as the location of the enriched motif in each 3’UTR (**Fig. 5**). This allows us to identify the transcripts that contained the enriched motif number 2, and predicted to be targeted by the two miRNAs.



**Figure 4. Comparison and alignment of enriched motif number 2 from MEME Suite with miRNA database (miRbase).** The figure shows the Tomtom output from searching a single 3’UTR RNA motif against a collection of mouse miRNAs binding site motifs collected in miRbase database. Each miRNA from the database is being aligned to the enriched motif queried from MEME Suite. For each optimal alignment, a p-value is associated with the significant overlap between a motif and a miRNA.


**Figure 5. Details of enriched motif number 2 from MEME Suite and site locations in 3’UTR of the sequences queried.** Looking back to the MEME Suite output is necessary to observe which transcript is predicted to be targeted by a miRNA through the enriched motif. In addition, the exact site and sequence of the motif is displayed for each query.

## Validation in silico of transcripts targeted by mmu-miR-7045-3p and mmu-miR-6971-5p
The final in silico step to support the evidence that a subset of transcripts is down-regulated because of translational silencing by a miRNA is to assess if the transcripts are predicted to be recognized by a given miRNA. TargetScan predicts biological targets of miRNAs by searching for the presence of conserved 8mer, 7mer, and 6mer sites that match the seed region of each miRNA (Agarwal et al., 2015). Out of the 32 transcripts containing the enriched motif number 2, twenty of these were significantly predicted to be recognized by at least one of the two miRNAs (mmu-miR-7045-3p and/or mmu-miR-6971-5p) (**Fig. 6**).


**Figure 6. Predicted binding sites of mmu-miR-7045-3p and mmu-miR-6971-5p in the 3’UTR of down-regulated transcripts after sleep deprivation.** Venn diagrams represent the overlap of transcripts and their 3’UTR targeted by the two miRNAs. The horizontal black lines represent the 3’UTR and their respective length starting at 1. The red dots are the predicted binding sites for the miRNAs. 

## References
Agarwal, V., Bell, G.W., Nam, J.-W., and Bartel, D.P. (2015). Predicting effective microRNA target sites in mammalian mRNAs. ELife 4, e05005. https://doi.org/10.7554/eLife.05005.
Gupta, S., Stamatoyannopoulos, J.A., Bailey, T.L., and Noble, W.S. (2007). Quantifying similarity between motifs. Genome Biol 8, R24. https://doi.org/10.1186/gb-2007-8-2-r24.
Lyons, L.C., Chatterjee, S., Vanrobaeys, Y., Gaine, M.E., and Abel, T. (2020). Translational changes induced by acute sleep deprivation uncovered by TRAP-Seq. Molecular Brain 13. https://doi.org/10.1186/s13041-020-00702-5.
Smedley, D., Haider, S., Ballester, B., Holland, R., London, D., Thorisson, G., and Kasprzyk, A. (2009). BioMart – biological queries made easy. BMC Genomics 10, 22. https://doi.org/10.1186/1471-2164-10-22.
Wang, Y., Liu, H., Cao, M., Wang, X., Cong, S., Sun, J., Jia, B., Ta-bu-shi, N., Bian, Y., and Luo, L. (2020). Mechanism of miRNA-based Aconitum leucostomum Worosch. Monomer inhibition of bone marrow-derived dendritic cell maturation. International Immunopharmacology 88, 106791. https://doi.org/10.1016/j.intimp.2020.106791.

