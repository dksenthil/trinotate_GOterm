

## Steps to setup custom reference genome for transcript annotation via Genpipes

Genpipes is a repository holds several bioinformatics pipelines developed at McGill University and Génome Québec Innovation Centre (MUGQIC), as part of the GenAP project.

https://bitbucket.org/mugqic/genpipes/src/ma

Trinotate makes use of a number of different well referenced methods for functional annotation including homology search to known sequence data (BLAST+/SwissProt), protein domain identification (HMMER/PFAM), protein signal peptide and transmembrane domain prediction (signalP/tmHMM), and leveraging various annotation databases (eggNOG/GO/Kegg databases) <https://github.com/Trinotate/Trinotate.github.io/wiki>.


Note: the following guidelines assumes that you have run rna_denovo_assembly pipelines in default mode and have the following folders  and files.

### 1. Create a working folder like say "trinotate_Bacillus_cereus_GO" and symlink Trinity.fasta inside

```{r foldersetup, eval=F}

> mkdir -p trinotate_Bacillus_cereus_GO/trinity_out_dir
> cd trinotate_Bacillus_cereus_GO/
> ln -s ../raw_reads .
> ln -s ../trinity_out_dir/Trinity.fasta trinity_out_dir/Trinity.fasta 
> cp design_table.tsv readset-gen.tsv .

#Final folder structure of the CWD should look like this

├── 01_run_trinotate_genpipes.sh
├── design_table.tsv
├── GCF_018309165.1_ASM1830916v1_protein.faa
├── GCF_018309165.1_ASM1830916v1_protein.faa.pdb
├── GCF_018309165.1_ASM1830916v1_protein.faa.phr
├── GCF_018309165.1_ASM1830916v1_protein.faa.pin
├── GCF_018309165.1_ASM1830916v1_protein.faa.pot
├── GCF_018309165.1_ASM1830916v1_protein.faa.psq
├── GCF_018309165.1_ASM1830916v1_protein.faa.ptf
├── GCF_018309165.1_ASM1830916v1_protein.faa.pto
├── run_gffread_to_get_transcript.sh
├── run_makeblastdb.sh
├── trinity_out_dir
    ├── Bacillus_cereus.ASM83007v1.dna.transcript.fa
    ├── Trinity.fasta
    ├── Trinity.fasta.gene_trans_map

```

### 2. Create transcript sequence from genome reference and using GFF coordinates

```{r gffread, eval=F}
#cat run_gffread_to_get_transcript.sh
module load mugqic/cufflinks/2.2.1

gffread -w GCF_018309165.1_ASM1830916v1_transcript.fa  -g GCF_018309165.1_ASM1830916v1_genomic.fna GCF_018309165.1_ASM1830916v1_genomic.gff
```


### 3. Make blastDB for the reference genome transcript

```{r makeblastDB, eval=FALSE}

#cat run_makeblastdb.sh


#BLAST version should match genpipes version used in the Genpipes RNA_assembly pipeline
module load mugqic/perl/5.22.1 mugqic/mugqic_tools/2.2.2 mugqic/blast/2.3.0+
  
# makeblastdb -dbtype nucl -in  GCF_018309165.1_ASM1830916v1_transcript.fna
 makeblastdb -dbtype prot -in  GCF_018309165.1_ASM1830916v1_protein.faa
```

### 4. Create a custom INI file with blast reference data path

```{r makeINI, eval=FALSE}
#cat my_custom.ini

[DEFAULT]
swissprot_db=/lb/project/genomes/trinotate_GO_bacteria/GCF_018309165.1_ASM1830916v1_protein.faa

```


### 5. Run genpipes forcing the steps7-16 needed for obtaining GO/KEGG terms

```{r run, eval=F}
mv 02_rnaseq_genpipesscripts.sh 02_rnaseq_genpipesscripts.sh.bk

module purge
module load mugqic/python/2.7.14
module load mugqic/genpipes/3.6.1

#Beluga


#rnaseq.py -c $MUGQIC_PIPELINES_HOME/pipelines/rnaseq/rnaseq.base.ini $MUGQIC_PIPELINES_HOME/pipelines/rnaseq/rnaseq.beluga.ini custom.ini -r readset_petunia.tsv  -s 1-18 > rnaseqDeNovoCommands.sh
# Abacus

rnaseq_denovo_assembly.py  -f -s 7-16  --no-json  -c $MUGQIC_PIPELINES_HOME/pipelines/rnaseq_denovo_assembly/rnaseq_denovo_assembly.base.ini   my_custom.ini   -j pbs   -d design_table.tsv  -r readset-gen.tsv      > 02_rnaseq_genpipesscripts.sh
```

### 6. Extract GO terms using Trinotate scripts.

[After running the generated genpipes scripts successfully, you will have a *trinotate* folder with **trinotate_annotation_report.tsv** file as output. you can now extract GO terms from this file]


```{r extractGO, eval=F}

module purge
module load mugqic/perl/5.22.1 mugqic/trinity/2.0.4_patch mugqic/trinotate/2.0.2
${TRINOTATE_HOME}/util/extract_GO_assignments_from_Trinotate_xls.pl  \
    --Trinotate_xls trinotate_annotation_report.tsv \
      -G --include_ancestral_terms \
        > go_annotations.txt
```




