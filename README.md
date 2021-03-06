# effectR: An R package to call cytoplasmic effectors

***
[![Build Status](https://travis-ci.com/Tabima/effectR.svg?token=9dgmZbkD4jn55E4FpK6z&branch=master)](https://travis-ci.com/Tabima/effectR)
***

# What is *effectR*

The `effectR` package is an R package designed to call oomycete RxLR and CRN effectors by searching for the motifs of interest using regular expression searches and hidden markov models (HMM). 

# Overview

The `effectR` packages searches for the motifs of interest (RxLR-EER motif for RxLR effectors and LFLAK motif for CRN effectors) using a regular expression search (`REGEX`). 
These motifs used by the REGEX `effectR` search have been reported in the literature ([Haas et al., 2009](https://www.nature.com/nature/journal/v461/n7262/full/nature08358.html), [Stam et al., 2013](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0059517)).

The `effectR` package aligns the REGEX search results using [`MAFFT`](http://mafft.cbrc.jp/alignment/software/), and builds a HMM profile based on the multiple sequence alignment result using the `hmmbuild` program from [`HMMER`](http://hmmer.org/). The HMM profile is used to search across ORF of the genome of interest using the `hmmsearch` binary from `HMMER`. 
The search step will retain sequences with significant hits to the profile of interest. 
`effectR` also combines the redundant sequences found in both REGEX and HMM searches into a single dataset that can be easily exported. 
In addition, `effectR` reads and returns the HMM profile to the user and allows for the creation of a [MOTIF logo-like plot](https://en.wikipedia.org/wiki/Sequence_logo) using `ggplot2`.

## Requirements 

- R packages:
 - [`seqinr`](https://CRAN.R-project.org/package=seqinr/seqinr.pdf)
 - [`ggplot2`](http://ggplot2.org/)
 
- External software 
  - [`MAFFT`](mafft.cbrc.jp/alignment/software/)
  - [`HMMER`](http://hmmer.org/)
 
## Installation

### From CRAN

The latest version of `effectR` can be installed from CRAN. To install, make sure R is at least version 3.4.0. In the R console type

```
install.packages("effectR")
```

### From GitHub

To install `effectR` via GitHub, make sure that the `devtools` package is installed (use `install.packages("devtools")`). After installing devtools, in the R console type:

```
devtools::install_github(repo = "grunwaldlab/effectR", build_vignettes = TRUE)
library("effectR")
```

### External software (REQUIRED)

The `effectR` package uses `MAFFT` and `HMMER3` to perform the hidden markov model seach across the results from the REGEX step. These two packages should be installed before running any of the `effectR` functions.

### Downloading and installing MAFFT

MAFFT is a multiple sequence alignment program that uses Fourier-transform algorithms to align multiple sequences. We recommend downloading and installing MAFFT by following the instructions and steps in the [MAFFT installation](http://mafft.cbrc.jp/alignment/software/) web site. 

#### Linux/OS X Users

Make sure that you remember the directory in which `MAFFT` is installed, of if the installation is sucessful, make sure to obtain the path via bash/tsh/console:

```bash
which mafft
```

```
/usr/local/bin/mafft
```
For more information about MAFFT go to the MAFFT website: http://mafft.cbrc.jp/

#### Windows Users

MAFFT comes in two main distributions for windows:

- [A version that requires a UNIX-like interface called Cygwin](http://mafft.cbrc.jp/alignment/software/windows_cygwin.html)
- [An "all-in-one" version"](http://mafft.cbrc.jp/alignment/software/windows_without_cygwin.html)

Please, download and install the **all-in-one** version.
We recommend that you download and save MAFFT in your Desktop, as it will make yyour path easily accesible.

### Downloading and installing HMMER

HMMER is used for searching sequence databases for sequence homologs. It uses hidden Markov models (profile HMMs) to search for sequences with hits to similar patterns than the profile. We use three main HMMER tools: 

  - `hmmbuild` to create the HMM database from the sequences ontained in the REGEX step of `effectR`
  - `hmmpress` converts the HMM database into a format usable by other `HMMER` programs
  - `hmmsearch` to excecute the HMM search in our sequence queries basde on the HMM profile

The `effectR` package requires all of these tools. A correct `HMMER` installation will install all three programs.

#### Linux/OS X users 

We recommend downloading and installing HMMER by following the instructions and steps in the [HMMER installation](eddylab.org/software/hmmer3/3.1b2/Userguide.pdf) web site. Make sure that you remember the directory in which `HMMER` is installed, of if the installation is sucessful, make sure to obtain the path via bash/tsh/console:

```bash
which hmmbuild
which hmmpress
which hmmsearch
```

```
/usr/local/bin/hmmbuild
/usr/local/bin/hmmpress
/usr/local/bin/hmmsearch
```

For more information about HMMER go to the HMMER website: http://hmmer.org/

#### Windows users

To use the `effectR` package in Windows, the user **must** download the Windows binaries of [HMMER](http://hmmer.org/binaries/hmmer3.0_windows.zip). `effectR` will not work with any other version of HMMER.  
## Data input

The `effectR` package is designed to work with amino acid sequences in [FASTA format](https://en.wikipedia.org/wiki/FASTA_format) representing the six-frame translation of every open reading frame (ORF) of an oomycete genome. 
Using the six-frame translation of all ORF's in a genome is recommended in order to obtain as many effectors as possible from a proteome. 
To obtain the ORF for a genome, we recommend the use of EMBOSS' [`getorf`](http://emboss.sourceforge.net/apps/cvs/emboss/apps/getorf.html).

`effectR` uses a list of sequences of the class `SeqFastadna` in order to perform the effector searches. 
The function `read.fasta` from the `seqinr` package reads the FASTA amino acid file into R, creating a list of `SeqFastadna` objects that represent each of the translated ORF's from the original FASTA file. 

## REGEX search

To perform the effector search, `effectR` searches for the motifs of interest found in RxLR and CRN motifs. 
We have created the function `regex.search` to perform the seach of the motif of interest. 
The function `regex.search` requires the list of `SeqFastadna` objects and the gene family of interest. 

## Multiple sequence alignment and HMMER search

To perform the HMM search and obtain all possible effector candidates from a proteome, `effectR` uses the `REGEX` results as a template to create a HMM profile and perform a search across the proteome of interest.
We have created the `hmm.search` function in order to perfomr this search.
The `hmm.search` function requires a local installation of `MAFFT` and `HMMER` in order to perform the searches. The *absolute paths* of the binaries must be specified in the `mafft.path` and `hmmer.path` options of the `hmm.search` function.
In addition, the `hmm.function` requires the path of the original FASTA file containing the translated ORF's in the `original.seq` parameter of the function. `hmm.search` will use this file as a query in the `hmmsearch` software from HMMER, and search for all sequences with hits against the HMM profile created with the REGEX results.

The `hmm.search` object returns a list of 3 elements: 

1. The REGEX sequences used to build the HMM profile in a `SeqFastadna` class
2. The sequences from the original translated ORF files with hits to the HMM profile in a `SeqFastadna` class
3. The HMM profile table created by HMMER's `hmmbuild` as a data frame

## Obtaining non-redundant effectors and motif summaries

The user can extract all of the non-redundant sequences and a summary table with the information about the motifs using the `effector.summary` function. 
This function uses the results from either `hmm.seach` or `regex.search` functions to generate a table that includes the name of the candidate effector sequence, the number of motifs of interest (RxLR-EER or LFLAK-HVLV) per sequence and its location within the sequence. 
In addition, when the `effector.summary` function is used in an object that contains the results of `hmm.search`, the user will obtain a list of the non-reduntant sequences. If the user provides the results from `regex.search`, the function will return the motif summary table.

The motif table has a column called **MOTIF**.
This column summarizes the candidate ORF into one of 4 categories:

- Complete: The candidate ORF has both motifs of interest (RxLR + EER or LFLAK + HVLV)
- Only RxLR/Only LFLAK: The candidate ORF only has the translocation domain
- Only EER/HVLV: The candidate ORF only has the second motif of interest
- No motifs: The sequence has a hit with the HMM profile but does not have any motif of interest

### Non-redundant sequences

To export the non-redundant effector candidates that resulted from the `hmm.search` or `regex.search` functions, we use the `write.fasta` function of the `seqinr` package. 
We recomend the users to read the documentation of the [`seqinr`](https://cran.r-project.org/package=seqinr) package
Since the objects that result from the `hmm.search` or `regex.search` function are of the `SeqFastadna` class, we can use any of the function of the `seqinr` package that use this class as well. 

## Visualizing the HMM profile using a sequence logo-like plot

To determine if the HMM profile includes the motifs of interest, we have created the function `hmm.logo`.
The function `hmm.logo` reads the HMM profile (obtained from the `hmm.search` step) and uses `ggplot2` to create a bar-plot.
The bar-plot will illustrate the bits (aminoacid score) of each amino acid used to construct the HMM profile according to its consensus position in the HMM profile. 
To learn more about sequence logo plots visit this [wikipedia article](https://en.wikipedia.org/wiki/Sequence_logo).
