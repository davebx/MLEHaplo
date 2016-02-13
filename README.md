# MLEHaplo
Maximum Likelihood Estimation for Viral Populations

VIPRA and MLEHaplo README file

Pre-requisites: 
1. multi-dsk: k-mer counting software ( Extension of dsk version 1.5655) : Counts k-mers for multiple values of k simultaneously. The software doesn't combines the counts of forward and reverse complement k-mers, as is performed in traditional k-mer counting softwares
2. perl with modules Bio::Perl, Getopt::Long, Graph


#################Preliminaries#########################

Step 1: Generate k-mer counts file 
Command: 
$ multi-k <fasta/fastq file> <listofkvalues> -d <diskspacelimit> -m <memorylimit>
# <fasta/fastq file> is the file name of the fasta/fastq file. If there are a list of files, use a text file containing locations of all the files, one file per line. Example if there are two files containing paired reads, file1.fastq and file2.fastq, create a file listofiles.txt containing following:
file1.fastq
file2.fastq
<listofkvalues> : contains a list of k values for which k-mer counting has to be performed. For example if k-mer counting is desired for k-values 55,45,35,25. Create a file listofkmers.txt containing these numbers in decreasing order:
55
45
35
25
<diskspacelimit> : defines the limit of temporary disk space used while performing k-mer counting
<memorylimit> : defines the memory limit for storage of temporary hashes while performing k-mer counting 
Output of multi-dsk is a collection of files with extension ".solid_kmers_binary.<kvalue>" in compressed format, which contains counts of k-mers present in the fasta/fastq file. 
Step 2: De-compress the output of multi-dsk
Command: 
parse_results ".solid_kmers_binary.<kvalue> file"  > "fasta/fastq file.kvalue" 

The file "fasta/fastq file.kvalue" now contains the k-mer counts for the fasta/fastq file in the format "k-mer count" per line.
 
Step 3: Generate the De Bruijn graph
Generating the graph needs two files and a parameter. 
1. fasta file containing all the reads 
2. k-mer counts file generated above 
3. a threshold value for ignoring erroneous k-mers

Command: 
Combine paired files into a single file 
perl construct_graph.pl <fastafile> <kmerfile> <threshold> <graphfile> "s"

Output is the <graphfile> containing pairs of k-mers that form edges in the De Bruijn graph. 

Step 4: Create the paired set file 
Create the paired set using the paired reads. It takes as input the two paired files,
file1.fastq and file2.fastq
the k-mer counts file,
file1.kvalue  
and a threshold for ignoring erroneous k-mers

Choice of threshold : Dependent on sequencing coverage. Lower threshold includes more erroneous k-mers in the graph, while higher threshold decreases the number of true k-mers and size of the graph. 

Command:
perl construct_paired_without_bloom.pl -file1 file1.fastq -file2 file2.fastq -paired -kmerfile file1.kvalue -thresh <number> -wr <outputPairedSetfile> 
Output is a file that contains pairs of k-mers on a line and the number of times such pair is observed:
kmer1 kmer2 count


########################VIPRA######################

Running the VIPRA algorithm takes inputs generated above and a parameter for the average insert size, threshold parameter and a value for M (factor) which decides the number of paths to generate per vertex

Command: 
perl dg_cover.pl -graph <graphfileStep3> -kmer <kmerfileStep2> -paired <PairedSetStep4> -fact <M> -thresh <Threshold> -IS <InsertSize> > outputfile

outputfile contains the paths generated from the graph with high paired end supports

Step a: Generate fasta file
Extracting fasta file from outputfile
Command: 
perl process_dg.pl <outputfile> > pathsfastafile

Output: fasta file of the paths generated by VIPRA

Step b: Generate paths file for maximum likelihood estimation
Extracting just the paths in terms of nodes in the graph 
perl get_paths_dgcover.pl -f <outputfile> -w <pathswritefile> 

Output: <pathswritefile>

########################MLEHaplo#########################
Running MLEHaplo takes as input intermediate files generated by VIPRA and pathswritefile generated in Step b

Command: 
perl likelihood_singles_wrapper_parallel.pl -condgraph file1.cond.graph -compset file1.comp.txt -pathsfile <pathswritefile> -back -slow -gl <approximategenomesize>  > <MLEtextfileoutput>

the files cond.graph and comp.txt are outputs generated by VIPRA and contain the condensed graph and compatible sets respectively of the De Bruijn graph and PairedSets. 

Final viral population generation using MLEtextfileoutput

Command:
perl extract_MLE.pl -f <pathsfastafile> -l <MLEtextfileoutput>  > "MLEHaplofastaOUTPUT"


Status API Training Shop Blog About Pricing
© 2016 GitHub, Inc. Terms Privacy Se
