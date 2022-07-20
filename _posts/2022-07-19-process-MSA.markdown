---
layout: post
title:  "Processing of Multiple Sequence Alignment Data"
date:   2022-07-19 20:16:29 -0400
categories: Kawahara Lab @ UF
---
### Introduction
---------------------------------------------------------------
While working with Joe Martinez, a postdoc at the [Kawahara lab](https://www.floridamuseum.ufl.edu/kawahara-lab/) - Musem of Natural History, I was presented with a problem pertaining to cleaning a multiple sequence alignment (MSA) file. This MSA file contained loci important to Martinez's research project, however, there were information between these loci that needed to be removed (seen in the image below). In the past, this problem has been solved by manually deleting regions between these loci. In this particular project, however, the number of regions to be cleaned was simply too large (385 sequences each with many loci), which proved tedious and time consuming to be done by hand. 

![MSA_problem](/assets/img/process_MSA_problem.png)

Below, I describe the processes I took to solve this problem, and the final executable script used to clean the MSA file.

### Creating a smaller subset of the data
---------------------------------------------------------------
The original alignment contains 385 sequences, which is reduced to a smaller subset of 5 so that it is easier to work with. The reference sequence (named "BMORI_R") is also pulled out into a separate file, which is used to determine where the different loci start and end.


```bash
%%bash
cd data/
# sequences of interest start with 'L0', such as 'L0000244_LEP81768_Noctuidae_Noctuinae_Noctua_pronuba' 
grep -A 1 'L0' FcC_supermatrix_Nhi_Vo.fas > L0.fas  # find and copy sequences of interest into new file 
grep '>' L0.fas | wc -l  # total number of whole genome sequences from the original file 
```

    201



```bash
%%bash 
cd data/
head -n 13 L0.fas > first_5.fas  # make a new file with the first 5 sequences 
grep '>' first_5.fas  # check the sequence names of the newly created file 
```

    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    >L0000244_LEP81768_Noctuidae_Noctuinae_Noctua_pronuba



```bash
%%bash 
cd data/
grep -A 1 '_BMORI_R' FcC_supermatrix_Nhi_Vo.fas > _BMORI_R.fas  # make a new file for the reference genome 
head -c 550 _BMORI_R.fas  # check the first 550 characters of the nedwly created file 
```

    >_BMORI_R
    TCAAAAAAGGAGAGAGTGCTTCCAACTGAGGGGGAGATCACCAAGTCAGTGTTCATGTCTCAATCGACTGATATTTATACGAACTTGGCTCTGGAGGATTGGTTGTATAAGAACATGGACTTCACAAATCATCATGTTATGATGGTGTGGAGAAATGAGCCATGTGTGGTTATTGGAAGACATCAAAATCCTTGGCTGGAGGCGAATGTTCCACTACTTTCTGAAAAAGAAATAGCTTTGGCACGTCGTAACAGTGGTGGTGGTACTGTTTATCATGACCGAGGAAATCT-----GAATATAACATTCTTTGCCCCACGTGAGAGATATGACA----------GAAATTACAATTTGAAGTTGATTAAG------AGAGCACTGTTCAGAAGTTTTGGCATTAAGTCAACTATTA--ATGAACGTCAGGACCTTATCGTCAGAGACAAATA-------------CAGTGATTGTTGGATCTGTATGGAATTAATGGACACATCACTAGACAAATTTTACAAATTTATATGTGAAAGAATG

### Object Oriented Python script to parse and clean the alignment data
---------------------------------------------------------------
An OOP Python script was written to parse and clean the alignment data. This script was then tested on the small subset of 6 sequences created above. 
In the Python script, a *Sequence* class was created with the appropriate attributes and methods to manipulate the data that will later be passed onto a Sequence object.  

The pseudocode is as follow:
- Open the file that contains the sequences
- Iterate through every line in the file 
    - The MSA file is similiar to a fasta file, in that each sequences takes up 2 lines, the first starts with `>` and the sequence's name, the second contains the sequence information itself. Therefore the file can be parsed easily. 
- Parse the file depending on which line it is using Python dictionary 
- For the reference genome: 
    - Determine the ranges of the flanking regions - which will then be used to clean other sequences
- For the rest of the sequences: 
    - Use the ranges above as a guide to remove information in flanking regions 
- Save the processed sequences into a new file. 



```python
class Sequence:
    def __init__(self, filename=''):
        """
        Initializes the object
        
        param mode: string, path of file to be edited
        """
        self.filename = filename  # name of file to be read 
        self.seqdict = {}  # dictionary of sequence names (keys) and sequences (values) 
        
        if filename:
            self.open_file('r')
            
    def open_file(self, mode):
        """
        Opens file contained in self.filename and sets the filehandle as self.file 
        If the file can't be opened, exit status will equal 1

        param mode: string, mode for opening file, usually 'r' or 'w'
        return: filehandle of opened file
        """
        try:
            self.file = open(self.filename, mode)
        except OSError:
            print(f'Error opening file ({self.filename})')
            # exit with status = 1
            exit(1)
            
    def next_line(self):
        """
        Returns the next line from a file
        
        return: logical, True if line is read, False when end of file 
        """
        line = self.file.readline().strip() 
        if not line: 
            # no more line, file is finished
            return False 
        
        self.line = line 
        return True  #return True when there is still line to be read 
    
    def read_file(self, seq_type, flank_ranges=''): 
        """
        Process the sequence data by determining flanking regions from the reference genome, and using those ranges to
        clean aligned sequences. 
        
        param mode: string, type of sequence, either 'reference_genome' (used to determine flanking regions)
        or 'aligned_sequences' (sequences to be edited). 
        """
        while self.next_line():  # iterate through every lines in the file
            if self.line.startswith('>'):  # create new dictonary key using sequence name/description 
                self.seqdict[self.line] = ''  # set value of dictionary key to be empty 
            else:
                last_key = list(self.seqdict.keys())[-1]  # access the last key created in the dictionary
                self.seqdict[last_key] += self.line  # append line (sequence) to the value of key 
        
        if seq_type == 'reference_genome':
            self.process_ref_genome()
        
        if seq_type == 'aligned_sequences':
            self.process_aligned_seq(flank_ranges)
    
    def process_ref_genome(self):
        """
        Create ranges of flanking regions based on the reference genome. The resulting ranges (coordinates) will then 
        determine the regions in other sequences to be cleaned. 
        """
        seq = list(self.seqdict.items())[0][1]  # access the reference genome sequence
        self.flank_range_list=[]  # list of dictionaries of flank ranges [{'id' : 1, 'start' : 20, 'end' : 25, 'total' : 6}]
        pos = 0  # keep track of position of characters in sequence string (first letter is 0)
        prev_char = False  # keep track of the previous character (true if '-', false if not '-')
        range_num = 1  # keep track of number of ranges 
        # iterate through each character in the reference sequence - keep count of character's position
        for char in seq:
            # check to see if character is '-'
            if char == '-':
                current_char = True  # set the current character = '-'
                if prev_char == False:  # if previous character isn't '-', then start a new range 
                    self.flank_range_list.append({'id' : range_num, 'start' : pos})
                    range_num+=1  

            else:  # if not '-'
                current_char = False  # set the current character as not '-'
                if prev_char == True:  # if previous character was '-', then prev position was the end 
                    temp_dict = self.flank_range_list[-1]
                    temp_dict['end'] = pos-1  # record the end of the range 
                    temp_dict['total'] = temp_dict['end'] - temp_dict['start'] + 1
                    
            pos+=1  # increase character position counter 
            prev_char = current_char  # set condition for previous character 
    
    def process_aligned_seq(self, flank_ranges):
        """
        The rest of the alignment is cleaned using the ranges/coordinates from the reference genome. 
        
        param mode: list of dictionaries of flanking regions/ranges (from the reference genome)
        """
        for key, value in self.seqdict.items():  #iterate through each name and their respective sequence
            for each_range in flank_ranges:  # iterate through each flanking region 
                start = each_range['start']
                end = each_range['end'] 
                total = each_range['total']
                add = '-'*total
                # retrieve the sequence using the name, then alter the sequence based on the coordinates given
                self.seqdict[key] = value[:start]+add+value[end+1:]
                value = self.seqdict[key]  # re-assign the new/edited sequence to the name
```

### Running the script above on the smaller subset
---------------------------------------------------------------
To make sure the script works, I tried it on the subset of 5 sequences created from before.


```python
if __name__ == '__main__':
    # PROCESS reference genome
    BMORI_path = 'data/_BMORI_R.fas'  # path to the reference genome file 
    BMORI = Sequence(BMORI_path)  # create Sequence object and open file
    BMORI.read_file('reference_genome')  # process the BMORI reference genome file 
    
    # PROCESS alignment data (including reference genome): 
    aligned_path = 'data/first_5.fas'
    aligned = Sequence(aligned_path)
    aligned.read_file('aligned_sequences', flank_ranges=BMORI.flank_range_list)
    
    # SAVE FILE: 
    with open('data/result.fas', 'w') as f:
        for key, value, in BMORI.seqdict.items():
            f.write(key)
            f.write('\n')
            f.write(value)
            f.write('\n')            
        for key, value, in aligned.seqdict.items():
            f.write(key)
            f.write('\n')
            f.write(value)
            f.write('\n')
```

##### Before sequence cleaning: 
Below, it can be seen that *L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda* has 2 flanking regions that contain sequence data. These 2 regions will be compared with the results from the cleaning done above.


```bash
%%bash
files="data/_BMORI_R.fas data/first_5.fas"
for file in $files; do
    cat $file | while read line; do
        if [[ ${line:0:1} == '>' ]]; then
            echo $line
        else
            echo ${line:280:120}
        fi
    done
done
```

    >_BMORI_R
    GAGGAAATCT-----GAATATAACATTCTTTGCCCCACGTGAGAGATATGACA----------GAAATTACAATTTGAAGTTGATTAAG------AGAGCACTGTTCAGAAGTTTTGGCA
    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    GTGGTAACCT-----TAACGTTACCTTCTTCACCCCTCGTGATAGATATGACC----------GAAAATACAACTTGGAGCTGATTAAA------AGGGCACTGTTCAGAGGTTTTGGAG
    
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    TCTGATGTCTACCAATCACAACACATGGATCATTACGCCAAACCATCATCACATGGTGATTGGTGAAATCCATATTACGGTACATCCAGTCTTCGAGAGCCAGGTTAGTATAAATGTCAT
    
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    GTGGCAACCT-----AAACATCACATTCTTCACACCCCGAGAAAGATATGACA----------GGAAATATAACTTGGAACTGATCAAG------AGGGCACTCTTCAGAAGCTTTGGAG
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    GTGGTAACCT-----TAATGTTACCTTCTTCACCCCTCGTGATAGATATGACA----------GAAAATACAACTTGGAGCTAATAAAA------AGGGCACTCTTCAGAGGTTTTGGAG
    
    >L0000244_LEP81768_Noctuidae_Noctuinae_Noctua_pronuba
    GAGGAAATCT-----CAACATCTCATTCTTCACTCCTCGTGAGAGATACGACA----------GAAAGTACAACTTGGAACTGATTAAG------AGGGCACTCTTCAGAGGATTTGGAG


##### Result of cleaning the subset:
As seen, the 2 flanking regions from *L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda* have been cleaned based on those from the reference sequence


```python
for key, value in BMORI.seqdict.items():  #print a small region from the reference genome
    print(key)
    print(value[280:400])
for key, value in aligned.seqdict.items():  # print the same region as above in the remaining 5 sequences
    print(key)
    print(value[280:400])
```

    >_BMORI_R
    GAGGAAATCT-----GAATATAACATTCTTTGCCCCACGTGAGAGATATGACA----------GAAATTACAATTTGAAGTTGATTAAG------AGAGCACTGTTCAGAAGTTTTGGCA
    >L0000176_LEP65384_Noctuidae_Pantheinae_Gaujoptera_amsa
    GTGGTAACCT-----TAACGTTACCTTCTTCACCCCTCGTGATAGATATGACC----------GAAAATACAACTTGGAGCTGATTAAA------AGGGCACTGTTCAGAGGTTTTGGAG
    >L0000444_LEP31542_Noctuidae_Lymantriinae_Spilosoma_lubricipeda
    TCTGATGTCT-----TCACAACACATGGATCATTACGCCAAACCATCATCACA----------TGAAATCCATATTACGGTACATCCAG------AGAGCCAGGTTAGTATAAATGTCAT
    >L0000158_LEP81744_Noctuidae_Bagisarinae_Bagisara_rapanda
    GTGGCAACCT-----AAACATCACATTCTTCACACCCCGAGAAAGATATGACA----------GGAAATATAACTTGGAACTGATCAAG------AGGGCACTCTTCAGAAGCTTTGGAG
    >L0000311_LEP82176_Noctuidae_Pantheinae_Millerana_austini
    GTGGTAACCT-----TAATGTTACCTTCTTCACCCCTCGTGATAGATATGACA----------GAAAATACAACTTGGAGCTAATAAAA------AGGGCACTCTTCAGAGGTTTTGGAG
    >L0000244_LEP81768_Noctuidae_Noctuinae_Noctua_pronuba
    GAGGAAATCT-----CAACATCTCATTCTTCACTCCTCGTGAGAGATACGACA----------GAAAGTACAACTTGGAACTGATTAAG------AGGGCACTCTTCAGAGGATTTGGAG


### Turning the script into an executable file
---------------------------------------------------------------
Below, I edited the script above into an executable file that takes in argument from the command line, so that it can be run outside of Jupyter Notebook. The script can be downloaded from my Github page. 


```bash
%%bash
cat programs/clean_seq.py  # printing the content of the executable script
```

    #!/usr/bin/env python
    
    import argparse
    
    parser = argparse.ArgumentParser()
    parser.add_argument("file", help="path to fasta file containing the multiple sequence alignment.")
    parser.add_argument("ref", help="path to fasta file containing the reference genome; other sequences will be trimmed based on the gap of this sequence.")
    parser.add_argument("--out", help="file output.")
    args = parser.parse_args()
    
    
    class Sequence:
        def __init__(self, filename=''):
            """
            Initializes the object
            
            param mode: string, path of file to be edited
            """
            self.filename = filename  # name of file to be read 
            self.seqdict = {}  # dictionary of sequence names (keys) and sequences (values) 
            
            if filename:
                self.open_file('r')
                
        def open_file(self, mode):
            """
            Opens file contained in self.filename and sets the filehandle as self.file 
            If the file can't be opened, exit status will equal 1
    
            param mode: string, mode for opening file, usually 'r' or 'w'
            return: filehandle of opened file
            """
            try:
                self.file = open(self.filename, mode)
            except OSError:
                print(f'Error opening file ({self.filename})')
                # exit with status = 1
                exit(1)
                
        def next_line(self):
            """
            Returns the next line from the file
            
            return: logical, True if line is read, False when end of file 
            """
            line = self.file.readline().strip() 
            if not line: 
                # no more line, file is finished
                return False 
            
            self.line = line 
            return True  #return True when there is still line to be read 
        
        def read_file(self, seq_type, flank_ranges=''): 
            """
            Process the sequence data by determining flanking regions from the reference genome, and using those ranges to
            clean aligned sequences. 
            
            param mode: string, type of sequence, either 'reference_genome' (used to determine flanking regions)
            or 'aligned_sequences' (sequences to be edited). 
            """
            while self.next_line():  # iterate through every lines in the file
                if self.line.startswith('>'):  # create new dictonary key using sequence name/description 
                    self.seqdict[self.line] = ''  # set value of dictionary key to be empty 
                else:
                    last_key = list(self.seqdict.keys())[-1]  # access the last key created in the dictionary
                    self.seqdict[last_key] += self.line  # append line (sequence) to the value of key 
            
            if seq_type == 'reference_genome':
                self.process_ref_genome()
            
            if seq_type == 'aligned_sequences':
                self.process_aligned_seq(flank_ranges)
        
        def process_ref_genome(self):
            """
            Create ranges of flanking regions based on the reference genome. The resulting ranges (coordinates) will then 
            determine the regions in other sequences to be cleaned. 
            """
            seq = list(self.seqdict.items())[0][1]  # access the reference genome sequence
            self.flank_range_list=[]  # list of dictionaries of flank ranges [{'id' : 1, 'start' : 20, 'end' : 25, 'total' : 6}]
            pos = 0  # keep track of position of characters in sequence string (first letter is 0)
            prev_char = False  # keep track of the previous character (true if '-', false if not '-')
            range_num = 1  # keep track of number of ranges 
            # iterate through each character in the reference sequence - keep count of character's position
            for char in seq:
                # check to see if character is '-'
                if char == '-':
                    current_char = True  # set the current character = '-'
                    if prev_char == False:  # if previous character isn't '-', then start a new range 
                        self.flank_range_list.append({'id' : range_num, 'start' : pos})
                        range_num+=1  
    
                else:  # if not '-'
                    current_char = False  # set the current character as not '-'
                    if prev_char == True:  # if previous character was '-', then prev position was the end 
                        temp_dict = self.flank_range_list[-1]
                        temp_dict['end'] = pos-1  # record the end of the range 
                        temp_dict['total'] = temp_dict['end'] - temp_dict['start'] + 1
                        
                pos+=1  # increase character position counter 
                prev_char = current_char  # set condition for previous character 
        
        def process_aligned_seq(self, flank_ranges):
            """
            The rest of the alignment is cleaned using the ranges/coordinates from the reference genome. 
            
            param mode: list of dictionaries of flanking regions/ranges (from the reference genome)
            """
            for key, value in self.seqdict.items():
                for each_range in flank_ranges:
                    start = each_range['start']
                    end = each_range['end'] 
                    total = each_range['total']
                    add = '-'*total
                    self.seqdict[key] = value[:start]+add+value[end+1:]
                    value = self.seqdict[key]
                    
    if __name__ == '__main__':
        print("Processing <" + args.file + "> using <" + args.ref + "> as the reference.")
        # PROCESS reference genome
        ref_path = args.ref  # path to the reference genome file 
        ref = Sequence(ref_path)  # create Sequence object and open file
        ref.read_file('reference_genome')  # process the reference genome file 
        
        # PROCESS alignment data: 
        aligned_path = args.file
        aligned = Sequence(aligned_path)
        aligned.read_file('aligned_sequences', flank_ranges=ref.flank_range_list)
        
        # SAVE FILE: 
        out = 'cleaned_result.fas'  # name for output file 
        if args.out:  # user specified output file name 
            out=args.out
            
        print("Cleaning completed, saving results")
        with open(out, 'w') as f:         
            for key, value, in aligned.seqdict.items():
                f.write(key)
                f.write('\n')
                f.write(value)
                f.write('\n')
                
        print("Results saved - program finished")

### Making sure the executable file works
---------------------------------------------------------------
In the first cell below, "!" prefix is used to indicate command line to Jupyter Notebook. The inputs are the same files used before. The resulting file, "cleaned_result.fas" is the same as the cleaned result from above, showing that the edited script for the command line works.


```python
!programs/clean_seq.py "data/first_5.fas" "data/_BMORI_R.fas" 
```

    Processing <data/first_5.fas> using <data/_BMORI_R.fas> as the reference.
    Cleaning completed, saving results
    Results saved - program finished



```bash
%%bash
cat cleaned_result.fas | while read line; do
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
    >L0000244_LEP81768_Noctuidae_Noctuinae_Noctua_pronuba
    GAGGAAATCT-----CAACATCTCATTCTTCACTCCTCGTGAGAGATACGACA----------GAAAGTACAACTTGGAACTGATTAAG------AGGGCACTCTTCAGAGGATTTGGAG


### Result from cleaning the original dataset of 395 sequences
---------------------------------------------------------------
As seen in a snipet of the cleaned result below, flanking regions between other sequences are cleaned based on the reference genome _BMORI_R.

![MSA_result](/assets/img/process_MSA_result.png)


```python

```
