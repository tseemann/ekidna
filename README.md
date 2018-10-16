[![Echidna cartoon](ekidna.png)](https://www.kisspng.com/png-hedgehog-porcupine-echidna-illustration-vector-cut-392690/)
[![Build Status](https://travis-ci.org/tseemann/ekidna.svg?branch=master)](https://travis-ci.org/tseemann/ekidna)
[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
![Don't judge me](https://img.shields.io/badge/Language-Perl_5-steelblue.svg)

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
% ekidna --version

ekidna 0.1.0
```

## Installation

### Conda
Install [Conda](https://conda.io/docs/) or [Miniconda](https://conda.io/miniconda.html):
```
conda install -c bioconda ekidna  # COMING DEC 2018
```

### Homebrew
Install [HomeBrew](http://brew.sh/) (Mac OS X) or [LinuxBrew](http://linuxbrew.sh/) (Linux).
```
brew install brewsci/bio/ekidna  # COMING DEC 2018
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

## Dependencies

* perl >= 5.26
* minimap2 + paftools.js >= 2.0
* samtools >= 1.9
* bcftools >= 1.9
* any2fasta >= 0.2
* seqtk >= 1.2
* snp-sites >= 2.0

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

