---
layout: default
title:  "Genomic Ranges Assembly Tool"
parent: My Projects
nav_order: 7
---

<h1><center>Genomic Ranges Assembly Tool</center></h1>  

### Introduction
--------------------------------------------------------------------------------
This program reads in either a GFF or a BlastX result file and:
1. Sorting:
    - Sorts the lines by chromosome/scaffold in alphabetical order
    - Sorts the start position in ascending order
2. Find sequences that overlap and add them to a range
3. Write each resulting range into a .txt file with the following format:
    - sequence id, begin position, end position, semicolon delimited list of the overlapping transcript IDs
    - Only the first 3 transcript IDs will be shown, the rest will be reported as a number  
    
Then, with the ranges from BlastX and GFF, the program:
1. Combine the ranges
2. Sort the ranges
3. Write each resulting range into a .txt file with the following format:
    - scaffold ID, start, end, number of sequences in blast, #sequences in gff
    - comma delimited list of blast sequences
    - comma delimited list of gff sequences


### Example Inputs and Output
--------------------------------------------------------------------------------
#### Example GFF file input: 


```bash
%%bash
head Hp.augustus.hints.txt
```

    scaffold8	AUGUSTUS	gene	1541414	1541665	.	+	.	ID=jg10489
    scaffold8	AUGUSTUS	transcript	1541414	1541665	.	+	.	ID=jg10489.t1;Parent=jg10489
    scaffold8	AUGUSTUS	start_codon	1541414	1541416	.	+	0	ID=jg10489.t1.start;Parent=jg10489.t1
    scaffold8	AUGUSTUS	CDS	1541414	1541665	0.44	+	0	ID=jg10489.t1.e0;Parent=jg10489.t1
    scaffold8	AUGUSTUS	stop_codon	1541663	1541665	.	+	0	ID=jg10489.t1.stop;Parent=jg10489.t1
    scaffold8	AUGUSTUS	gene	728710	733713	.	+	.	ID=jg10261
    scaffold8	AUGUSTUS	transcript	728710	733713	.	+	.	ID=jg10261.t1;Parent=jg10261
    scaffold8	AUGUSTUS	start_codon	728710	728712	.	+	0	ID=jg10261.t1.start;Parent=jg10261.t1
    scaffold8	AUGUSTUS	CDS	728710	733713	0.56	+	0	ID=jg10261.t1.e0;Parent=jg10261.t1
    scaffold8	AUGUSTUS	stop_codon	733711	733713	.	+	0	ID=jg10261.t1.stop;Parent=jg10261.t1


#### Output from GFF input above: 


```bash
%%bash
head gff.ranges.test.txt
```

    range scaffold10 142 273 jg9994.t1
    range scaffold10 631 2378 jg9995.t1;jg9995.t2
    range scaffold10 2523 2988 jg9996.t1
    range scaffold10 3665 4487 jg9997.t1;jg9997.t2
    range scaffold10 4686 6440 jg9998.t1
    range scaffold10 6816 7274 jg9999.t1;jg9999.t2
    range scaffold10 7424 7740 jg10000.t1
    range scaffold10 8456 8795 jg10001.t1
    range scaffold10 8881 9329 jg10002.t1
    range scaffold10 9378 13875 jg10003.t1;jg10003.t2;jg10003.t3 + 1 more


#### Example BlastX file input: 


```bash
%%bash 
head genome.small_uniref50.txt
```

    scaffold8_size1621971	UniRef50_W4JNQ2	68.4	1791	375	25	337976	332703	1	1633	0.0e+00	2137.1
    scaffold8_size1621971	UniRef50_W4JMZ0	82.1	1450	185	10	1020664	1025001	433	1811	0.0e+00	2034.2
    scaffold8_size1621971	UniRef50_A0A4S4M7F6	55.7	2034	692	32	338663	332703	423	2293	0.0e+00	1925.6
    scaffold8_size1621971	UniRef50_A0A4Y9Z8Y5	49.6	1771	729	32	564374	569497	395	2065	0.0e+00	1464.9
    scaffold8_size1621971	UniRef50_A0A409WYT1	50.1	1633	660	32	369986	374608	9	1578	0.0e+00	1419.1
    scaffold8_size1621971	UniRef50_A0A0C3BKH6	46.7	1806	804	31	564374	569497	264	2008	0.0e+00	1349.0
    scaffold8_size1621971	UniRef50_A0A4S4LNA1	50.8	1501	504	18	298041	293548	1	1270	0.0e+00	1342.8
    scaffold8_size1621971	UniRef50_W4JN72	92.9	733	52	0	1109687	1107489	83	815	0.0e+00	1333.9
    scaffold8_size1621971	UniRef50_A0A4Y9YUT6	47.2	1479	611	18	397939	393890	1844	3281	0.0e+00	1313.1
    scaffold8_size1621971	UniRef50_W4JN63	76.2	870	89	5	1309991	1312600	1	752	0.0e+00	1288.1


#### Output from BlastX input above: 


```bash
%%bash
head blast.ranges.test.txt
```

    range scaffold10 4582 6201 W4KI18;W4KIR5;W4KIT9 + 1 more
    range scaffold10 7086 9314 A0A0C9YXZ4;A0A0W0FFE0;A0A4S8M2J1 + 4 more
    range scaffold10 11858 12928 W4K6X1;W4K9R5;W4KAC0 + 6 more
    range scaffold10 12972 13787 W4KIW0
    range scaffold10 16224 18701 A0A1J8QBT6;A0A5K1K0T9;S8ELC6 + 2 more
    range scaffold10 26638 30693 A0A015J7Q1;A0A015L1E1;A0A016RSS9 + 3510 more
    range scaffold10 35737 37431 W4JVS3;W4JYJ1
    range scaffold10 37871 42043 A0A014P2S2;A0A015J4G7;A0A016RZP3 + 2086 more
    range scaffold10 42142 43299 W4JNB3;W4JWB1;W4K5K6
    range scaffold10 47258 51313 A0A014MV30;A0A016W340;A0A016W703 + 1015 more


#### Example output from combining BlastX and GFF ranges: 


```bash
%%bash
head -n30 combined.ranges.txt | tail -n 19
```

    scaffold10	6816	9329	7	5
    	A0A0C9YXZ4, A0A0W0FFE0, A0A4S8M2J1, A0A4T0FVX5, A0A5B0MP72, A0A5N5QU40, E3KRC1
    	jg10000.t1, jg10001.t1, jg10002.t1, jg9999.t1, jg9999.t2
    scaffold10	9378	13871	10	3
    	W4K6X1, W4K9R5, W4KAC0, W4KC44, W4KC85, W4KCA6, W4KGU8, W4KHL0, W4KIW0, W4KJN0
    	jg10003.t1, jg10003.t2, jg10003.t3
    scaffold10	15417	15932	0	1
    	jg10004.t1
    scaffold10	16206	18701	5	1
    	A0A1J8QBT6, A0A5K1K0T9, S8ELC6, W4JVV5, W4KHN3
    	jg10005.t1
    scaffold10	26638	30693	3513	1
    	A0A015J7Q1, A0A015L1E1, A0A016RSS9, A0A016RWX8, A0A016SWR3, A0A016T554, A0A016TD83, A0A016TPF9,
    	A0A016TQU4, A0A016TSV2, A0A016U4X2, A0A016U7X8, A0A016UCZ9, A0A016UF53, A0A016UQN2, A0A016URH6,
    	A0A016UTY1, A0A016UVT6, A0A016V3M6, A0A016V6L6, A0A016WKG0, A0A016WX09, A0A016WX99, A0A023AVB1,
    	A0A023AWJ7, A0A023AXI1, A0A023EY26, A0A023EY78, A0A023EYN5, A0A034VAM6, A0A034VKV9, A0A034WRX5,
    	A0A060S8N1, A0A060SB49, A0A060SCW0, A0A060SL03, A0A060SN68, A0A060T683, A0A060VW34, A0A060YYM2,
    	A0A061DMW6, A0A061DRY4, A0A061E6T4, A0A061EVN4, A0A061FWM3, A0A061FWU4, A0A068SA08, A0A068SFI6,
    	A0A068SGZ4, A0A068SH41, A0A068SHF5, A0A068SII5, A0A068SIM2, A0A068WYH4, A0A068WYN8, A0A072UJW9,



### Genomic Ranges Assembly Codes
--------------------------------------------------------------------------------
Below are the codes for the tool.  
Note: this script has not been edited to be an executable file that takes in user arguments, and cannot be ran from the command line.


```python
class GenomeRange:

    def __init__(self, filename=''):
        """
        Genome ranges are linear regions of a sequence from the begin to end position
        (inclusive)
        feature keys:
            sequence: ID of sequence, usually a chromosom or scaffold
            begin: starting position of the sequence, 1 based
            end: ending position of the feature, 1 based, inclusive
            name: name of the feature
        """
        # attributes for the read_gff; readBlast functions:
        self.filename = filename  # name of file
        self.file = None  # filehandle
        self.features = []  # list of selected features

        # utility attributes
        self.line = ''  # last line read from input file

        if filename:
            self.open_file('r')

    def open_file(self, mode):
        """
        Opens the file in self.filename with error checking and sets the filehandle, self.file
        Exit with status=1 if the file can't be opened
        param mode: string, mode for opening file, usually 'r' or 'w'
        return: filehandle of opened file
        """
        try:
            self.file = open(self.filename, mode)
        except OSError:
            print(f'Error opening file ({self.filename})')
            # since you cannot continue, exit with status = 1
            exit(1)

    def next_line(self):
        """
        Returns the next line/row from either GFF of BlastX file, skipping blank lines and comments
            - comments begin with #
        return: logical, True if line is read, False when EOF is reached
        """
        line = self.file.readline().strip()
        while line and line.isspace() or line.startswith('#'):
            # skip empty lines (\n at the bottom) and comments
            line = self.file.readline().strip()

        if not line:
            # file is finished
            return False

        self.line = line
        return True

    def read_GFF(self, select=['transcript']):
        """
        Populate the features with a list dictionaries from a GFF file. The dictionary has the keys
        sequence, name, begin, end, where sequence is the ID of the genomic sequences and name is
        the feature ID
        Rows are selected if the feature (column 3)  is in the parameter select.
        example GFF format:
        scaffold9   AUGUSTUS    gene        135854  136842  .   +   .   ID=jg677
        scaffold9   AUGUSTUS    transcript  135854  136842  .   +   .   ID=jg677.t1;Parent=jg677
        scaffold9   AUGUSTUS    start_codon 135854  135856  .   +   0   ID=jg677.t1.start;Parent=jg677.t1
        scaffold9   AUGUSTUS    CDS         135854  135929  1   +   0   ID=jg677.t1.e0;Parent=jg677.t1
        sequence    source      feature     begin   end  score strand frame attributes
        param select: list of strings corresponding to GFF fields
        return: int, number of features read
        """
        n_feature = 0
        while self.next_line():
            field = self.line.split('\t')
            if len(field) < 9:
                continue

            # all features
            gff = {'sequence': field[0], 'source': field[1], 'feature': field[2],
                   'begin': int(field[3]), 'end': int(field[4]), 'strand': field[6],
                   'frame': field[7], 'attr': field[8]}

            # the attributes are a semicolon delimited list of tag value pairs; the feature ID is in the attributes
            attrs = gff['attr'].split(';')
            attribute = {}
            for a in attrs:
                tag, value = a.split('=')
                attribute[tag] = value

            if gff['feature'] in select:
                self.features.append({'sequence': gff['sequence'], 'name': attribute['ID'],
                                      'begin': gff['begin'], 'end': gff['end']})
                n_feature += 1

        return n_feature

    def read_blast(self, e_threshold=1e-50, select=[0, 1, 6, 7]):
        """
        Returns a list of features from a BlastX results, such as:
            - scaffold number, start position, end position, subject/geneID
        """
        self.features = []  # remove any existing features

        while self.next_line():
            if not self.line:
                # end of file
                break
            if self.line.isspace():
                # blank line
                continue

            field = self.line.rstrip().split('\t')
            begin = int(field[6])
            end = int(field[7])
            if begin > end:
                # make sure begin is less than end
                begin, end = end, begin

            subject = field[1].replace('UniRef50_', '')
            ul_pos = field[0].index('_')
            query = field[0][:ul_pos]

            if float(field[10]) <= e_threshold:
                self.features.append({'sequence': query, 'begin': begin, 'end': end, 'name': subject})

        return len(self.features)

    def sort_by_pos(self):
        """
        Sorts a list of features first by scaffold number, then by start position
        in ascending order (small to large)
        """
        self.features = sorted(self.features, key=lambda c: (c['sequence'], c['begin']), reverse=False)

    def find_range(self):
        """
        Determine if the coordinates of a row overlap with the current range.
        If yes (overlap): add features onto the current range
        If no (no overlap): create new range
        """

        ################################################################################################################
        def find_overlaps(range, feature):
            """
            Note: this is currently a local function inside find_range. It would be better written as a static method
            find overlaps works on the dictionaries of two features, the first is the overlap range being
            created and the second is the feature that may of may not overlaps. Since the features are
            pre-sorted, the only possible overlap is between range['end'] and feature['begin']
            params range, feature: dict, keys=['sequence', 'name', 'begin', 'end']
            Returns Logical:, True if there is no overlap, False if is an overlap
            """

            # special cases: for the first range, range==None, for new sequences the sequences are different
            if not range or range['sequence'] != feature['sequence']:
                return False

            if feature['begin'] - 1 <= range['end']:
                return True

            else:
                return False

            # end of find_overlaps local function

        ################################################################################################################

        # create a genome range to store the overlapped features, and a pointer to the current range
        grange = GenomeRange()
        current = None

        for feature in self.features:
            # for every selected feature, compare to the current grange feature

            if find_overlaps(current, feature):
                # there is an overlap
                # add previous range into the range list before overwriting range object:
                current['end'] = max(current['end'], feature['end'])
                current['name'] += f';{feature["name"]}'

            else:
                # no overlap
                # start a new range based on the current feature
                grange.features.append({'sequence': feature['sequence'], 'name': feature['name'],
                                        'begin': feature['begin'], 'end': feature['end']})

                current = grange.features[-1]

        return grange

    def write_file(self, max_names=3):
        """
        Writes a list into a .txt file (space delimited), lists of names are sorted alphabetically and the list
        truncated at max_names. self.filename is used for the file
        max_names: int, maximum numbers of names to list, the number not shown is written
        return: int, number of features written
        """
        self.open_file('w')
        namelist = ''
        for f in self.features:
            names = f['name'].split(';')
            if len(names) > 3:
                more = len(names) - 3
                namelist = ';'.join(sorted(names)[:3]) + f' + {more} more'
            else:
                namelist = ';'.join(sorted(names))

            self.file.write(f"range {f['sequence']} {f['begin']} {f['end']} {namelist}\n")

        self.file.close()
        return len(self.features)

    def combine_ranges(self, ranges2combine=[]):
        """
        This function combines 2 .features attributes from the previous objects (gff and blast ranges)
        For each ranges object, a number is added to the 'name' key in features dictionary
            - Reason: so that the names can be separated later based on where they came from
        parameters:
            - ranges2combine: list of lists of dictionaries
                - the lists of dictionaries are from .features of other ranges object
        output:
            - does not return any value
            - assign values to self.features: list of dictionaries
        """
        # add numbers to separate the file types later (blast/gff):
        file_type = 0
        for dictlist in ranges2combine:
            for features_dict in dictlist:
                features_dict['name'] += ('_' + str(file_type))
            file_type += 1

        # combine all dictionaries in the different lists:
        # (make it list of dictionaries instead of list of lists of dictionaries)
        self.features = [features for dictlist in ranges2combine for features in dictlist]

    def write_combined_file(self):
        """
        This function writes the list of features into a txt. file with the following format:
            scaffold# start end #blast_names #gff_names (tab delimited)
            all the blast names (comma delimited)
            all the gff names (comma delimited)
        output:
            - output .txt file described above
        """
        self.open_file('w')
        for f in self.features:  # for each dictionary in the list
            file_type = 0
            counter = [0, 0]  # keep track of how many IDs
            name_dict = {0: [], 1: []}
            for name in (f['name'].split(';')):  # for each ID in the 'name' key
                value = name.split('_')  # ex: value = ['jg9994.t1','0']
                if int(value[1]) == 0:  # the same group
                    counter[0] += 1  # add to counter
                    name_dict[0].append(value[0])  # add to list of names
                else:  # different group
                    counter[1] += 1
                    name_dict[1].append(value[0])
            # writing the first line of the range
            self.file.write(f"{f['sequence']}\t{f['begin']}\t{f['end']}\t{counter[0]}\t{counter[1]}\n")
            # writing the names in the range
            for key in name_dict:
                if name_dict[key]:  # only work with values that are not empty
                    name_dict[key] = self.name_formatting(sorted(name_dict[key]))  # format the names
                    self.file.write(f"\t{', '.join(name_dict[key])}\n")  # write the names into the file

        self.file.close()

    # static function, but I wanted to include it with the rest HW7 functions:
    def name_formatting(self, name_list, threshold=72):
        """
        This function formats the names to be written into the .txt file. How long each line to be
        writen to the file can be adjusted.
        parameter:
            - name_dict: dictionary where the keys separate GFF and blast, and the values are
            lists of names
        output:
            - formatted list: list of names that will result in proper formatting when written
            into a file.
        """
        formatted_list = []  # list of names (in all the lines)
        length_list = []  # list of names (for each line)
        for each_name in name_list:  # iterate through each name in the list of names
            if sum([len(name) for name in length_list]) <= threshold:
                length_list.append(each_name)  # add name to a small list if threshold hasn't been reached
            else:
                # if the length is longer than the threshold:
                formatted_list.append(length_list)  # add the previous list into a bigger list
                length_list = ['\n\t' + each_name]  # write over the small list
                # (summary: this is like creating a new range when there is no overlap)
        formatted_list.append(length_list)  # add the last small list into the big list
        formatted_list = [name for namelist in formatted_list for name in namelist]  # flatten the list
        # (when there isn't enough names to reach the threshold, the small list does not get append
        # to the bigger list, the lines below fixes that):
        if length_list[-1] not in formatted_list:
            formatted_list = length_list

        return formatted_list


if __name__ == '__main__':
    """ 
    Opens the GFF and BlastX files, creates a file that contains ranges of overlapping sequences.
    Then, it creates new ranges that contains the overlaps between ranges created from GFF and BlastX files.
    The program then writes the combined ranges into a new file.
    """
    # WORKING WITH GFF:
    gff_file = 'Hp.augustus.hints.gff'
    gff_tabular = GenomeRange(filename=gff_file)  # create new object for GFF file
    gff_n = gff_tabular.read_GFF()  # select and store a sorted list of features
    print(f'\n{gff_n} features read from {gff_file}')
    gff_tabular.sort_by_pos()

    gff_range = gff_tabular.find_range()  # determine overlap
    gff_range.filename = 'gff_ranges.txt'
    range_n = gff_range.write_file()  # write the ranges into a file
    print(f'{range_n} ranges written to {gff_range.filename}')

    # WORKING WITH BLASTX RESULTS:
    blast_file = 'genome.small_uniref50.dmndblastx'
    blast_tabular = GenomeRange(filename=blast_file)
    blast_n = blast_tabular.read_blast(e_threshold=1e-50)
    print(f'\n{blast_n} features read from {blast_file}')
    blast_tabular.sort_by_pos()

    blast_range = blast_tabular.find_range()
    blast_range.filename = 'blast_ranges.txt'
    range_n = blast_range.write_file()
    print(f'{range_n} ranges written to {blast_range.filename}')

    combined_tabular = GenomeRange()  # new range that combines previously created ranges objects
    # combine the list of ranges from GFF and blast
    combined_tabular.combine_ranges(ranges2combine=[blast_tabular.features, gff_tabular.features])
    combined_tabular.sort_by_pos()  # sort the combined ranges

    combined_range = combined_tabular.find_range()  # determine overlap and return new ranges object
    combined_range.filename = 'vo21_combined.ranges.txt'
    combined_range.write_combined_file()  # write the ranges into a file
```
