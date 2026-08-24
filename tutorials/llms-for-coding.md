# Exercise: Using LLMs to Write a Data-Parsing Script

### Background

This exercise uses real proteomics data from a published TAILS (N-terminomics) study
comparing healthy and ulcerative colitis colon biopsies (Gordon et al., *ACS Chem. Biol.* 2019,
DOI: [10.1021/acschembio.9b00608](https://doi.org/10.1021/acschembio.9b00608)).
Raw and processed data are deposited on PRIDE under accession
[PXD014479](https://www.ebi.ac.uk/pride/archive/projects/PXD014479).

You will work with:

- **Supplementary Table 1** ([`SupplementaryTables1-6_NEW.xlsx`](./SupplementaryTables1-6_NEW.xlsx),
  sheet "Supplementary Table 1"): identified peptides with their matching UniProt accessions,
  searched without enzyme specificity constraints; already extracted to
  [peptides.txt](tutorials/llms-for-coding-data/peptides.txt)
- The human proteome FASTA file used for the original search:
  [HUMAN_FASTA_uniprot-proteome3AUP000005640.fasta](https://ftp.pride.ebi.ac.uk/pride/data/archive/2019/09/PXD014479/HUMAN_FASTA_uniprot-proteome3AUP000005640.fasta)

### The coding task

Each peptide in the table needs to be located within its parent protein sequence, so that the
flanking amino acids around it can be extracted. This kind of task, matching one dataset against
a reference sequence and pulling out context around a match, is common in bioinformatics and a
good test case for working with an LLM as a coding assistant.

### What the LLM needs to do

1. Read the peptide and accession columns from the Excel file.
2. Parse the FASTA file into an accession-to-sequence lookup.
3. For each peptide, locate its position within the corresponding protein sequence(s).
4. Extract a defined window of residues upstream and downstream of the match.
5. Handle the edge cases that real data always contains: multiple accessions per peptide,
   contaminant/decoy entries, peptides with no match, and peptides too close to a protein's
   start or end for a full window.
6. Write the results to a new table.

### Goal of the exercise

You will write prompts of increasing detail and observe how the script's correctness changes. A vague prompt produces a script that runs but may get edge cases wrong, whereas a more precise prompt, one that names the exact steps and potential failure modes, can produce a working script on the first attempt. The point is to build a feel for how much specification an LLM needs from you and where you can rely on it to make reasonable default decisions. You will also notice how an iterative planning conversation, held before any code is written, helps you and the LLM collaboratively build up a detailed plan that the LLM then carries into implementation.
