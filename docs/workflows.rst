Workflows
=========

**TITAN** (**T**\ he **I**\ ntensive **T**\ ranscript **AN**\ notation pipeline)
--------------------------------------------------------------------------------

.. image:: _static/logo_titan.jpg.png
  :width: 450
  :align: center

A **Nextflow** and **Docker** based pipeline for full genome annotation, starting from raw RNA-seq Illumina/Iso-Seq data, reference protein data and reference genome assembly/gene annotation to annotate a new genome assembly.

Overview
^^^^^^^^^^^^

This pipeline requires only **Nextflow** and **Docker**, ensuring reproducibility and ease of use.  
It takes as input:

- Raw RNA-seq data (Illumina and optionnal PacBio Iso-Seq) in `.fastq.gz` format
- A reference protein set (`.fasta`)
- A reference genome assembly and its gene annotation (`.fasta` + `.gff3`)
- A new genome assembly to be annotated

It outputs:

- A final high-confidence gene annotation (`.gff3`) for the new genome
- A masked version of the new genome (`.fasta`)
- Functional annotations for the predicted genes (GO terms)

Pipeline Steps
^^^^^^^^^^^^

1. **Transcriptome Reconstruction**

   - `StringTie` and `PsiClass`: build transcript models from RNA-seq alignments  
   - `BRAKER3`: combines ab initio prediction with RNAseq/protein evidence

2. **Annotation Projection**

   - `Liftoff`: lifts gene annotations from the reference genome to the new assembly

3. **Automatic Structural Annotation**

   - NCBI's `EGAPX`: generates structural annotations using transcript and protein data

4. **Repeat Masking**

   - `EDTA`: identifies and soft-masks repetitive elements in the genome

5. **Annotation Merging & Filtering**

   - `AEGIS`: merges annotations (BRAKER3, EGAPX, Liftoff, transcriptomes (Stringtie/PsiClass), proteins)  
   - Applies quality filters and produces a unified `.gff3` annotation

6. **Functional Annotation**

   - `Diamond2GO`: annotates predicted proteins with Gene Ontology (GO) terms

Key Features
^^^^^^^^^^^^

- Fully portable and containerized with Docker
- Uses trusted community tools
- Accepts both short-read and long-read RNA-seq data
- Enables de novo genome annotation guided by a reference genome

Installation
^^^^^^^^^^^^

.. image:: _static/Nextflow_logo.png
    :width: 49 %
.. image:: _static/Docker_logo.png
    :width: 49 %

First, you have to clone the pipeline repository from git:

.. code-block:: bash

  git clone https://github.com/Grapedia/workflows.git

The TITAN annotation pipeline is in the folder workflows/TITAN

.. note::
  Git can be installed from `Git website <https://git-scm.com/downloads>`_ 

The pipeline only requires docker to be installed. All the tools used by the pipeline are available as docker images at https://quay.io/biocontainers/ or at https://hub.docker.com/.

.. note::

  To install Docker, follow the instructions `here <https://docs.docker.com/get-docker/>`_ for Docker desktop (Mac/Windows/Linux), or if you are on Linux you can install also Docker engine, following the instructions `here <https://docs.docker.com/engine/install/>`_

Also, to launch the pipeline, Nextflow must be installed on your computer/server following these `instructions <https://www.nextflow.io/docs/latest/install.html>`_

With the workflows/TITAN, Nextflow and docker installed, you can simply run the TITAN pipeline after data preparation (see next section).

Data preparation
^^^^^^^^^^^^^^^^

File structure to be prepared :

.. code-block:: bash

  ├── data
  │   ├── annotations
  │   ├── assemblies
  │   ├── protein_data
  │   └── RNAseq_data

In the workflows/TITAN folder, you can create a "data" folder containing all the data needed by TITAN to run.

**data/annotations** : contains the previous annotation in GFF3 format (eg : Vitis_vinifera_gene_annotation_on_V2_20.gff3)

**data/assemblies** : contains previous assembly (eg PN12Xv2.fasta) and new assembly (eg Chinese_ref_v2.fa)

**data/protein_data** : contains all the protein data files (FASTA) to perform protein alignments. Contains also a samplesheet describing the protein data file to use.

          Example :

          .. code-block:: bash
  
            organism,filename
            viridiplantae,Viridiplantae_swissprot.fasta
            eudicotyledones_orthoDB,eudicotyledons_odb10.fasta

.. warning::

  This Samplesheet is used by BRAKER3 and Aegis. For Aegis, the order of the lines is important. For example, in this example, viridiplantae will be most important as eudicotyledones_orthoDB. So order the proteins according to their importance here.

**data/RNAseq_data** : contains all the RNAseq data for transcriptome assembly. Contains also the RNAseq_samplesheet. If data is a FASTQ file, the fastq file must be in the right folder, if SRA, the workflow will download the SRA file and convert it to fastq.gz file.

          Example of RNAseq_samplesheet :

          .. code-block:: bash

            sampleID,SRA_or_FASTQ,library_layout
            ERR1059552,FASTQ,paired
            ERR1059553,FASTQ,paired
            ERR1059554,SRA,paired
            ERR1059555,SRA,paired
            SRR5435969,FASTQ,paired
            SRR8775072,FASTQ,paired
            SRR3046429,SRA,paired
            SRR3046438,SRA,paired
            SRR520373,SRA,single
            SRR17318658,SRA,long

The sampleID correspond to the SRR ID for SRA or the file ID for FASTQ. The SRA_or_FASTQ can take two possible values, "SRA" and "FASTQ". If the value is SRA, TITAN will donwload the file from public database, else the FASTQ filein .gz format must be in data/RNAseq_data. Then, the library_layout column can take three different values : "single" (if sample si single-end), "paired" (if sample is paired-end) or "long" (if sample is long reads).

.. warning::

  In data/RNAseq_data, for the FASTQ files, the name need to be ${sampleID}.fastq.gz for single-end and ${sampleID}_1.fastq.gz and ${sampleID}_2.fastq.gz for paired-end.

.. warning::

  In data/RNAseq_data, stranded short reads are mandatory, and unstranded short reads and long reads are optional. Also, if there is no library_layout as "long" in the RNAseq_samplesheet, this is not a problem. Don't forget to put the right parameters in nextflow.config : use_long_reads = false // true or false

**data/input_egapx.yaml** : The egapx parameter file, required if the egapx option is set to “yes”

          .. code-block:: bash

          genome: /mnt/project/data/assemblies/riesling.hap2.chromosomes.phased.fa
          taxid: 29760
          reads:
            - /mnt/project/data/RNAseq_data/RCDN23_S1_1.fastq.gz
            - /mnt/project/data/RNAseq_data/RCDN23_S1_2.fastq.gz
            - /mnt/project/data/RNAseq_data/KXXF10_1.fastq.gz
            - /mnt/project/data/RNAseq_data/KXXF10_2.fastq.gz
          annotation_provider: egapx_ncbi
          annotation_name_prefix: Assembly
          locus_tag_prefix: EGAPX

.. warning::

  Important: the "taxid" parameter must be that of your organism as referenced in NCBI. This allows the best reference data for your specific organism to be used automatically. Search it here : https://www.ncbi.nlm.nih.gov/taxonomy

.. warning::

  Important: The path to your “data” folder must be /mnt/project/, the mount point used. Don't worry about this, just change the name of the RNAseq fastq files in the configuration file.

Launch the pipeline
^^^^^^^^^^^^^^^^^^^

Before launching the pipeline, fill in the configuration file called “nextflow.config” in the “workflows/TITAN” folder.

  nextflow.config file

.. code-block:: bash

  // Manifest section: Defines metadata about the pipeline
  manifest {
    author = 'David Navarro (david.navarro.paya@gmail.com), Antonio Santiago (antsanpaj@gmail.com), Amandine Velt (amandine.velt@inrae.fr)'
    name = 'TITAN (The Intensive Transcript ANnotation pipeline)'
    version = '1.0'
    description = 'Gene annotation pipeline'
    homePage = 'https://github.com/Grapedia/workflows/tree/main/TITAN'
    nextflowVersion = '24.04.3'
    mainScript = 'main.nf'
  }
  
  // Docker section: Enables containerization using Docker
  docker {
    enabled = true
  }
  
  // Process settings: Defines resource allocation for processes
  process {
    // Default configuration for all other processes
    withLabel: 'default' {
      memory = '100GB'
      cpus = 10
    }
  
  }
  
  // Parameters section: Defines user-configurable parameters
  params {
    workflow = "aegis" // possible value : generate_evidence_data, aegis or all
    output_dir = "$projectDir/OUTDIR"
    previous_assembly = "$projectDir/data/assemblies/v4_genome_ref.fasta"
    new_assembly = "$projectDir/data/assemblies/riesling.hap2.chromosomes.phased.fa"
    previous_annotations = "$projectDir/data/annotations/v4_3_just_ref.gff3"
    RNAseq_samplesheet = "$projectDir/data/RNAseq_data/RNAseq_samplesheet.txt"
    protein_samplesheet = "$projectDir/data/protein_data/samplesheet.csv"
    EDTA = "yes" // Whether to run EDTA (transposable element annotation tool) - "yes" or "no"
    use_long_reads = true // Flag to indicate whether long-read sequencing data should be used (true/false)
    // PsiClass options to decrease the monoexon genes number
    PSICLASS_vd_option = 5.0 // FLOAT : the minimum average coverage depth of a transcript to be reported
    PSICLASS_c_option = 0.03 // FLOAT: only use the subexons with classifier score <= than the given number
    STAR_memory_per_job = 60000000000 // if the depth of your RNAseq samples is high, TITAN may crash with an out of memory error, using the STAR alignment tool. You can increase the memory here, it's in bytes, for example 60000000000 is about 55Gb per sample.
    egapx_paramfile="$projectDir/data/input_egapx.yaml"
  }
  }

.. note::

The $projectDir variable is the absolute path to the "workflows/TITAN folder. If you have correctly followed the folders/files structure creation that is mandatory and suggested in the data preparation section, you only need to modify the file names and not the paths to these files.

**Notes about supplementary options :**

**PSICLASS_vd_option** = 5.0 // FLOAT : the minimum average coverage depth of a transcript to be reported - to test to reduce false mono exons genes

**PSICLASS_c_option** = 0.03 // FLOAT: only use the subexons with classifier score <= than the given number - to test to reduce false mono exons genes

**STAR_memory_per_job** : For STAR alignment process. If the depth of your RNAseq samples is high, TITAN may crash with an out of memory error, during the STAR alignment step. You can increase the memory here, it's in bytes, for example 60000000000 is about 55Gb per sample/job. default: "60000000000" 

**egapx_paramfile** : a file containing the parameters for egapx NCBI pipeline.

Minimal example of **egapx_paramfile** :

.. code-block:: bash

  genome: /path/to/TITAN/data/assemblies/genome_assembly.fa
  taxid: 29760
  reads:
    - /path/to/TITAN/data/RNAseq_data/sample1_1.fastq.gz
    - /path/to/TITAN/data/RNAseq_data/sample1_2.fastq.gz
    - /path/to/TITAN/data/TITAN/data/RNAseq_data/sample2.fastq.gz
  annotation_provider: egapx_ncbi
  annotation_name_prefix: Assembly
  locus_tag_prefix: EGAPX

Once the data has been correctly prepared and the configuration file completed, simply launch the Nextflow pipeline directly in the workflows/TITAN folder. Here is an example of bash script to launch on your server.

.. code-block:: bash

  #!/usr/bin/env bash
  # Exit immediately if a command exits with a non-zero status
  # Ensures AEGIS doesn't run if generate_evidence_data fails
  set -e
  # Navigate to the project workflow directory
  cd /path/to/workflows/TITAN
  # Load required Nextflow module or use "export PATH"
  module load nextflow/24.04.3
  # Run the 'generate_evidence_data' workflow and generate its DAG
  nextflow run main.nf \
   -with-dag dag_evidence_data.png \
   --workflow generate_evidence_data -resume
  # Run the 'aegis' workflow and generate its DAG
  nextflow run main.nf \
    -with-dag dag_aegis.png \
    --workflow aegis -resume


TITAN workflow
^^^^^^^^^^^^^^^^^^^

.. image:: _static/TITAN_diagram.jpg
  :width: 1000
  :align: center


