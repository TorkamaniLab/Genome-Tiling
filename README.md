{\rtf1\ansi\ansicpg1252\cocoartf2761
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 # Genome Tiling Using Box Scores (2024-04-27)\
\
Developed by Doug Evans, Torkamani Lab, Scripps Research\
\
The files contained in this archive provide a demonstration of genome tiling,\
and is pre-coded for chromosome 9 from the 1000G consortium (vcf not included)\
\
Warning: This code is not well documented or verified. Users are advised to read through\
and understand this code before attempting to integrate into serious applications\
\
References to M and V generally refer to mountains (or peaks) and valleys. \
Mountains are regions of the genome of high correlation (low recombination) and high boxscores, while \
valleys are regions of the genome of low correlation (high recombination) and have low boxscores\
\
### Files included in the repo\
- README.ipynb\\\
  This file\
  \
- BoxScore.Version.2.0.0\\\
  Program for computing Boxscores, outputs to a file\
\
- chr9.corboxscores.corcut.p45.4000.500.txt\\\
  This is an example but real boxscore file produced from BoxScore.Version.2.0.0 for 1000G chr9\
  \
- FindMVMs.Version.2.0.0\\\
  Program for splitting Boxscores based on regions of low and high correlations\\\
  Outputs a file of genomic positions for peaks and valleys of boxscores\\\
  Also outputs a file of genomic positions in an attempt break up large peaks\
\
- chr9.p45.mnts.valleys.tsv\\\
  This is an example, but real, of a mountains and valleys file\
  \
- chr9.p45.4500.mnts.minima.tsv\\\
  This is an example , but real, of a file with additional break points (minima) for some cases where mountains are too big\
  \
- All.chr9.v5b.linenum.AF.txt\\\
  A file derived from chr 9 1000G vcf (See BoxScore File for details on how to create this file)\
\
- ChIPseq_Peaks.YFP_HumanPRDM9.antiGFP.protocolN.p10e-5.sep250.Annotated.txt\\\
  A file containing a list of putative PRDM9 binding sites. This is for reference only and is not used for tiling.\
\
- ALL.chr9.phase3_shapeit2_mvncall_integrated_v5b.20130502.genotypes.vcf.gz\\\
  ALL.chr9.phase3_shapeit2_mvncall_integrated_v5b.20130502.genotypes.vcf.gz.tbi\\\
  These are thes vcf files used as examples as coded, but are are not included in the repo\\\
  They are available at: http://hgdownload.cse.ucsc.edu/gbdb/hg19/1000Genomes/phase3\
\
### The intention of this code\
\
Probably the easiest way to understand the code and files in this repo is to understand the application for which it was intended.\
\
Torkamani lab wished to develop a tool for performing reference free genomic imputation based on deep learning models we would develop.\
If successful, deep learning models would impute missing genotypes based on subregions (tiles) of the genome, quickly, cheaply, and\
more accurately than existing imputions tools.\
\
The reason for creating tiles was to attempt to identify and isolate regions of the genome where recombination rates were low, and to train\
models for each highly correlated region (or tile). Also, we needed small tiles becasue the deep learning algorithm was constrained by the \
number of variants that could be accommodated nicely by GPU VRAM. The goal was to try and split the genome into regions of about 4500 \
variants or so (per tile).\
\
To achieve this, we devised a scoring method (boxscores), where we computed a correlation matrix of common variants (MAF > .05)\
and counted the number of upstream and downstream elements (cells in the matrix) which had threshold values above 0.45 (absolute value = 0.45).\
This size (of the box) determines how far upstream and downstream of a pair of genomic loci we need to compute the correlation matrix. The score\
for each locus (loci) is the sum of the cells in that matrix that had correlations above 0.45. If no loci correlate above or below a locus (loci)\
the boxscore is near zero, and this is referred to as a valley. The opposite it true for highly correlated variants upstream to downstream\
of a pair of variant loci, with high boxscores.\
\
The easiest way to envision this is as an integrating (sum) as a box sliding along the top of a diagonal within a larger thresholded correlation matrix.\
The trick is, we cannot do a whole VCF at once. So we break the matrix up into small windows referred to blocks. Each block is a large correlation matrix and\
should be much bigger than the boxsize. There are tradeoffs for setting blocks too large due to memory needs and unnecessary computational overhead.\
\
Finally, box scores cannot be computed fully at the beginning and ending of the chromosome, because the box crowds the boundary. So a number\
of variants (boxsize) are not computed at the upper and lower boundary of the chromosome and are padded with preset values (-1) for easy detection\
by downstream tools in the scores file.\
\
Once a box score file is created by the BoxScore program, the FindMVMs file can be used to discover valleys, and them used to split up the chromosome into blocks.\
This first step is to simply look for very low values of boxscore and tag them as valleys. Valleys and mountains can then be annotated, along with the\
variant count in each mountain and valley. \
\
Unfortunately, we were not always successful in getting small enough tiles to fit into GPU VRAM, so a second bit\
of code is used to split large mountains up further using binning and discovering local minimum. If no local minima of suitable size can be found, then\
downstream applications will be forced to break up larger remaining mountains manually into smaller tiles.\
}