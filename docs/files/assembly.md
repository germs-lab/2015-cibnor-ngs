#Introduction

The largest challenge facing the usage of metagenomic approaches in microbiology is the need to extend traditional microbiology training to include metagenomic or sequencing data analysis. Sean Eddy (a compuational biologist at the Howard Hughes Medical Institute) nicely describes the impacts of high throughput sequencing on biology and its training in his [keynote address](http://cryptogenomicon.org/2014/11/01/high-throughput-sequencing-for-neuroscience/#more-858).

To facilitate the barriers to microbiologists for metagenomic assembly, we have complemented this review with a tutorial of how to estimate the abundance of reference sequences (e.g., genes, contigs, etc.) in a metagenome. We include approaches that include using references that are both (i) available genome references or (ii) assembled from the metagenome. In general, to complete this tutorial and most metagenomic assembly, one would need:

Access to a server. Most metagenomic assembly will require more memory than most researchers will have on their personal computers. In this tutorial, we will provide training on the publicly accessible Amazon EC2 instances which can be rented by anyone with a registered account.

Access to a metagenomic dataset. We have selected the usage of the [HMP Mock Community WGS dataset](http://www.hmpdacc.org/HMMC/) for this tutorial given its availability, practical size, and availability of reference genomes. This dataset represents a mock metagenome of 22 known organisms for which DNA was extracted from cultured isolates, combined, and sequenced.
Software for assembly, read mapping, and annotation. We will demonstrate the installation of this software on an Ubuntu-based server.

Remember, to log in to your EC2 machine: 

    ssh -i MyKeyPair.pem ubuntu@ec2-XX-XX-XX-XXX.compute-1.amazonaws.com

You need to do a couple things to get this tutorial running:


##Update machine

    apt-get update
    apt-get -y install screen git curl gcc make g++ python-dev unzip default-jre pkg-config libncurses5-dev r-base-core r-cran-gplots python-matplotlib sysstat python-setuptools python-pip
	
Copy and paste the following commands one by one into your command line and press ENTER after each one:

    sudo bash

    cd /mnt

    git clone https://github.com/germs-lab/frontiers-review-2015.git



##Download the tutorial dataset.
We will begin this tutorial by downloading the HMP mock metagenome from the NCBI Short Read Archives (SRA). Many public metagenomes are stored as SRA files in the NCBI. The easiest way to get these SRA files is to use a special set of tools called the sratoolkit. If you have your dataset SRA run ID (in this case SRR172903), you can download the dataset and convert it to the standard "fasta" or "fastq" sequencing format is to use a special program to convert the file.

    wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.4.5-2/sratoolkit.2.4.5-2-ubuntu64.tar.gz

    tar -xvf sratoolkit.2.4.5-2-ubuntu64.tar.gz

You can see that we now have a file containing the software with the "ls" command. You'll also see this notebook in the list of files in the present location we are working in.

    ls

Now we'll use the installed sratoolkit program to download the HMP mock dataset in "fastq" format. (This takes a minute or two. You'll find that patience is require for working with metagenomes. The nice thing about working in the cloud is that you are "renting" the computational power so it is not using your personal computer's memory -- freeing it up for things you can do while waiting.

    sratoolkit.2.4.5-2-ubuntu64/bin/fastq-dump SRR172903


##Checking the diversity - What is the distribution of "Who is There?"

An advantage of metagenomic sequencing is the ability to quantify microbial diversity in an environment without the need to first cultivate cells. Typically, most studies access taxonomic diversity (especially with the usage of targeted sequencing of the 16S rRNA gene*). Diversity can also be measured in the representation of specific sequence patterns in a metagenome. For example, one can quantify the abundance of unique nucleotide "words" of length K, or k-mers, in a metagenome. These k-mers can also be used in the assembly of metagenomes where overlapping k-mers are indicative of reads that should be connected together. The diversity of these k-mers can give you insight into the the diversity of your sample. Further, since assembly compares each k-mer against all k-mers, larger numbers of k-mers present will require more computational memory. A nice review on k-mers and assembly is Miller et al.

(*Note that 16S rRNA amplicon sequencing is a targeted approach and not considered metagenomics in this review. Shotgun metagenomic sequencing uses DNA extracted from all cells in a community and sequenced. Targeted sequencing amplifies a specific genomic locus and independently sequenced. A great review on metagenome analysis is Sharpton et al.)

Let's install the software package khmer.

    cd /mnt
    easy_install -U setuptools
    pip install khmer

Let's count the total number of unique k-mers:

    python unique-kmers.py -R unique_count -k 17 SRR172903.fastq

##Getting a sequence coverage profile: What genes are present in my metagenome?

Most metagenomic analysis require one to estimate the abundance of reference genes (e.g., orginating from genomes or one's own metagenomic assembly). This tutorial will cover both cases where references are available or unavailable (requiring de novo assembly).

For the mock HMP metagenome, the HMP has sequenced the genomes of the isolates used for this simulated dataset. The list of these genomes can be obtained on the HMP website, and we provide it here in a Github repository, a tool used for collaboratively sharing data and code. The command below will download data for this tutorial.

    cat ncbi_acc.txt

The following command downloads all the genomes for each ID in the above list into a directory called "genomes".

    cp frontiers-review-2015/fetch-genomes-fasta.py /mnt/.
    cp frontiers-review-2015/ncbi_acc.txt /mnt/.
    python fetch-genomes-fasta.py ncbi_acc.txt genomes

##Estimating abundance of assembled contigs

To estimate the representation of reference genes or genomes in your metagenome, you can align reads to references using read mapping software (e.g., Bowtie2, BWA, etc.). In this tutorial, we will use Bowtie2 which we will install on this server. We will then be mapping the metagenome to a single reference genome (that we downloaded above).

    wget http://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.2.5/bowtie2-2.2.5-linux-x86_64.zip
    unzip bowtie2-2.2.5-linux-x86_64.zip

I've written a script that will automatically map a set of reads to a given reference and output a file containing the number of reads that are mapped to a given reference. To use this script, we'll also need to install samtools. A samfile is a super compressed file that efficiently stores mapped information from mappers. Samtools helps us interact with this file.

    apt-get install samtools

To map reads to a reference, we have provided an easy to use program. The steps the program performs are as follows:

Index your reference genome,
Map reads to your index genome (with default bowtie parameters),
Use Samtools to estimate the number of reads mapped, number of reads unmapped, and provide a tab delimited file with each line consisting of reference sequence name, sequence length, # mapped reads and # unmapped reads.
This takes about 8-10 minutes.

    cp frontiers-review-2015/bowtie.sh /mnt/.
    bash bowtie.sh genomes/NC_000913.2.fa SRR172903.fastq

We can look at the total number of reads mapped and unmapped from our metagenome to the genome NC_000913.2. We can also get a file that shows the reference sequence name (first column), reference sequence length (second column), # mapped reads (third column) and # unmapped reads (last column). The other columns contain information that samtools can use for other queries, you can read about samtools here, http://samtools.sourceforge.net/samtools.shtml.

To look at mapped reads:

    cat reads-mapped.count.txt

To look at UNmapped reads:

    cat reads-unmapped.count.txt

To look at contigs with mapped read information:

    cat reads.by.contigs.txt


If you want a challenge, you can try mapping the metagenome to all reference genomes provided in the genome folder. To do so, try concatentating all genomes into one file (using this command: "cat genomes/*fa >> all-genomes.fa") and running the program on all-genomes.fa instead of NC_000913.2.fa.

##De novo assembly of reference genes.

Assembly of the HMP mock metagenome

Assembly is the process of merging overlapping metagenomic reads from hopefully the same genome into a longer, continguous sequence (most commonly called a contig). It is advantageous in that it provides longer lengths for sequences that can later be used as references (that may be previously unknown), reduces the dataset size for analysis, and provides references that are not dependent on previous knowledge.

The choice of what assembler to use is not an easy one and is a subject of debate (see http://assemblathon.org). It is most important to remember that an assembly is a hypothesized consensus representation of your dataset. The assembly itself is an initial step that needs to be followed by an evaluation of its accuracy and usefulness. For most assemblers, the inputs are sequencing reads and paramters for the assembly software. For this tutorial, we will be completing the assembly with an assembler published in 2014 called Megahit (Li et al., 2015, https://github.com/voutcn/megahit). Sharpton's review (Sharpton, 2014) also reviews quite nicely some of the many assembly programs and approaches for metagenomic assembly.

To reduce the memory that is needed, it is often advantageous to normalize the distribution of k-mers in a metagenome. Removing extraneous information not needed for assembly also removes reads that may contain errors and may improve assembly (http://arxiv.org/abs/1203.4802). These scripts and tutorials are available at http://ged.msu.edu/angus/diginorm-2012/tutorial.html.

For this tutorial, we will assemble our metagenome with the Megathit assembler so first we have to install it.

    cd /mnt
    git clone https://github.com/voutcn/megahit.git
    cd megahit
    make

To run megahit:

    cd /mnt
    megahit/megahit --memory 10e9 -l 250 --k-max 81 -r SRR172903.fastq --cpu-only -o megahit_assembly

To take a look at the assembly, let's run the khmer assembly summary program on it, the final contigs are in megahit_assembly/final.contigs.fa. Let's get statistics on all contigs greater than or equal to 200 bp.

    cp frontiers-review-2015/assemstats3.py /mnt/.
    python assemstats3.py 200 megahit_assembly/final.contigs.fa


##Estimating abundances of contigs

Once the assembly is finished, you have a set of reference contigs which you can now estimate the abundance of the metagenome. The approach for doing so is identical to that shown above where you use reference genomes.

This will take about 20 minutes.

    bash bowtie.sh megahit_assembly/final.contigs.fa SRR172903.fastq

You can take a look at the results of the mapping much like you did above when we were mapping reads to the NCBI genome.

    cat reads-mapped.count.txt
    cat reads-unmapped.count.txt
    cat reads.by.contigs.txt

##Annotating the assembled contigs

Sequencing is often used to determine "who" and/or "what" is in your sample. In our case, we know that the HMP mock community should orignate from a set of genomes (which we actually downlaoded above). One of the most popular tools of comparing an unknown sequence to a known reference is The Basic Local Alignment Search Tool (or BLAST). To identify the origin of our contigs, we will align assembled contigs to the genomes used in the HMP mock community.

Since you have already done the BLAST exercise, why don't you give a try and see if you can annotate the contigs against a database of reference genomes.



