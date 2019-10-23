[![Echidna cartoon](ekidna.png)](https://www.kisspng.com/png-hedgehog-porcupine-echidna-illustration-vector-cut-392690/)
[![Build Status](https://travis-ci.org/tseemann/ekidna.svg?branch=master)](https://travis-ci.org/tseemann/ekidna)
[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
![Don't judge me](https://img.shields.io/badge/Language-Perl_5-steelblue.svg)

:warning: This software is still in early development

# ekidna
Assembly based core genome SNP alignments

## Introduction

In an ideal world, to determine a core genome amongst a set of genomes, we
would perform a "multiple whole genome alignment" and extract the conserved
sites (mono- and poly- morphic). Software like 
[Mauve](http://darlinglab.org/mauve/mauve.html) can do this, but it does
not scale to more than 10s of genomes, due to the exponential computational
need.

Instead, we usually choose a reference genome and align isolate genomes sequentially 
to the reference. The "genomes" could be already assembled genomes (contigs
in FASTA) or raw sequencing data (reads in FASTQ). Tools like 
[ParSNP](https://github.com/marbl/parsnp) 
and
[Roary](https://github.com/sanger-pathogens/Roary)
can achive this using assemblies.
Many 
[SNP calling pipelines](https://thegenomefactory.blogspot.com/2018/10/a-unix-one-liner-to-call-bacterial.html)
will combine SNPs into a core genome alignment.
My SNP pipeline 
[Snippy](https://github.com/tseemann/snippy) 
will accept both assemblies and reads, but internally shreds the assemblies
into fake reads rather than use the contigs natively.

One of [Heng Li](https://en.wikipedia.org/wiki/Heng_Li)'s past experiments
was [fermikit](https://github.com/lh3/fermikit) which did "rough" _de novo_
assemblies and then aligned them to the reference, and called SNPs.  One of
the advantages of this is improved calling of
[indels](https://en.wikipedia.org/wiki/Indel).

## Another experiment

Here we present `ekidna` which will accept either reads or contigs.
Reads will be assembled in a _fast_ and _conservative_ method into contigs.
A reference will be chosen from the contig sets, and the remainder contigs
will be directly aligned to the reference using `minimap2` and variants
called using `paftools`. VCF files will then be combined into a core genome
alignment suitable for building phylogenies for population analysis.

## Synposis

```
% ekidna -t -o outdir *.fna *_R1.fastq.gz
<snip>

% figtree outdir/ekidna.nwk
<admire the SNP resolution population structure>
```

## Installation

### Conda
Install [Conda](https://conda.io/docs/) or [Miniconda](https://conda.io/miniconda.html):
```
conda install -c bioconda ekidna  # COMING ONE DAY
```

### Homebrew
Install [HomeBrew](http://brew.sh/) (Mac OS X) or [LinuxBrew](http://linuxbrew.sh/) (Linux).
```
brew install brewsci/bio/ekidna  # COMING ONE DAY
```

### Source
This will install the latest version direct from Github.
You'll need to add the Ekidna `bin` directory to your `$PATH`,
and also ensure all the [dependencies](#Dependencies) are installed.
```
cd $HOME
git clone https://github.com/tseemann/ekidna.git
$HOME/ekidna/bin/ekidna --help
```

## Interface

```
USAGE
  ekidna [options] -o <outdir> <SAMPLE1 SAMPLE2 SAMPLE3 ...>
SAMPLES
  Contigs    contigs.{fna,gff,gbk}[.gz] (assembled genomes)
  Reads      R1.{fq,fastq}[.gz] (only want R1)
OPTIONS
  -h         Print this help
  -v         Print version and exit
  -q         No output while running, only errors
  -k         Keep intermediate files
  -o OUTDIR  Output folder [mandatory]
  -p PREFIX  Prefix for output files [ekidna]
  -j CPUS    Number of CPU threads to use [1]
  -m MINLEN  Minimum alignment size to consider [500]
  -a ASMCMD  Assember command [skesa ...]
  -t         Also build tree
```

## Input files

### Assembled genomes or contigs
* FASTA, Genbank, EMBL, GFF ; optionally compressed with gzip, bzip2, zip

### Read sequences
* FASTQ ; optionally compressed with gzip
* These will be assembled rapidly and roughly into contigs
* Only one FASTQ file is accepted ; suggest `_R1` if you have paired reads

## Usage

```
% cd test

% ls
NC_018594.fna.gz  NC_021004.fna.gz  NC_021006.fna.gz  NC_021028.fna.gz
NC_021003.fna.gz  NC_021005.fna.gz  NC_021026.fna.gz

% ekidna -o outdir *.fna.gz
<snip>

% ls outdir
ekidna.aln  ekidna.fna  ekidna.full.aln  ekidna.log  ekidna.vcf

% bcftools stats outdir/ekidna.vcf | grep ^SN
SN      0       number of samples:      6
SN      0       number of records:      34157
SN      0       number of no-ALTs:      0
SN      0       number of SNPs: 34157
SN      0       number of MNPs: 0
SN      0       number of indels:       0
SN      0       number of others:       0
SN      0       number of multiallelic sites:   275
SN      0       number of multiallelic SNP sites:       275

% ekidna -t -o outdir_with_tree *.fna.gz
<snip>

% ls outdir_with_tree
ekidna.aln  ekidna.fna  ekidna.full.aln  ekidna.log  ekidna.nwk  ekidna.vcf

% nw_indent outdir_with_tree/ekidna.nwk
(
  1:0.0053451412,
  (
    2:0.0000468773,
    3:0.0000132048
  )100:0.0047928014,
  (
    (
      (
        4:0.0000018784,
        6:0.0000041298
      )100:0.0000329960,
      7:0.0000872871
    )100:0.0018254115,
    5:0.0014513321
  )100:0.0036613244
);
```

## Output files

File | Contents | Format
-----|----------|--------
`.log` | log file of all the message output of the pipeline commands | ASCII text
`.vcf` | multisample VCF file of SNPs found | VCF
`.fna` | reference genome chosen from largest of input genomes | FASTA
`.aln` | FASTA alignment of core genome SNPs | FASTA (aligned)
`.full.aln` | FASTA alignment of genomes relative to the `.fna` reference | FASTA (aligned)
`.nwk` | Tree built from `.full.aln` using `iqtree` GTR+G4 model | Newick

## Dependencies

* `perl` >= 5.18
* `minimap2` + `paftools.js` >= 2.13
* `samtools` >= 1.9
* `bcftools` >= 1.9
* `any2fasta` >= 0.4
* `seqtk` >= 1.2
* `snp-sites` >= 2.0
* `bedtools` >= 2.0

## Etymology

The name Ekidna is named for the native Australian "spiny ant-eater" 
called an [echidna](https://en.wikipedia.org/wiki/Echidna). The are
coated with coarse hair and spines, and like the 
[platypus](https://en.wikipedia.org/wiki/Platypus), 
are egg-laying mammals
([monotreme](https://en.wikipedia.org/wiki/Monotreme)).
In other words, _weird_ but _cool_.

## License

Ekidna is free software, released under the
[GPL 3.0](https://raw.githubusercontent.com/tseemann/snippy/master/LICENSE).

## Issues

Please submit suggestions and bug reports to the
[Issue Tracker](https://github.com/tseemann/ekidna/issues)

## Author

[Torsten Seemann](https://twitter.com/torstenseemann)
