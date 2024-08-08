Workflows
=========

**TITAN** (**T**\ he **I**\ ntensive **T**\ ranscript **AN**\ notation pipeline)
--------------------------------------------------------------------------------

.. image:: _static/logo_titan.jpg.png
  :width: 450
  :align: center

Installation
^^^^^^^^^^^^

First, you have to clone the pipeline repository from git:

.. code-block:: bash

  git clone https://github.com/Grapedia/workflows.git

The TITAN annotation pipeline is in the folder workflows/genes_annotation-workflow

.. note::
  Git can be installed from `Git website <https://git-scm.com/downloads>`_ 

The pipeline only requires docker to be installed. All the tools used by the pipeline are available as docker images at https://quay.io/biocontainers/ or at https://hub.docker.com/.

.. note::

  To install Docker, follow the instructions `here <https://docs.docker.com/get-docker/>`_ for Docker desktop (Mac/Windows/Linux), or if you are on Linux you can install also Docker engine, following the instructions `here <https://docs.docker.com/engine/install/>`_

Also, to launch the pipeline, Nextflow must be installed on your computer/server following these `instructions <https://www.nextflow.io/docs/latest/install.html>`_

With the workflows/genes_annotation-workflow, Nextflow and docker installed, you can simply run the TITAN pipeline after data preparation (see next section).

Data preparation
^^^^^^^^^^^^^^^^

File structure to be prepared :

.. code-block:: bash

  ├── data
  │   ├── annotations
  │   ├── assemblies
  │   ├── protein_data
  │   └── RNAseq_data

In the workflows/genes_annotation-workflow folder, you can create a "data" folder containing all the data needed by TITAN to run.

**data/annotations** : contains the previous annotation in GFF3 format (eg : Vitis_vinifera_gene_annotation_on_V2_20.gff3)

**data/assemblies** : contains previous assembly (eg PN12Xv2.fasta) and new assembly (eg Chinese_ref_v2.fa)

**data/protein_data** : contains all the protein data files (FASTA) to perform protein alignments. Contains also a samplesheet describing the protein data file to use.

          Example :

          .. code-block:: bash
  
            organism,filename,braker2
            arabidopsis,arabidopsis_prot_2022_01.fasta,no
            viridiplantae,Viridiplantae_swissprot.fasta,yes
            eudicotyledones_uniprot,eudicotyledons_uniprot.fasta,no
            eudicotyledones_orthoDB,eudicotyledons_odb10.fasta,yes
            vitales,vitales.fasta,no

**data/RNAseq_data** : contains all the RNAseq data for transcriptome assembly. Contains also the RNAseq_samplesheet. If FASTQ, the fastq file must be in the right folder, if SRA, the workflow will download the SRA file and convert it to fastq.gz file.

          Example of RNAseq_samplesheet :

          .. code-block:: bash

            sampleID,SRA_or_FASTQ,paired_or_single
            ERR1059552,FASTQ,paired
            ERR1059553,FASTQ,paired
            ERR1059554,SRA,paired
            ERR1059555,SRA,paired
            SRR5435969,FASTQ,paired
            SRR8775072,FASTQ,paired
            SRR3046429,SRA,paired
            SRR3046438,SRA,paired
            SRR520373,SRA,single

.. warning::

  In data/RNAseq_data, for the FASTQ files, the name need to be ${sampleID}.fastq.gz for single-end and ${sampleID}_1.fastq.gz and ${sampleID}_2.fastq.gz for paired-end.

Launch the pipeline
^^^^^^^^^^^^^^^^^^^

Before launching the pipeline, fill in the configuration file called “nextflow.config” in the “workflows/genes_annotation-workflow” folder.

  nextflow.config file

.. code-block:: bash

  manifest {
    author = 'Amandine Velt'
    name = 'Annotation pipeline'
    version = '1.0'
    description = 'Annotation pipeline'
  }
  
  docker {
    enabled = true
  }
  
  process {
    cpus = 20
    memory = 20.GB
  }
  
  params {
    previous_assembly = "$projectDir/data/assemblies/PN40024_40X_REF_chloro_mito.chr_renamed.fasta"
    new_assembly = "$projectDir/data/assemblies/Chinese_ref_v2.fa"
    previous_annotations = "$projectDir/data/annotations/PN40024_pseudomolecules.v4.3.BETA.gff3"
    RNAseq_samplesheet = "$projectDir/data/RNAseq_data/samplesheet.test.csv"
    protein_samplesheet = "$projectDir/data/protein_data/samplesheet.csv"
  }

.. note::

  The $projectDir variable is the absolute path to the "workflows/genes_annotation-workflow" folder. If you have correctly followed the folders/files structure creation that is mandatory and suggested in the data preparation section, you only need to modify the file names and not the paths to these files.

Once the data has been correctly prepared and the configuration file completed, simply launch the Nextflow pipeline directly in the workflows/genes_annotation-workflow folder.

.. code-block:: bash

  nextflow run main.nf
