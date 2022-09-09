---
layout: default
title:  "Processing of Multiple Sequence Alignment Data"
parent: My Projects
nav_order: 1
---
<h1><center>Processing of Multiple Sequence Alignment Data</center></h1>  

### Introduction
---------------------------------------------------------------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;While working with Dr. Martinez, a postdoctoral researcher from <a href="https://www.floridamuseum.ufl.edu/kawahara-lab/" target="_blank">Kawahara Lab</a>, a research lab investigating the evolution of butterflies and moths at the Florida Museum of Natural History, I was presented with a problem pertaining to cleaning a multiple sequence alignment (MSA) file.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This MSA file contained <a href="https://www.genome.gov/genetics-glossary/Locus" target="_blank">loci</a>, whcih are the locations of genes within a genome) important for Dr. Martinez's <a href="https://bio.libretexts.org/Bookshelves/Introductory_and_General_Biology/Book%3A_Introductory_Biology_(CK-12)/05%3A_Evolution/5.12%3A_Phylogenetic_Classification" target="_blank">phylogenetic classification</a> of the Noctuoidea (owlet moth) family. However, there were genetic information in the regions between these loci, also called flanking regions, that prevented proper phylogenetic analysis. Therefore they needed to be removed.   
![MSA_problem](/assets/img/process_MSA_problem.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In the past, this issue has been resolved by manually deleting regions between the loci. With this particular dataset, however, the number of flanking regions that needed to be cleaned was too large (385 sequences each with many loci), which proved tedious and time consuming to be done by hand.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To remove the extra data in these flanking regions, I wrote an *object oriented Python script* that iterates through every sequences in a given multiple alignment file, and edits them so that each flanking regions are like that of a given reference genome. The full script can be found on my <a href="https://github.com/nhwivo/clean-MSA-loci/blob/main/clean_MSA_loci.py" target="_blank">Github</a>.

### Procedures
---------------------------------------------------------------
#### Creating a smaller subset of the data
The original alignment contains 385 sequences, which was reduced to a smaller subset of 5 so that it was easier to work with. The reference sequence (named "BMORI_R") was also copied into a new file, which was then used to determine the coordinates of the loci's starting and ending position.


```bash
%%bash
cd ../data/
head -n 10 L0.fas > subset_MSA.fas  # make a new file with the first 5 sequences 
grep -A 1 '_BMORI_R' FcC_supermatrix_Nhi_Vo.fas >> subset_MSA.fas
grep '>' subset_MSA.fas  # check the sequence names of the newly created file 
```

    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    >_BMORI_R


#### Object Oriented Python script to parse and clean the alignment data  
An object oriented Python script was written to parse and clean the alignment data. The script was tested on the small subset of 5 sequences created above before being executed on the original file.   
In the script, a *Sequence* class was created with the appropriate attributes and methods to manipulate the data that will later be passed onto a Sequence object.  

The ***pseudocode*** was as follow:
1. Open the file that contains the MSA data.
2. Iterate through every line in the file.
    - The MSA file is similiar to a fasta file, in that each sequences takes up 2 lines, the first starts with `>` and the sequence's name, the second contains the sequence information itself. Therefore the file can be parsed into seqname and the sequence itself easily. 
3. Parse each line depending on whether it starts with '>' and pass onto a Python dictionary. 
    - The seqnames are assigned to dictionary keys with the corresponding genomic sequence as their value.
        - Example: `{"seqname1":"ACTATCTCGATCA", "seqname2":"CCTATCTCGATCA"}`
4. Process the created dictionary depending on whether it is the reference genome or the sequences to be edited. 
    - This information would be provided by the user as they input the parameters to execute the script. 
5. For the reference genome: determine the ranges (including the start and end coordinates) of the flanking regions, which will then be used as a guide to clean other sequences.
    - Ranges are determined by iterating through every character in the refseq and checking if it is a '-' character. 
        - If it is the first '-' character, then a new range just started.
        - If it is a '-' character, but not the first, then the range needs to be extended.
        - If it is not a '-' character, but the previous one was, then the range ended. 
    - Each range is recorded into a list of dictionaries with the following format: `[{'id' : 1, 'start' : 20, 'end' : 25, 'total' : 6}]`
6. For the rest of the sequences: use the ranges obtained from the reference genome to alter the flanking regions of the remaining sequences. 
    - Iterate through each sequences 
    - With each sequence, iterate through each flanking region (obtained from the reference genome) and alter the sequence based on that data. 
7. Save the processed sequences into a new file. 

#### Running the script above on the smaller subset
To make sure the script works on the smaller subset before running it on the larger original file, the script was executed using the subset as the parameter. 


```python
!../programs/clean_MSA_loci.py "../data/subset_MSA.fas" "_BMORI_R"
```

    Processing <../data/subset_MSA.fas> using <_BMORI_R> as the reference genome.
    Cleaning completed, saving result
    Resulting file named 'cleaned_loci_subset_MSA.fas' saved - program is finished.


#### Checking the resulting cleaned subset
Before the cleaning was done, the 2nd sequence in the un-processed file, *L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda*, has 2 flanking regions that contained sequence data (compared to the gaps in the reference genome BMORI). These 2 regions were later compared with the output from the section above to ensure the script worked sucessfully on this subset.


```bash
%%bash
cat ../data/subset_MSA.fas | while read line; do
    if [[ ${line:0:1} == '>' ]]; then
        echo $line
    else
        echo ${line:280:120}
    fi
done
```

    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    GTGGTAACCT-----TAACGTTACCTTCTTCACCCCTCGTGATAGATATGACC----------GAAAATACAACTTGGAGCTGATTAAA------AGGGCACTGTTCAGAGGTTTTGGAG
    
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    TCTGATGTCTACCAATCACAACACATGGATCATTACGCCAAACCATCATCACATGGTGATTGGTGAAATCCATATTACGGTACATCCAGTCTTCGAGAGCCAGGTTAGTATAAATGTCAT
    
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    GTGGCAACCT-----AAACATCACATTCTTCACACCCCGAGAAAGATATGACA----------GGAAATATAACTTGGAACTGATCAAG------AGGGCACTCTTCAGAAGCTTTGGAG
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    GTGGTAACCT-----TAATGTTACCTTCTTCACCCCTCGTGATAGATATGACA----------GAAAATACAACTTGGAGCTAATAAAA------AGGGCACTCTTCAGAGGTTTTGGAG
    >_BMORI_R
    GAGGAAATCT-----GAATATAACATTCTTTGCCCCACGTGAGAGATATGACA----------GAAATTACAATTTGAAGTTGATTAAG------AGAGCACTGTTCAGAAGTTTTGGCA


In the output file, the 2 flanking regions from *L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda* have been cleaned based on those from the reference sequence, which showed that the script worked on the small subset. 


```bash
%%bash
cat cleaned_loci_subset_MSA.fas | while read line; do
    if [[ ${line:0:1} == '>' ]]; then
        echo $line
    else
        echo ${line:280:120}
    fi
done
```

    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    GTGGTAACCT-----TAACGTTACCTTCTTCACCCCTCGTGATAGATATGACC----------GAAAATACAACTTGGAGCTGATTAAA------AGGGCACTGTTCAGAGGTTTTGGAG
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    TCTGATGTCT-----TCACAACACATGGATCATTACGCCAAACCATCATCACA----------TGAAATCCATATTACGGTACATCCAG------AGAGCCAGGTTAGTATAAATGTCAT
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    GTGGCAACCT-----AAACATCACATTCTTCACACCCCGAGAAAGATATGACA----------GGAAATATAACTTGGAACTGATCAAG------AGGGCACTCTTCAGAAGCTTTGGAG
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    GTGGTAACCT-----TAATGTTACCTTCTTCACCCCTCGTGATAGATATGACA----------GAAAATACAACTTGGAGCTAATAAAA------AGGGCACTCTTCAGAGGTTTTGGAG
    >_BMORI_R
    GAGGAAATCT-----GAATATAACATTCTTTGCCCCACGTGAGAGATATGACA----------GAAATTACAATTTGAAGTTGATTAAG------AGAGCACTGTTCAGAAGTTTTGGCA


#### Turning the script into an executable file
---------------------------------------------------------------
The script was then adjusted to be ran on the command line by adding arguments using `argparse`.    

Example usage of the script includes: 
> To get a cleaned output named 'cleaned_loci_file_name.fas' (default output name):
    `clean_MSA_loci.py file_name.fas 'BMORI'`  

> To specify name of output file:
    `clean_MSA_loci.py FcC_supermatrix.fas 'BMORI' --out desired_name.fas`

#### Running the script on the original dataset
Finally, the script was ran on the original file named *FcC_supermatrix_nv.fas*, which contained 395 aligned sequences. 


```python
!../programs/clean_MSA_loci.py "../data/FcC_supermatrix_nv.fas" "_BMORI_R"
```

    Processing <../data/FcC_supermatrix_nv.fas> using <_BMORI_R> as the reference genome.
    Cleaning completed, saving result
    Resulting file named 'cleaned_loci_FcC_supermatrix_nv.fas' saved - program is finished.


### Results
---------------------------------------------------------------
*Aliview*, an alignment viewer and editor software, was used to visually check the results. 
As seen in a snipet of the cleaned result below, flanking regions between other sequences are cleaned based on the reference genome _BMORI_R.

![MSA_result](/assets/img/process_MSA_result.png)
