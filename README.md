![Logo](https://github.com/bcgsc/ntEdit/blob/master/ntedit-logo.png)

# ntEdit

## Scalable genome assembly polishing
## 12/2018
## email: rwarren [at] bcgsc [dot] ca


### Description
-----------

ntEdit is a genomics application for polishing genome assembly drafts.
ntEdit simplifies polishing and "haploidization" of gene and genome sequences with its re-usable Bloom filter design.
We expect ntEdit to have additional applications in fast mapping of simple nucleotide variations between any two individuals or species’ genomes.


### Implementation and requirements
-------------------------------

ntEdit v2.0 is written in C++. 


### Install
-------

Clone this directory and enter this directory.
<pre>
https://github.com/bcgsc/ntEdit.git
cd ntEdit
</pre>
Compile ntEdit
<pre>
make ntedit
</pre>


### Dependencies
-------

1. ntHits (https://github.com/bcgsc/nthits)
2. BloomFilter utilities (provided in ./lib)
3. kseq (provided in ./lib)


### Documentation
-------------

Refer to the README.md file on how to run ntEdit and our manuscript for information about the software and its performance 
Questions or comments?  We would love to hear from you!
rwarren@bcgsc.ca


### Citing ntEdit 
------------

<pre>
René L Warren, Lauren Coombe, Hamid Mohamadi, Jessica Zhang, Barry Jaquish, Nathalie Isabel, Steven JM Jones, Jean Bousquet, Joerg Bohlmann and Inanç Birol
ntEdit: scalable genome assembly polishing
TBD
</pre>

The experimental data described in our paper can be downloaded here: http://www.bcgsc.ca/downloads/btl/ntedit/
Thank you for using, developing and promoting this free software.


### Credits
-------

ntEdit:
Rene Warren

nthits / nthash / BloomFilter.pm:
Hamid Mohamadi (recursive/ntHash)

C++ implementation
Jessica Zhang


### How to run in a pipeline
-------

1. Running nthits (please see nthits documentation)
nthits -c <kmer coverage threshold> -k <kmer length> -j <number of threads> reads
eg.
./nthits -c 1 --outbloom --solid -k 25 -j 48 Sim_100_300_1.fq Sim_100_300_2.fq
or
./nthits -c 1 --outbloom --solid -k 25 -j 48 @reads.in

Where @reads.in is a file listing the path to all fastq files


2. Running ntEdit (see complete usage below)
./ntedit -f <fasta file to polish> -k <kmer length> -r <Bloom filter from nthits>
eg.
./ntedit -f ecoliWithMismatches001Indels0001.fa -r solidBF_k25.bf -k 25 -b ntEditEcolik25


### Running ntEdit
-------------
<pre>
e.g. ./ntedit -f ecoliWithMismatches001Indels0001.fa -r solidBF_k25.bf -k 25 -b ntEditEcolik25

Usage: ../ntEdit.pl [v1.0.1]
 Options:
	-t,	number of threads [default=1]
	-f,	Draft genome assembly (FASTA, Multi-FASTA, and/or gzipped compatible), REQUIRED
	-r,	Bloom filter file (generated from ntHits), REQUIRED
	-b,	output file prefix, OPTIONAL
	-k,	kmer size, REQUIRED
	-z,	minimum contig length [default=100]
	-i,	maximum number of insertion bases to try, range 0-5, [default=4]
	-d,	maximum number of deletions bases to try, range 0-5, [default=5]
	-x,	k/x ratio for the number of kmers that should be missing, [default=5.000]
	-y, 	k/y ratio for the number of editted kmers that should be present, [default=9.000]
	-v,	verbose mode (-v 1 = yes, default = 0, no)

	--help,		display this message and exit 
	--version,	output version information and exit
Report bugs to rwarren@bcgsc.ca

</pre>


### Test data
---------
<pre>
Go to ./demo
</pre>
(cd demo)


run:
-------------------------------------
./runme.sh (../ntedit -f ecoliWithMismatches001Indels0001.fa.gz -r solidBF_k25.bf -k 25 -b ntEditEcolik25)

ntEdit will polish an E. coli genome sequence with substitution error ~0.001 and indels ~0.0001 using pre-made nthits Bloom filter

Expected files will be:
ntEditEcolik25.log
ntEditEcolik25_changes.tsv
ntEditEcolik25_edited.fa


### How it works
------------
![Logo](https://github.com/bcgsc/ntEdit/blob/master/figS1.png)
Sequence reads are first shredded into kmers using ntHits, keeping track of kmer multiplicity. The kmers that pass coverage thresholds (ntHits, -c option) are used to construct a Bloom filter (BF). The draft assembly is supplied to ntEdit (-f option, fasta file), along with the BF (-r option) and sequences are read sequentially. Draft assembly contigs are shredded into kmers (at a specified –k value matching that used to build the BF), and each kmer from 5’ to 3’ queries the BF data structure for presence/absence (step 1). When kmers are not found in the filter, a subset (k/3) of k kmers containing the 3’-end base is queried for absence (step 2). When >=k/5 kmers (by default, -x option) are absent, editing takes place, otherwise step 1 resumes and the next assembly kmer is assessed. The 3’-end base is permuted to one of the three alternate bases (step 3), and a subset of k kmers containing the change is assessed (>= k/9 by default, -y option). When a base substitution is made that qualifies, it is tracked along with the number of supported kmers and the remaining alternate 3’-end base substitutions are also assessed (ie. resuming step3 until all bases inspected). If it does not qualify, then a cycle of base insertion(s) (step 4) and deletion(s) (step 5) begins. As is the case for the substitutions, a subset of k kmers containing the indel change is assessed (>= k/9 by default, -y option). If there are no qualifying changes, then the next alternate 3’-end base is inspected as per above; otherwise the change is applied to the sequence string and the next assembly kmer is inspected (step 1). The process is repeated until a qualifying change or until no suitable edits are found (back to step 1).  

### OUTPUT FILES
------------------------

|Output files|                    Description|
|---|---|
|_changes.tsv                 | tab-separated file; ID      bpPosition+1    OriginalBase    NewBase Support 25-mers (out of k/3)   AlternateNewBase   Alt.Support k-mers   eg. U00096.3_MG1655_k12     117     A       T       9|
|_edited.fa                   | fasta file; contains the polished genome assembly |



### License
-------

LINKS Copyright (c) 2018-2019 British Columbia Cancer Agency Branch.  All rights reserved.

LINKS is released under the GNU General Public License v3

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, version 3.
 
This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.
 
You should have received a copy of the GNU General Public License
along with this program. If not, see <http://www.gnu.org/licenses/>.
 
For commercial licensing options, please contact
Patrick Rebstein <prebstein@bccancer.bc.ca>
