# TCRMatch
### Maintainer: Austin Crinklaw
#### Please open any issues on Github, or feel free to email acrinklaw(at)lji(dot)org with feedback or concerns


## Requirements:
- Linux OS
- Python 3+
  - Python packages: numpy

## Installation:
For now, TCRMatch can be installed by cloning the git repo, navigating to the TCRMatch folder and executing:
```shell
pip install .
```


## Usage:
### Input  
-  A text file containing newline separated CDR3beta amino acid sequences
  ```
ASSLRWGDEQY
SADRVGNTLY
ASSFGREQY
ASSQDQVQDTQY
ASSPGQGPYEQY
ATSSGTGAYEQY
ASEAGGSYEQY
ASGDRGADTQY
ASSQAGAYEQY
  ```

- Alternatively, you can upload a TSV file in the AIRR format [(see specifications here)](https://docs.airr-community.org/en/stable/datarep/rearrangements.html).
This can be specified by passing in the command line parameter ```-f airr```


### Commands
- To update the IEDB data file:
```shell
python -m TCRMatch update
```
-  Using a .txt file as an input:
```shell
python -m TCRMatch match -i /path/to/input.txt -o /path/to/output.txt
```

There are a few other command line parameters:
- -t specifies the threshold (should be a value between 0 and 1)
- -p specifies the number of threads to use if operating on more than 1 core
- -f specifies the input file format, this is optional but needs to be specified as ```airr``` if using AIRR tsv format files

### API
The TCRMatch module also exposes a simple method for calculating the distance between two CDR3b sequences. This can be called by importing the module like so:
```Python
from tcrmatch_c import tcrmatch

res = tcrmatch("ASSQDRDTQY", "ASGDAGGGYEQY")
```
The tcrmatch method returns a tuple containing the following format (seq1, seq2, tcrmatch_score)
```Python
("ASSQDRDTQY", "ASGDAGGGYEQY", .74)
```

### Output  
-  Output file has 7 columns in TSV format. 
-  First column is the user provided input sequence.  
-  Second column contains a CDR3beta sequence from the IEDB scoring higher than the threshold
-  The third column contains the score based on the kernel similarity metric (ranging from threshold to .99)
- The fourth column contains the epitopes associated with the matched IEDB sequence. If there are multiple, they will be comma separated.
- The fifth column contains the receptor groups (these can be used to reference the sequences back to the IEDB. If the receptor group is 12843, you can find all the receptors associated by visiting https://iedb.org/receptor/12843)
- The sixth column contains the name of the specific antigen for the matched receptor
- The sevent column contains the name of the source organism for the antigen

The last two columns are not always available depending on the data that was curated.

*Note that due to the trimming of IEDB CDR3b sequences, there may be multiple receptor groups that map to the same trimmed sequence. In these cases, the receptor groups will be comma separated and each link to their own unique receptor on the IEDB.

| input_sequence | match_sequence | score | epitopes                      | receptor_group | antigen                                                                               | source_organism                                                                                     |
|----------------|----------------|-------|-------------------------------|----------------|---------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| ASSSRSSYEQY    | ASSNRSSYEQY    | 0.97  | GILGFVFTL                     | 16861          | Matrix protein 1                                                                      | Influenza A virus                                                                                   |
| ASSSRSSYEQY    | ASSARSSYEQY    | 0.98  | GILGFVFTL,RAKFKQLL,GLCTLVAML  | 53258          | Matrix protein 1,Trans-activator protein BZLF1,Transcriptional regulator IE63 homolog | Influenza A virus,Human herpesvirus 4 (Epstein Barr virus),Human herpesvirus 4 (Epstein Barr virus) |
| ASSSRSSYEQY    | ASSSRASYEQY    | 0.98  | GILGFVFTL,GILGFVFTL           | 55749,35208    | Matrix protein 1,Matrix protein 1                                                     | Influenza A virus,Influenza A virus                                                                 |
| ASSSRSSYEQY    | ASSERSSYEQY    | 0.97  | AMKRHGLDNYREYSLGN             | 21200          |                                                                                       |                                                                                                     |
| ASSSRSSYEQY    | ASSKRSSYEQY    | 0.97  | GILGFVFTL                     | 41132          | Matrix protein 1                                                                      | Influenza A virus                                                                                   |
| ASSSRSSYEQY    | ASSRSSYEQY     | 0.98  | FVCNLLLLFVTVYSHLLLV,GILGFVFTL | 120740,13575   | ORF3a protein [Severe acute respiratory syndrome coronavirus 2],Matrix protein 1      | SARS-CoV2,Influenza A virus                                                                         |
| ASSSRSSYEQY    | ASSQRSSYEQY    | 0.97  | GILGFVFTL,GILGFVFTL           | 35204,54706    | Matrix protein 1,Matrix protein 1                                                     | Influenza A virus,Influenza A virus                                                                 |

## How it works:
- TCRMatch implements a similarity metric as defined in [arXiv:1205.6031v2](https://arxiv.org/abs/1205.6031v2)
- Each input sequence is compared to all TCR CDR3beta sequences characterized in the IEDB
- Sequences that have a  score greater than the threshold are returned as matches

## References:
[Chronister, William D. et al. "Tcrmatch: Predicting T-Cell Receptor Specificity Based On Sequence Similarity To Previously Characterized Receptors". Frontiers In Immunology, vol 12, 2021. Frontiers Media SA, doi:10.3389/fimmu.2021.640725.](https://www.frontiersin.org/articles/10.3389/fimmu.2021.640725/full)

["Towards a Mathematical Foundation of Immunology and Amino Acid Chains" arXiv:1205.6031v2](https://arxiv.org/abs/1205.6031v2)

