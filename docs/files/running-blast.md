The goal of this tutorial is to run you through a demonstration of the
command line, which you may not have seen or used much before.

Prepare for this tutorial by working through
amazon/start-up-an-ec2-instance, but follow the instructions to start up
amazon/starting-up-a-custom-ami instead; use AMI ami-7606d01e.

All of the commands below can and should be copy/pasted rather than
re-typed.

Note: on Windows using TeraTerm, you can select the commands in the Web
browser, then go to TeraTerm and click your right mouse button to paste.
On Mac OS X using Terminal, you can select the commands in the Web
browser, use Command-C to copy, and then go the terminal and use
Command-V to paste.

Switching to root
=================

Start by making sure you're the superuser, root:

    sudo bash

Updating the software on the machine
====================================

Copy and paste the following two commands :

    apt-get update
    apt-get -y install screen git curl gcc make g++ python-dev unzip \
            default-jre pkg-config libncurses5-dev r-base-core \
            r-cran-gplots python-matplotlib sysstat

(make sure to hit enter after the paste -- sometimes the last line
doesn't paste completely.)

If you started up a custom operating system, then this should finish
quickly; if instead you started up Ubuntu 14.04 blank, then this will
take a minute or two.

Install BLAST
=============

Here, we're using curl to download the BLAST distribution from NCBI;
then we're using 'tar' to unpack it into the current directory; and then
we're copying the program files into the directory /usr/local/bin, where
we can run them from anywhere. :

    cd /root

    curl -O ftp://ftp.ncbi.nih.gov/blast/executables/release/2.2.26/blast-2.2.26-x64-linux.tar.gz
    tar xzf blast-2.2.26-x64-linux.tar.gz
    cp blast-2.2.26/bin/* /usr/local/bin
    cp -r blast-2.2.26/data /usr/local/blast-data

OK -- now you can run BLAST from anywhere!

Again, this is basically what "installing software" means -- it just
means copying around files so that they can be run, and (in some cases)
setting up resources so that the software knows where specific data
files are.

Running BLAST
=============

Try typing:

    blastall

You'll get a long laundry list of output, with all sorts of options and
arguments. Let's play with some of them.

First! We need some data. Let's grab the mouse and zebrafish RefSeq
protein data sets from NCBI, and put them in /mnt, which is the scratch
disk space for Amazon machines :

    cd /mnt

    curl -O ftp://ftp.ncbi.nlm.nih.gov/refseq/M_musculus/mRNA_Prot/mouse.1.protein.faa.gz
    curl -O ftp://ftp.ncbi.nlm.nih.gov/refseq/D_rerio/mRNA_Prot/tmpold/zebrafish.protein.faa.gz

If you look at the files in the current directory, you should see both
files, along with a directory called lost+found which is for system
information:

    ls -l

should show you:

    drwx------ 2 root root   16384 2013-01-08 00:14 lost+found
    -rw-r--r-- 1 root root 9454271 2013-06-11 02:29 mouse.1.protein.faa.gz
    -rw-r--r-- 1 root root 8958096 2013-06-11 02:29 zebrafish.protein.faa.gz

Both of these files are FASTA protein files (that's what the .faa
suggests) that are compressed by gzip (that's what the .gz suggests).

Uncompress them :

    gunzip *.faa.gz

and let's look at the first few sequences:

    head -11 mouse.1.protein.faa

These are protein sequences in FASTA format. FASTA format is something
many of you have probably seen in one form or another -- it's pretty
ubiquitous. It's just a text file, containing records; each record
starts with a line beginning with a '\>', and then contains one or more
lines of sequence text.

Let's take those first two sequences and save them to a file. We'll do
this using output redirection with '\>', which says "take all the output
and put it into this file here." :

    head -11 mouse.1.protein.faa > mm-first.fa

So now, for example, you can do 'cat mm-first.fa' to see the contents of
that file (or 'less mm-first.fa').

Now let's BLAST these two sequences against the entire zebrafish protein
data set. First, we need to tell BLAST that the zebrafish sequences are
(a) a database, and (b) a protein database. That's done by calling
'formatdb' :

    formatdb -i zebrafish.protein.faa -o T -p T

Next, we call BLAST to do the search :

    blastall -i mm-first.fa -d zebrafish.protein.faa -p blastp

This should run pretty quickly, but you're going to get a LOT of
output!! What's going on? A few things --

> -   if you BLAST a sequence against a large database, odds are it will
>     turn up a lot of spurious matches. By default, blastall uses an
>     e-value cutoff of 10, which is very relaxed.
> -   blastall also reports the first 100 matches, which is usually more
>     than you want.
> -   a lot of proteins also have trace similarity to other proteins!

For all of these reasons, generally you only want the first few BLAST
matches, and/or the ones with a "good" e-value. We do that by adding '-b
2 -v 2' (which says, report only two matches and alignments); and by
adding '-e 1e-6', which says, report only matches with an e-value of
1e-6 or better :

    blastall -i mm-first.fa -d zebrafish.protein.faa -p blastp -b 2 -v 2 -e 1e-6

Now you should get a lot less text! (And indeed you do...) Let's put it
an output file, 'out.txt' :

    blastall -i mm-first.fa -d zebrafish.protein.faa -p blastp -b 2 -v 2 -o out.txt

The contents of the output file should look exactly like the output
before you saved it into the file -- check it out:

    cat out.txt

Now, try

    blastall -i mm-first.fa -d zebrafish.protein.faa -p blastp -b 2 -v 2 -o out.txt -m 8 

What did that do?

