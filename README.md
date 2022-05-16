## RepeatDefeaters - Utilities for unclassified consensus sequences

## Table of Contents

- [**Overview**](#overview)

  - [Motivation](#motivation)
  - [Key features](#key-features)
  - [TE activity](#te-activity)

- [**Usage**](#usage)
  - [Dependancies](#dependancies)
  - [Workflow inputs](#workflow-inputs)
  - [Workflow outputs](#workflow-outputs)
  - [Customisation for Uppmax](#customisation-for-uppmax)

## Overview

### Motivation

For recently sequenced non-model organisms, repeat discovery tools
often fail to classify a large portion of their repeats. These
unclassified repeats can be host genes that were duplicated, or TEs
that are just not present in the databases. Analyses of Transposable
Elements (TE) can be misleading if host genes are present.
RepeatDefeaters is a tool to further classify which repeats should
be considered host genes or TEs.

A critical task during repeat discovery is to accurately annotate
TEs. This can be challenging for new organisms that have few
references to compare with. RepeatDefeaters aims to provide an
easy-to-follow guideline on how to tackle these consensus sequences
that are difficult to classify automatically.

### Key features

RepeatDefeaters provides:

1. Utilities that work together to determine if your consensus
   sequence of interest is related to TE activity.

### TE activity

A list of keywords which suggest TE activity has been included
in the file `assets/pfam_te_domain_keywords.txt`. Most of these
keywords are known transposon protein domains while other keywords
are plainly as "virus", "viral", and "transpos" - for the purpose
of including both "transposon" and the verb "transpose".
From these keywords, a list of Pfam sequence Ids that might
relate to TE activities has been
generated by running the `PFAM_TRANSPOSIBLE_ELEMENT_SEARCH` process.
The list can be found under `assets/Pfam_R32.Proteins_wTE_Domains.seqid`.

By setting the workflow parameter `pfam_proteins_with_te_domain_list`
in the `params` configuration block in a custom config (supplied with `-c`), the
`PFAM_TRANSPOSIBLE_ELEMENT_SEARCH` process can be skipped to save computation time.

```nextflow
params {
    pfam_proteins_with_te_domain_list = "$projectDir/assets/Pfam_R32.Proteins_wTE_Domains.seqid"
}
```

## Usage

This workflow has been designed with portability and reproducibility in
mind. The workflow is implemented using the workflow manager Nextflow
which supports a wide range of execution platforms, from local
execution, to HPC and the cloud. Software package managers are
used to bundle software dependancies to ensure programs work in the
same manner across different execution platforms.

Usage:

```bash
nextflow run -params-file params.yml  [ -c <custom.config> ] [-profile <executor profile>] GroundB/RepeatDefeaters
```

where:

- `params.yml` is a YAML formatted file containing workflow parameters
  such as input paths to the data.
  A [params.yml template](params.yml.TEMPLATE) is provided to copy
  for convenience.
  Alternatively parameters can be provided on the
  command-line using the `--parameter` notation (e.g., `--species_short_name <str>` ).
- `<custom.config>` is a nextflow configuration file which provides
  additional configuration (see the [custom.config template](custom.config.TEMPLATE)).
- `<executor profile>` is one of the preconfigured execution profiles
  (`uppmax`, `singularity_local`, `docker_local`). Alternatively,
  you can provide a custom configuration to configure this workflow
  to your execution environment. See [Nextflow Configuration](https://www.nextflow.io/docs/latest/config.html#scope-executor)
  for more details.

### Dependancies

- [Nextflow](Nextflow.io/): It can be installed
  into a custom conda environment (recommended), or directly
  into your bin. A conda environment file (nextflow_conda-env.yml) is
  provided to create a running environment with the necessary
  dependancies.
- Package Manager: One of the following (Docker or Singularity is preferred).
  - [Docker](https://www.docker.com/): A container platform.
  - [Singularity](https://sylabs.io/singularity/): A container platform,
    more commonly used on multi-user HPC environments.
  - [Conda](https://docs.conda.io/en/latest/miniconda.html): A package
    manager, which is operating system dependent.

### Workflow inputs

Mandatory:

- `repeat_modeler_fasta`: The consensus sequences from RepeatModeler2, annotated by RepeatClassifier.
- `species_short_name`: A label based on the species name, used in naming sequences and output.

Optional:

- `results`: The path to the results folder (default: `results` in the
  executing directory).
- `publish_mode`: (values: `'symlink'` (default), `'copy'`) The file
  publishing method from the intermediate results folders (see [Table of publish modes](https://www.nextflow.io/docs/latest/process.html#publishdir)).

- `protein reference`: Default is the SwissProt database (`ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz`). Additional references can be added using the params.yml file.
- `transposon_keywords`: Default path is `<workflow_dir>/assets/pfam_te_domain_keywords.txt`. It contains a list of regular expressions
  to identify proteins with TE domains. This file should not include empty lines, including newlines at the end of a file.
- `transposon_blacklist`: Default path is `<workflow_dir>/assets/te_domain_keyword_blacklist.txt`. It contains a list of regular
  expressions of protein names to exclude. This file should not include empty lines, including newlines at the end of a file.
- `pfam_proteins_with_te_domain_list`: Default is unset. When set
  this skips the `PFAM_TRANSPOSIBLE_ELEMENT_SEARCH` process, and uses
  the Pfam protein sequence ids provided in this list. A list of sequence
  ids (`Pfam_R32.Proteins_wTE_Domains.seqid`) for the default keywords are provided in the `assets` folder.
- `pfam_hmm_db`: The Pfam HMM database (default:`ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam32.0/Pfam-A.hmm.gz`).
- `pfam_hmm_dat`: The Pfam HMM dat file (default:`ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam32.0/Pfam-A.hmm.dat.gz`).
- `pfam_a_db`: The Pfam A database (default:`ftp://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam32.0/Pfam-A.full.uniprot.gz`).

Workflow package manager options:

- `enable_conda`: Enables the use of conda as the package manager (default:`false`).
- `singularity_pull_docker_container`: Construct Singularity images from
  the Docker image instead of pulling existing Singularity image (default:`false`).

Uppmax cluster options:

- `project`: SNIC Compute project allocation.

Tool specific customisation:

The tools `makeblastdb`, `blastx`, and `pfam` can have their
parameters modified by altering their module specific configuration
in your `custom.config` file.

For example, to override the parameters for `blastn` in the `TREP_BLASTN` process 
(found in`configs/modules.config`), add the following block to your custom
configuration file.

```nextflow
process {
    withName: 'TREP_BLASTN' { // Select the process name using the `withName` selector
        // tool non-file parameters are supplied using ext.args (ext.args2, ext.args3, ... 
        // - check the relevant module for which parameter to modify )
        ext.args = '-outfmt 6 -max_target_seqs 1 -evalue 1e-10'
    }
}
```

### Workflow outputs

Output folders: (subfolders in the folder provided by the `results` parameter).

- `01_Renamed_Repeat_modeler_sequences`:
  - `${short_species_name}.fasta`: Fasta file containing the renamed fasta sequences.
- `02_Pfam_TE_IDs`:
  - `Pfam.Proteins_wTE_Domains.seqid`: List of Pfam protein sequence IDs with TE activity.
- `03_Blastx`:
  - `*.{plus,minus}.blastx.tsv`: Strand-specific tab separated file of Protein sequence IDs and the sequence with a blast match to the `protein_reference`.
  - `*.{plus,minus}.predicted.fasta`: Strand-specific fasta of
    the sequences in the blast output.
- `04_Pfam_scan`:
  - `*.pfamtbl`: Pfam search of HMM domains against the blast sequences.
- `05_Reannotated_Repeat_modeler_sequences`:
  - `*.renamed.fasta`: Reannotated fasta of sequences with strand specific non-TE domains.
  - `*.Unclassified_consensus_TEs`: List of sequences with TE domains.
  - `*.consensus.both.strand`: List of sequences with non-TE domains on both strands.
- `pipeline_info`:
  - `versions.yml`: YAML file containing a list of software versions used
    in each process.
  - `execution_timeline.html`: If enabled in the `params.config`, an execution timeline of the workflow.
  - `execution_report.html`: If enabled in the `params.config`, a
    report of the resources used during the execution of the workflow.
  - `execution_trace.txt`: If enabled in the `params.config`, a trace of the workflow execution.
  - `pipeline_dag.svg`: If enabled in the `params.config`, an image
    of the workflow execution graph (which processes follow which).

### Customisation for Uppmax.

Uppmax is a set of High Performance Clusters (HPC) available to the Swedish
research community. A custom profile is available to ease use on
an Uppmax HPC. Nextflow will submit jobs to the slurm queue manager, use
the container technology Singularity to manage software dependancies,
and use the node local storage `$SNIC_TMP` for intermediate computations.

```bash
nextflow run -c <parameter.config> -profile uppmax GroundB/RepeatDefeaters
```

The command above supplies custom configuration using the `-c` option, selects the
`uppmax` configuration pipeline, and automatically downloads the workflow
from https://github.com/GroundB/RepeatDefeaters.

In order to submit to slurm, a SNIC project allocation must be provided.
This can be provided using the workflow parameter `project`.
E.g., `project = snic20xx-yy-zz` in the `params` block of the
configuration file, or `--project snic20xx-xx-zz` on the command line.

On Uppmax systems, Nextflow needs to be loaded using either the module
system:

```bash
module load bioinfo-tools Nextflow
export NXF_HOME=/proj/<snic_compute_allocation>/nextflow
```

or by activating a conda environment:

```bash
conda activate /proj/<snic_compute_allocation>/conda/nextflow-env
```

created with the command:

```bash
wget https://raw.githubusercontent.com/GroundB/RepeatDefeaters/main/nextflow_conda-env.yml
conda env create \
    --prefix "/proj/<snic_compute_allocation>/conda/nextflow-env" \
    -f nextflow_conda-env.yml
```
