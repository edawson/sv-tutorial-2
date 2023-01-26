# Calling structural variants in human genome data

Instructor: Eric Dawson

## Background

Structural variants (SVs) are large (usually defined as >50 basepairs) differences within a genome.
While structural variants occur much less frequently than SNVs and indels[^1], they are important contributors to human phenotypes.
In this exercise, we will call structural variants in short reads from HG002 Genome in a Bottle individual.
We will use svaba[^2]to call variants, then compare these variants to calls from Manta[^3] run on the same sample and a truth set from Genome in a Bottle[^4].

## Data

Genome in a Bottle is an initiative from the US National Institutes for Standards and Technology to provide standardized genomes and variant calls for scientific analysis. HG002 is the child of an Ashkenazi Jewish trio; this individual has been sequenced on many sequencing platforms and their genome has been extensively characterized.
Reads from HG002 have already aligned to hg19 using BWA.


The BAM file is available here: [https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/hg0002.grch37.22.bam](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/hg0002.grch37.22.bam)

and the .bai index file is here:  

[https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/hg0002.grch37.22.bai](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/hg0002.grch37.22.bai)

A reference FASTA file for chromosome 22 (GRCh37 / hg19) is provided here:  

[https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/22.fa](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/22.fa)

Remember: you can download these files with the wget command.

## Setup
You may want to enable clipboard sharing to make copy-pasting links easier. To do so, click the devices->shared_clipboard->bidirectional setting in the VirtualBox menu bar. 

## Task

1. Create a directory called “sv_project” to work in, and cd into it. Hint: remember the “mkdir” command?
2. Download and install prerequisites . **You need only do either the easy or hard instructions (NOT BOTH).**

[^1]: Sudmant et al. 2015, An integrated map of structural variation in 2,504 human genomes. [https://www.nature.com/articles/nature15394](https://www.nature.com/articles/nature15394)
[^2]: Wala et al. 2018, SvABA: genome-wide detection of structural variants and indels by local assembly. [https://genome.cshlp.org/content/early/2018/03/13/gr.221028.117](https://genome.cshlp.org/content/early/2018/03/13/gr.221028.117)
[^3]: Chen et al. 2016, Manta: rapid detection of structural variants and indels for germline and cancer sequencing applications. [https://academic.oup.com/bioinformatics/article/32/8/1220/1743909](https://academic.oup.com/bioinformatics/article/32/8/1220/1743909)
[^4]: Chapman et al 2020, A crowdsourced set of curated structural variants for the human genome. [https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1007933](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1007933)

## Installation: Easy instructions (for conda installations)

For svaba, install using the following command: conda install -c bioconda svaba
For BWA, install using: conda install -c bioconda bwa

## Installation: Hard instructions (for non-conda / advanced instruction)

**Why use the hard instructions?** This is unfortunately a realistic example of what installing bioinformatics software can be like. If you can get familiarized with this process, you’ll have some understanding of what your system administrator does when they install packages for you. You’ll also start to build out the skills to do this yourself.

1. To install svaba and BWA, you’ll need to install the GNU C compiler (GCC) as well as several libraries that are used to compress and decompress BAM files. You can do that in your VM with the following commands:

   1. Update ubuntu:
   
		```bash
   	 	sudo apt-get update
		```
   2. When prompted for a password, type “manager”; type “Y” if asked for permission to continue.
   3. Install GCC / G++ version 9:
   		```bash
		sudo apt-get install gcc-9 g++-9
		```
	4. When prompted for a password, type “manager”; type “Y” if asked for permission to continue.
	5. Tell linux to use G++ and GCC verison 9 as the default. **Note**: you may want to undo this sometimes, and it’s important to remember the default GCC version. You can find this by running gcc --version. **Note**: The order of the below command parameters (i.e., the bit after “--install” is: `<link> <alias> <path> <priority>`

		```bash
		sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 0
		sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 0
		```
	6. Finally, install the dependencies for BWA and svaba (zlib and other compression libraries):

	```bash
	sudo apt-get install zlib1g-dev liblzma-dev liblz4-dev libbz2-dev 
	```

	7. Next, we’ll build svaba. Svaba is available on github at `https://github.com/walaj/svaba` . Use git to clone the repo (hint: try “git clone” followed by the github link) **making sure to use the `--recursive` flag to download all the pieces of the code.**. Your final command should be: `git clone --recursive https://github.com/walaj/svaba`.
	8. `cd` into the newly created “svaba” directory and see what’s in it.
      	1.  We need to run a program called autoconf to find all the dependencies we just installed. Run it by typing `./configure`
	9. We can now build svaba using Make. Type `make` to build svaba from source. **Pro tip: you can make this go faster by running in parallel using `make -j`**
	10. Lastly, type “make install” to install svaba into the “bin” directory. 


	Test it out! Running “./bin/svaba” should show you the usage of the program. 

	Try running `bwa` as well. If you don't have `bwa` installed, you can do it with `conda` in the easy instructions or manually with the following commands:
	
	```bash
		git clone https://github.com/lh3/bwa 
		cd bwa
		make
	```


## Getting our Data

We’ll use wget to grab the remote data files. Just run “wget <file>” to get a specific file.  
Download the .BAI file and the fasta reference listed in the Data section as well.

## Running SVABA to call variants

At this point, you should have a directory with the BAM, the BAI, and the reference genome in it. Now we’ll get started on analyzing our structural variants (after we index our reference genome).

### Create a FASTA index and the BWA index files for the reference genome

1. Use “samtools faidx” to create a FASTA index.
2. Use “bwa index” to create the BWA index files. These are used by svaba to perform targeted realignment. Remember that you will need to use the full path to bwa (e.g., if you’re in your sv_project directory, you would run “./bwa/bwa index 22.fa”). This should take about 1 minute to complete. 
3. Run svaba to detect genomic SV
	1. We need to run svaba in germline mode. Details of how to do so are here: [https://github.com/walaj/svaba#whole-genome-germline-sv-and-indel-detection](https://github.com/walaj/svaba#whole-genome-germline-sv-and-indel-detection).
    Substitute the right file names for GERMLINE_BAM and REF.
	Replace CORES to run using the number of cores assigned to your VM (likely 2).
	Again, remember that you’ll need to use the relative path to svaba (e.g., from your sv_project directory, ./svaba/bin/svaba). (Hint: you just need to run the last line of the first text box - not the comments or any other text boxes.) 
4. Load the BAM file and svaba SV VCF into IGV. You should see the reads as well as a track corresponding to the svaba VCF.
   Look at some of the SVs by zooming in (the zoom slider is in the top right of the IGV window).
5. Do you see any with evidence (e.g., discordant reads or altered depth)? 
6. Are any SVs in genes? If so which ones? If not, why do you think this is the case? 
7. Compare the VCF you made using svaba to the composite callset from Genome in a Bottle (HG002_SVs_Tierand a callset from Manta.
	1. The GIAB SV calls can be downloaded from here: [https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/HG002_SVs_Tier1_v0.6.vcf.gz](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/HG002_SVs_Tier1_v0.6.vcf.gz)
   		Manta calls are available here (you’ll want both the vcf.gz and the vcf.gz.tbi): [https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/manta.hg0002.22.diploidSV.vcf.gz](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/manta.hg0002.22.diploidSV.vcf.gz) and [https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/manta.hg0002.22.diploidSV.vcf.gz.tbi](https://r2-public-worker.atacama.workers.dev/sv-tutorial-2/manta.hg0002.22.diploidSV.vcf.gz.tbi)
	2. You might want to use IGV or bedtools. Remember, you might need to unzip the VCF file(s) first.
	3. Also, examine the SV calls from SVABA. You should see multiple different values for the EVDNC info tag - what do you think they mean? 
	4. Find a high quality SV (or a few) - these will have the strongest combined evidence (ASDIS), a high discordant read MAPQ, and a high alt read MAPQ. Record some of these, look at them in IGV, and see if you can verify and describe them. Take some screenshots and note your findings so you can share them with the class! 

