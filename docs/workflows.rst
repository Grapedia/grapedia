Workflows
=========

**TITAN** (**T**\ he **I**\ ntensive **T**\ ranscript **AN**\ notation pipeline)
--------------------------------------------------------------------------------

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

With the workflows/genes_annotation-workflow and docker installed, you can simply run the TITAN pipeline after data preparation (see next section).

Data preparation
^^^^^^^^^^^^^^^^

File structure to be prepared :

.. code-block:: bash

  ├── data
  │   ├── annotations
  │   ├── assemblies
  │   ├── evidencemodeler_weights_file
  │   ├── geneid_param_file
  │   ├── pasa_config_file
  │   ├── protein_data
  │       ├── for_abinitio_gene_models_selection
  │   └── RNAseq_data
  │       ├── stranded
  │       └── unstranded

In the workflows/genes_annotation-workflow folder, you can create a "data" folder containing all the data needed by TITAN to run.

**data/annotations** : contains the previous annotation in GFF3 format (eg : Vitis_vinifera_gene_annotation_on_V2_20.gff3)

**data/assemblies** : contains previous assembly (eg PN12Xv2.fasta) and new assembly (eg Chinese_ref_v2.fa)

**data/evidencemodeler_weights_file** : contains the weights.txt file for EvidenceModeler (to remove ?)

          Example :

          .. code-block:: bash

            ABINITIO_PREDICTION maker 3
            ABINITIO_PREDICTION geneid_v1.4 1
            ABINITIO_PREDICTION GlimmerHMM 1
            ABINITIO_PREDICTION AUGUSTUS 1
            ABINITIO_PREDICTION Liftoff 5
            PROTEIN exonerate 1
            TRANSCRIPT  PsiCLASS_RNAseq_stranded  10
            TRANSCRIPT  PsiCLASS_RNAseq_unstranded  8

**data/geneid_param_file** : contains the parameter file for geneid (vvinifera.param.Jan_12_2007)

**data/pasa_config_file** : contains the config file for PASA (pasa.alignAssembly.Template.txt)

          Example :

          .. code-block:: bash

            ## templated variables to be replaced exist as <__var_name__>
  
            # database settings
            DATABASE=pasa
  
            #######################################################
            # Parameters to specify to specific scripts in pipeline
            # create a key = "script_name" + ":" + "parameter"
            # assign a value as done above.
  
            #script validate_alignments_in_db.dbi
            validate_alignments_in_db.dbi:--MIN_PERCENT_ALIGNED=<__MIN_PERCENT_ALIGNED__>
            validate_alignments_in_db.dbi:--MIN_AVG_PER_ID=<__MIN_AVG_PER_ID__>
  
            #script subcluster_builder.dbi
            subcluster_builder.dbi:-m=50

**data/protein_data** : contains all the protein data files (FASTA) to perform protein alignments. Contains also a samplesheet describing the protein data file to use.

          Example :

          .. code-block:: bash
  
            organism,filename,maker_braker2
            arabidopsis,arabidopsis_prot_2022_01.fasta,no
            viridiplantae,Viridiplantae_swissprot.fasta,yes
            eudicotyledones_uniprot,eudicotyledons_uniprot.fasta,no
            eudicotyledones_orthoDB,eudicotyledons_odb10.fasta,yes
            vitales,vitales.fasta,no

**data/protein_data/for_abinitio_gene_models_selection** : contains the NR database and the uniprot database for the final process filter_evidencemodeler_gff3()

**data/RNAseq_data/{stranded,unstranded}** : contains all the RNAseq data for transcriptome assembly. Contains also the RNAseq_samplesheet. If FASTQ, the fastq file must be in the right folder, if SRA, the workflow will download the SRA file and convert it to fastq.gz file.

          Example of RNAseq_samplesheet :

          .. code-block:: bash

            sampleID,stranded_or_unstranded,SRA_or_FASTQ,paired_or_single
            ERR1059552,stranded,FASTQ,paired
            ERR1059553,stranded,FASTQ,paired
            ERR1059554,stranded,SRA,paired
            ERR1059555,stranded,SRA,paired
            SRR5435969,unstranded,FASTQ,paired
            SRR8775072,unstranded,FASTQ,paired
            SRR3046429,unstranded,SRA,paired
            SRR3046438,unstranded,SRA,paired
            SRR520373,unstranded,SRA,single

.. warning::

  In data/RNAseq_data/{stranded,unstranded}, for the FASTQ files, the name need to be ${sampleID}.fastq.gz for single-end and ${sampleID}_1.fastq.gz and ${sampleID}_2.fastq.gz for paired-end.

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
    assemblies_folder = "$projectDir/data/assemblies/"
    previous_assembly = "PN40024_40X_REF_chloro_mito.chr_renamed.fasta"
    new_assembly = "Chinese_ref_v2.fa"
    annotations_folder = "$projectDir/data/annotations/"
    previous_annotations = "PN40024_pseudomolecules.v4.3.BETA.gff3"
    RNAseq_samplesheet = "$projectDir/data/RNAseq_data/samplesheet.test.csv"
    protein_samplesheet = "$projectDir/data/protein_data/samplesheet.csv"
    geneid_param_file = "$projectDir/data/geneid_param_file/vvinifera.param.Jan_12_2007"
    pasa_config_file = "$projectDir/data/pasa_config_file/pasa.alignAssembly.Template.txt"
    evm_config_file = "$projectDir/data/evidencemodeler_weights_file/weights.txt"
    NR_proteins_fasta = "$projectDir/data/protein_data/for_abinitio_gene_models_selection/nr.fasta"
    uniprot_fasta = "$projectDir/data/protein_data/for_abinitio_gene_models_selection/uniprot_sprot.fasta"
  }

.. note::

  The $projectDir variable is the absolute path to the "workflows/genes_annotation-workflow" folder. If you have correctly followed the folders/files structure creation that is mandatory and suggested in the data preparation section, you only need to modify the file names and not the paths to these files.

Once the data has been correctly prepared and the configuration file completed, simply launch the Nextflow pipeline directly in the workflows/genes_annotation-workflow folder.

.. code-block:: bash

  nextflow run main.nf

.. note::

  To launch the pipeline, Nextflow must be installed on your computer/server following these `instructions <https://www.nextflow.io/docs/latest/install.html>`_
