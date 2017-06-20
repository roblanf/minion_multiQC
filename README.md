# Multi-tool QC for data from Oxford Nanopore's Minion Sequencer

## What?

A single script that is adapted to perform QC with a range of popular tools on the output of a single flowcell of Oxford Nanopore's Minion sequencer.

## Why? 

Becuase there are lots of good tools, and if you really want to know whether your data are any good, it's useful to use many of them at once.

## Dependencies / tools used

Get a coffee - there are a few things to install first...

* Albacore: https://community.nanoporetech.com/protocols/albacore-offline-basecalli/v/abec_2003_v1_revs_29nov2016/run-albacore-on-linux
* minion_QC.R: https://github.com/roblanf/minion_qc
* NGMLR: https://github.com/philres/ngmlr
* qualimap: http://qualimap.bioinfo.cipf.es/
* samtools: https://github.com/lh3/samtools
* miniasm: https://github.com/lh3/miniasm
* Quast: http://quast.sourceforge.net/download.html

Install all of these, and then make sure everything is somewhere in your path so that you can run each tool succesfully as follows:

```
read_fast5_basecaller.py # albacore
minion_QC.R
ngmlr
qualimap 
samtools
minimap
miniasm
quast.py
```

Once you have all of those up and running, you can run the script.

## Settings

I like to hard-code the settings in the script. For each flowcell I just copy the script into the folder with the data, change the settings below, and then run it (see below for how to run it). I do it this way so that the script itself provides the record of exactly what I did to QC each flowcell. 

* **inputf**: an input folder of raw fast5 files from the minion. Fine if it has lots of subfolders.
* **outputbase**: the output file where you want all the processed data, QC reports, and summary stats to go. Output will be stored in subfolders of this. I usually just set it to be the parent folder of the input folder, as below, so that the QC and the raw data are in the same place.
* **ref**: The reference genome. Fine if it's a distant reference - we just use the mapping stats to get a rough idea of the what's in the reads, and to compare flowcells.
* **gff**: a gff of genes in your reference genome
* **threads**: how many cores to use for all the analyses
* **mem_size**: how much memory can you give to Qualimap. You may need to reduce this if you're running this on a desktop with <50G. WE have this here becuase qualimap sometimes needs a lot of memory.
* **flowcellID**: the ID of your flowcell. Albacore needs this
* **kitID**: the ID of your library prep kit. Albacore needs this too.

Here's an example

```
inputf="/data/nanopore/RB7_A2/20170609_0424_Epauciflora_A2/"
outputbase="/data/nanopore/RB7_A2/"
ref="/data/active_refs/Epau.fa.gz"
gff="/data/active_refs/Egrandis_genes_chr1_to_chr11.gff3"
threads=55 # number of threads to use
mem_size='50G'
flowcellID="FLO-MIN107"
kitID="SQK-LSK108"
```

## Running it

Easy:

```
sh minion_multiQC.sh
```

## Overview of what it does

1. Basecall with Albacore
2. QC plots and stats on the called bases using minion_QC.R
3. Map to the reference with NGMLR
4. Run qualimap on the whole reference, and just the genes (we do the latter because you often expect better mapping to genes with a distant reference, because genes tend to be more conserved)
5. Assess the mapping of reads at various lengths with samtools
6. Assemble with minimap/miniasm
7. Assess assembly with Quast

## Output

A lot. Dig around in the folders and files, and look at the websites for each tool to get a handle on the information.

## How long does it take?
On a server with 30 cores assigned, the whole thing runs in a few hours on a flowcell with ~4GB of data. 

