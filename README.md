# Frequent Itemset Mining

Python implementations of two classic frequent-itemset mining algorithms, Apriori (with an optional closed-itemset mode) and FP-Growth, run against synthetic transaction datasets of 1 thousand to 1 million transactions. Each miner reads a transaction file, finds every itemset whose support meets a minimum-support ratio, and writes the results sorted by support. The output format matches the bundled verification script, so results can be checked against a reference answer.

## Algorithms

| Script | Algorithm | Notes |
| --- | --- | --- |
| `apriori.py` | Apriori | Level-wise search. Candidate k-itemsets are joined from the frequent (k-1)-itemsets, then their support is counted over all transactions. With `-c`, it also derives closed itemsets: an itemset is dropped when a frequent superset at the next level has the same support. Adapted from the open-source asaini/Apriori project, as noted in the file header. |
| `fp_growth.py` | FP-Growth | Builds an FP-tree from frequency-ordered transactions, with a linked "route" per item for header-table traversal. It mines recursively by building conditional trees from each item's prefix paths. Adapted from an existing open-source implementation; the original author header is kept in the file. |
| `fpgrowth.py` | FP-Growth (alternative) | A second FP-Growth implementation with an explicit header table and conditional pattern bases (`constructTree`, `findPrefixPath`, `mineTree`). The support of each frequent itemset is recounted over the transaction list before output. |

## Datasets

All transaction files use one transaction per line in the form `<id> <id> <item_count> <item> <item> ...`. The miners skip the first three fields.

| Path | Transactions | Distinct items | Avg. transaction length |
| --- | --- | --- | --- |
| `A_1000_500/A_1000_500.data` | 1,000 | 500 | 10 |
| `B_100000_600/B_100000_600.data` | 100,000 | 600 | 10 |
| `C_1000000_600/C_1000000_600.data` | 1,000,000 | 600 | 10 |
| `toy_dataset_and_verification_code/T10I6001M.data` | 1,000,000 | | |

Each synthetic dataset folder also contains the generator configuration (`.conf`) and a summary of the embedded patterns (`.pat`). `toy_dataset_and_verification_code/T10I6001Ms1_freq_itemsets.txt` is the reference answer for the toy dataset: 341 frequent itemsets at 1% minimum support.

## Repository structure

```
.
├── apriori.py
├── fp_growth.py
├── fpgrowth.py
├── A_1000_500/            # .data, .conf, .pat
├── B_100000_600/
├── C_1000000_600/
└── toy_dataset_and_verification_code/
    ├── T10I6001M.data
    ├── T10I6001Ms1_freq_itemsets.txt
    └── verification code/
        └── ItemsetVerifier.py
```

## Requirements

- Python 3
- Standard library only (`optparse`, `argparse`, `itertools`, `collections`, `re`)

## Usage

All three miners take the same two options:

| Flag | Meaning | Default |
| --- | --- | --- |
| `-f`, `--inputFile` | Transaction file | `A.data` |
| `-s`, `--minSupport` | Minimum support as a fraction of transactions (for example `0.01` for 1%) | `0.01` |

`apriori.py` also accepts `-c` / `--closed` to mine closed itemsets.

```
python apriori.py -f A_1000_500/A_1000_500.data -s 0.01
python apriori.py -f A_1000_500/A_1000_500.data -s 0.01 -c
python fp_growth.py -f B_100000_600/B_100000_600.data -s 0.01
python fpgrowth.py -f C_1000000_600/C_1000000_600.data -s 0.01
```

### Output files

Files are written to the current directory. Each result line is `<support %, one decimal>\t{item, item, ...}`, sorted by descending support.

| Script | Output |
| --- | --- |
| `apriori.py` | `apriori_itemset_list.txt` (frequent itemsets) and `statistics_file.txt` (the total count, then per-level counts of frequent k-itemsets) |
| `apriori.py -c` | `closed_itemset_list.txt` |
| `fp_growth.py`, `fpgrowth.py` | `fp_itemset_list.txt` (both scripts write the same file name) |

### Verifying results

`ItemsetVerifier.py` compares two itemset files: `-r` / `--reference` for the expected answer and `-s` / `--submission` for the file to check. It prints any itemsets missing from either side and the number of itemsets in each file. For example, to check an FP-Growth run on the toy dataset:

```
python fp_growth.py -f toy_dataset_and_verification_code/T10I6001M.data -s 0.01
python "toy_dataset_and_verification_code/verification code/ItemsetVerifier.py" -r toy_dataset_and_verification_code/T10I6001Ms1_freq_itemsets.txt -s fp_itemset_list.txt
```

The toy dataset, its reference answer and the verification script come from a starter framework.
