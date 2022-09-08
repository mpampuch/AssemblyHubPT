# AssemblyHubPT
AssemblyHubPT


# From Danish's notebook 

## September 1st, 2022

- Attempted to get UCSC assembly hub working
- Structured assembly hub file structure 
- Created proper files
- Work can be seen in the following repo: https://github.com/DanishZahid1/AssemblyHubPT
- Installed fasta to 2 bit converter
```
conda install -c bioconda ucsc-fatotwobit
conda install -c bioconda/label/cf201901 ucsc-fatotwobit

# Converted fasta to 2bit
faToTwoBit Phaeodactylum_tricornutum.ASM15095v2.dna.toplevel.fa PT.2bit
faToTwoBit: error while loading shared libraries: libssl.so.1.0.0: cannot open shared object file: No such file or directory
[ble: exit 127]

# For some reason I get this error

# Mark suggested using different Conda env (new one)
# That worked
faToTwoBit Phaeodactylum_tricornutum.ASM15095v2.dna.toplevel.fa PT.2bit
<sftp> get PT.2bit

```

- Was eventually able to get the assembly hub working
	- Tomorrow will add tracks
	
- Instructions:
	- Go to https://genome.ucsc.edu/cgi-bin/hgHubConnect?hubId=3613437&hgHub_do_disconnect=on&hgHubConnect.remakeTrackHub=on&hgsid=1440384845_DoCocqhtgEG1XCRVA3ovsM86amaE
	- Connect the hub using the following URL:
		- https://raw.githubusercontent.com/DanishZahid1/AssemblyHubPT/main/myHub/hub.txt
		
		
## September 2nd 2022


(In danish conda)
- Installed GFF to bed conversion package
conda install -c bioconda bedops



(In 2bitConverter conda)
- Installed bedToBigBed program from the binary utilities directory
conda install -c bioconda ucsc-bedtobigbed
- To get chrom sizes
conda install -c bioconda ucsc-twobitinfo


GFF TO bigBed PROCESS
```python
# Convert GFF to bed

conda activate danish
(danish) dzahid@agrajag:/Volumes/ubdata/dzahid/blastTest/perFeature$ gff2bed < ../../seqFinderTool/phatr3_gene_models_with_href.gff3 > phatr3_gene_models_with_href.bed

# Sort bed
sort -k1,1 -k2,2n phatr3_gene_models_with_href.bed > phatr3_gene_models_with_href_sorted.bed

# remove last 4 columns
cut -d$'\t' -f 7,8,9,10 --complement phatr3_gene_models_with_href_sorted.bed > phatr3_gene_models_with_href_clean.bed
sort -k1,1 -k2,2n phatr3_gene_models_with_href_clean.bed > phatr3_gene_models_with_href_clean_sorted.bed

# Get chrom sizes
conda activate 2bitConverter
twoBitInfo PT.2bit stdout | sort -k2rn > PT.chrom.sizes

# Get bigBed file
bedToBigBed -type=bed6+4 phatr3_gene_models_with_href_sorted.bed PT.chrom.sizes phatr3_gene_models_with_href.bb

# Bizzarley things werent generated a working bb file
# Error message seemed to indicate it was because the bed file had . in it (in columns which it should not)
# Replaced all of these with 0
 awk 'BEGIN{OFS=FS="\t"} {gsub("\.","0",$5)}1 {gsub("\.", "Null", $4)}1' phatr3_gene_models_with_href_sorted.bed > PT.bed


# Now got bigBed
# Get bigBed file
bedToBigBed -type=bed6+4 PT.bed PT.chrom.sizes PT.bb


# Fixing that still didn't result in a working bb file
# New error: on using bedToBigBed -type=bed6+4 PT.bed PT.chrom.sizes PT.bb
pass1 - making usageList (90 chroms): 27 millis
Expecting 10 words line 4 of PT.bed got 11

# Tried now fixing the bed file by remove the last 4 columns (extra data that bed dosent usually have)
cut -d$'\t' -f 7,8,9,10 --complement PT.bed > PT_clean.bed

sort -k1,1 -k2,2n PT_clean.bed > PT_clean_sorted.bed

# re-ran
bedToBigBed PT_clean_sorted.bed PT.chrom.sizes PT.bb

# No errors (seemed to work)

```

- Made a database for the new PT genome (generated 2bit file like described earlier)

- Finally got the conversion to the bigBed file working 
- Only problem is because files get converted like this gff -> bed -> bigBed. 
	- When the conversion from gff to bed happens some features don't get named nicely (the 4th column) and so we cant see their names on the annotation track. 
		- But we can click on them to see their chromosome positions. 
- But when we just add a custom track by uploading a file through the browser we can use our gff file (no conversions needed) and so we don't run into this issue there.

**Instructions**:
- Go to https://genome.ucsc.edu/cgi-bin/hgHubConnect?hubId=3613437&hgHub_do_disconnect=on&hgHubConnect.remakeTrackHub=on&hgsid=1440384845_DoCocqhtgEG1XCRVA3ovsM86amaE
- Connect the hub using the following URL:
	- Old
		- https://raw.githubusercontent.com/DanishZahid1/AssemblyHubPT/main/myHub/hub.txt
	- New
		- https://raw.githubusercontent.com/DanishZahid1/AssemblyHubPT/main/myHubNew/hub.txt
- ***To add custom annotation track***:
	- press add custom tracks
	- upload GFF file
	- ![image](https://user-images.githubusercontent.com/97483486/188243178-f9b7fd68-930b-437b-b8c2-639e2c30687b.png)
	- ![image](https://user-images.githubusercontent.com/97483486/188243225-65b2e2d2-04f5-46b6-b0cd-2c1372d85088.png)
	- ![image](https://user-images.githubusercontent.com/97483486/188243320-a16338c8-7624-40e0-9160-5ea7f57d99f8.png)

