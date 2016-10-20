# README

This is the software package for the Read-Split-Fly pipeline. Included are the scripts and software necessary to run the entire process from beginning to end.

## INSTALLATION

### Installing the pipeline:

decompress rsr.tar.gz (tar -zxvf rsr.tar.gz) to the location of your choice; this is the **INSTALLATION DIRECTORY**.

### Perl 5.16 (or later):

The converter for the gene reference file (see below) requires Perl to run. We tested it with version 5.16, though it is likely that earlier versions will work. [Download Perl](http://www.perl.org). 

To verify your installation of Perl is compatable, check the output of the command: 
- *perl --version*


### Python 2.7 (or later):

The encoding guesser requires that python be installed on the system and its executable in an accessible location. Earlier versions may work, but that isn't guaranteed.  [Download Python](https://www.python.org).

To verify your installation of python is compatable, check the output of the command:
- *python --version*

### Bowtie 1.0.1 (or later): 

Ensure that you have bowtie (version 1.0.1 or later) install. By default, Read Split Fly will look for bowtie in the **INSTALLATION DIRECTORY**/bowtie. If you install it in another location, make sure it is in the PATH (see also: **CONFIGURATION** and **DEPENDENCIES** below; it is recommended that the path to bowtie be in the PATH anyway). [Download Bowtie](http://bowtie-bio.sourceforge.net)

To verify your installation of bowtie is compatable, check the output of the command: 
- *bowtie --version*

### gcc(g++) version 4.8 (or later):

Compiling the associated programs requires gcc version 4.8 or higher (to accommodate features of c++11 that are used in splitpairs).

To verify your installation of bowtie is compatable, check the output of the command:
- *gcc --version*

When gcc is installed, one can easily create the necessary binary programs from the src directory by changing directory to the **INSTALLATION DIRECTORY** and typing *make*. Note that binaries are included in the package. These binaries were compiled on linux, and may or may not work on your system.  It is recommended to run *make* to compile binaries for your system before use.

### Bowtie Index files:

You will need bowtie index files and knownGene files for the genome(s) of your choice. Put the index files in the indexes subdirectory of your bowtie install directory (if you cannot install there, any directory will work). Set the **BOWTIE\_INDEXES ENVIRONMENT** variable to the location of the indexes (this is exactly the same as the requirement for running bowtie by itself). 

Indexes can be found on the [bowtie website](http://bowtie-bio.sourceforge.net). We use the [human hg19](ftp://ftp.ccb.jhu.edu/pub/data/bowtie_indexes/hg19.ebwt.zip) and [mouse mm9](ftp://ftp.ccb.jhu.edu/pub/data/bowtie_indexes/mm9.ebwt.zip) genomes.

**Note**: Normally, bowtie will attempt to guess where the index files are, but because of the necessities below, we cannot do that. You *must* set the variable.

Setting the variable examples: 
- bash: *export BOWTIE\_INDEXES=/usr/local/bowtie/indexes*
- csh:    *setenv BOWTIE\_INDEXES /usr/local/bowtie/indexes*


### refFlat files:

For the genomes to which you plan to align, download and uncompress their refFlat reference file from [UCSC](http://hgdownload.cse.ucsc.edu/downloads.html). Place the refFlat.txt file in the same directory as your bowtie indexes, and **change the name of the file to have the following pattern**: OrganismAssemblyName.refFlat.txt

- Human Assembly hg19: *hg19.refFlat.txt*
  - [Human hg19 reference](http://hgdownload.cse.ucsc.edu/goldenPath/hg19/database/refFlat.txt.gz)
- Mouse Assembly mm9sp35: *mm9sp35.refFlat.txt*
  - [Mouse mm9 reference](http://hgdownload.cse.ucsc.edu/goldenPath/mm9/database/refFlat.txt.gz)

**Note**: These files are case sensitive.

The splitPairs portion of Read-Split-Fly requires a special parsed refFlat reference with intron/exon boundaries identified. We provide a script to create this file called *refflat\_parse\_RSW.pl* in the **RSF INSTALLATION DIECTORY**. Run this script and supply the refFlat reference file as the input argument.

- *perl refflat_parse_RSW.pl /usr/local/bowtie/indexes/hg19.refFlat.txt*
  - This generates the annotated file *hg19.refFlat.txt.intronBoundary.exonsgaps*

## CONFIGURATION (optional):

There is no mandatory configuration that needs be done if you have followed the installation instructions to this point. The following are presented as options for the advanced user.

The configuration file *rsr_config.sh* contains all the configurable values used by the pipeline. It can be edited with any plain-text editor (nano, vim, etc). You can change the following variables in the *USER CONFIGURATION* section to suit your needs:

- ***BOWTIE_TEMP_DIR:***     location to store intermediate bowtie files.
  - Default: **INSTALLATION DIRECTORY**/tmp/bowtie

- ***SPLIT_TEMP_DIR:*** location to store intermediate split reads files.
  - Default: **INSTALLATION DIRECTORY**/tmp/split
                    
- ***RSR_TEMP_DIR:*** location to store intermediate RSR options and output files.
  - Default: **INSTALLATION DIRECTORY**/tmp/splitPairs

- ***LOG_DIR:*** location to store diagnostic and operational logs.
  - Default: **INSTALLATION DIRECTORY**/logs

- ***BOWTIE_PROGRAM:*** the absolute path to bowtie executable (e.g. /usr/bin/bowtie).
  - Default: **INSTALLATION DIRECTORY**/bowtie/bowtie
    - If the bowtie executable is not in this directory, RSF will check the PATH.

- ***REFDIR:*** Directory containing refFlat files (and gene Intron/exon boundary files). All files for your available genomes must be in this directory.
  - Default: BOWTIE_INDEXES

- ***BASES_TO_TRIM:*** How many bases should be trimmed off the right-end of your reads?  Set this according ot the quality of your reads.
  - Default: 0.`

All other variables are internal-use and should not be changed. Read the comments in the configuration file for details as to what function they provide.  

## DEPENDENCIES:

All shell scripts (files ending in .sh) rely on "rsf_config.sh," which holds the configuration data. Other scripts call upon other execuables as needed, depicted below.

    rsf_batch_job.sh
        -pipeline.sh
        -bowtie.sh
            -guess-encoding.py
                -Python 2.7 or newer
            - bowtie v1.0.1 or newer
        -split.sh
            -srr
        -sfc 
        -splitPairs.sh
            -sp4
    compare_sh
        -compare
    
    refFlat_parse_RSW.pl
        Perl 5.16 or newer
    sbc
        -None

## RUNNING:

To run the pipeline, first, set the ***BOWTIE\_INDEXES*** environment variable to the location of your bowtie indexes directory according to your shell:

If your indexes are in */usr/share/bowtie/indexes*, type:
- bash: *export BOWTIE_INDEXES=/usr/share/bowtie/indexes*
- csh: *setenv BOWTIE_INDEXES /usr/share/bowtie/indexes*

After that, you can execute rsw_batch_job.sh with the following inputs:

- **mode:** *<string>*
  - *analytic* or *comparison*
  - Analytic jobs produce RSF output files. This is the standard mode.
  - Comparative jobs produce RSF output files for both data sets and a file which shows the differences between the two.

- **genome:** *<string>*
  - The assembly name of the bowtie index for the genome to which to align reads. Also specifies which refFlat file to use (see INSTALLATION).

- **readsFile:** *<double-quoted string>*
  - The file(s) with RNA-Seq data in plain-text FASTQ format. The nature of your run will determine how you should specify your files.
    - **Single-ended, no replicates:**
      - *"file_name_with_full_path"*
    - **Single-ended with replicates:**
      - *"replicate1.fastq,replicate2.fastq,\.\.\."*
    - **Paired-ended, no replicates:**
      - *"left-data.fastq|right-data.fastq"*
    - **Paired-ended with replicates:** 
      - *"Replicate1_1.fastq,Replicate2_1.fastq|Replicate1_2.fast1,Replicate2_2.fastq"*
      - Make sure your pairs are ordered correctly:
        - "REPLICATE 1 LEFT, REPLICATE 2 LEFT | REPLICATE 1 RIGHT, REPLICATE 2 RIGHT"
- **[readsFile2]:** *<optional string>*
  - For use in *comparison* mode, a second set of reads-files goes here,the format is the same as above.

- **maxGoodAlignments:**   *<integer>* 
  - Maximum number of matches allowed in bowtie (see bowtie -k and -m parameters).

- **minSplitSize:** *<integer>*        
  - Smallest length to split your reads into. If you specify more than half the reads' length, the pipeline will exchange it with (readlength - minSplitSize).
    - The smaller your split, the more memory, disk space, and time will be needed.

- **minSplitdistance:** *<integer>*
  - Minimum distance allowed between split-reads to be considered a splice-junction candidate.

- **maxSplitdistance:** *<integer>*
  - Largest amount of distance between split-reads to be considered a splice-junction candidate.

- **regionBuffer:** *<integer>*
  - Maximum distance between the start-position of candidate junctions considered for support.

- **requiredSupports:** *<integer>*
  - Minimum number of supporting reads a splie-junction candidate must have to be reported.

- **pathToSaveFesults:** *<string>*
  - Path to directory where RSF results will be stored.

- **BLAST e-value:** *<decimal>*
  - e-value passed to BLAST to query RSF results against miRNA and u12db databases.
  - If set to 0 or lower, this post-processing step will be ignored.

### Examples:

Here are presented example command-lines for doing various kinds of runs the assembly names are real but the file names are made-up...

#### Analytic runs:
##### Normal Run: 
*rsw_batch_job.sh analytic mm9 "mus1.fastq" 11 11 3 30000 4 2 ~/mm9_results 0.1*

##### Normal Run with Replicates:
*rsw_batch_job analytic hg19 "hg19-1.fastq,hg19-2.fastq,hg19-3.fastq" 2 30 2 50000 5 2 ~/hg19_results 0.01*

##### Paired Run: 
*rsw_batch_job.sh analytic hg19sp101 "hg19_1.fastq|hg19_2.fastq" 11 33 3 100000 5 2 ~/hg19_paired_results .001*

##### Paired-ended Run with Replicates:
*rsw_batch_job analytic hg19sp101 "hg19-1_1.fastq,hg19-2_1.fastq|hg19-1_2.fastq,hg19-2_2.fastq" 11 33 3 50000 5 2 ~/hg19_pair_repl_results .1*

#### Comparative runs:
##### Normal Run: 
*rsw_batch_job.sh comparison mm9 "set1.fastq" "set2.fastq" 2 15 3 30000 4 2 ~/mm9_compare 0.1*

##### Normal Run with Replicates:
*rsw_batch_job analytic hg19 "set1replicate1.fastq,set1replicate2.fastq" "set2replicate1.fastq,set2replicate2.fastq" 2 33 3 50000 5 2 ~/hg19_results 0.1*

##### Paired Run:
*rsw_batch_job.sh analytic hg19sp101 "set1_1.fastq|set1_2.fastq" "set2_1.fastq|set2_2.fastq" 11 33 3 50000 5 2 ~/hg19_paired_results 0.1*

# KNOWN ISSUES

The quality-encoding detection portion of bowtie.sh is known to cause a broken pipe with awk. This is acceptable and does not interfere with the performance of the pipeline.

# COPYRIGHT
For questions, please contact Jeff Kinne <jkinne@cs.indstate.edu>

Read-Split-Run is copyright(c) 2014-2015 Yongsheng Bai, Brandon Donham, Randal J. Kaufman, Jeff Kinne.
<br>Read-Split-Fly is copyright(c) 2015-2016 Yongsheng Bai, Jeff Kinne, Aaron Cox, Feng Jiang.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
a copy is also provided in the LICENSE file, accompanying this.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## BOWTIE
This package of Read-Split-Fly installs Bowtie version 1.1.2 and uses it by default. 
Bowtie is licensed under the Artistic License, a copy of which can be [found here](bt/LICENSE).
We have modified the Makefile and bowtie_inspect.cpp, which we are including as bowtie_inspect_RSR.cpp.

These modified files are being released as part of Read-Split-Fly under the Apache License, Version 2.0 .

## Downstream Processing
To further extend the userfulness of the Read-Split-Fly software, we have built in optional downstream
processing into the pipeline. We use the BLAST+ suite to compare various nucleotide sequences found in the miRBase
and U12DB databases against the candidate splice junctions identified by Read-Split-Fly.

### BLAST+
[BLAST Homepage](ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/)

[BLAST+ Article](https://www.ncbi.nlm.nih.gov/pubmed/20003500?dopt=Citation)

### miRBase
[miRBase Homepage](http://www.mirbase.org/)

[miRBase Article](http://nar.oxfordjournals.org/content/42/D1/D68)

### U12DB
[U12DB Homepage](http://genome.crg.es/datasets/u12/)

[U12DB Article](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1635337/)

