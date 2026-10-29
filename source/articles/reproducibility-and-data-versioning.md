# Reproducibility and data versioning

**Covers:** reproducibility, random seed, data versioning, DVC, CSV

## Getting the same answer twice {#overview}
In an experiment, you change something and compare the result. If the result changes even when you changed nothing, comparison is impossible. Machine learning is more sensitive to this than ordinary software, because its results depend on four things at once: the code, the data, the environment and chance.

## Reproducibility {#reproducibility}
Reproducibility is being able to get the same result from a machine learning experiment again later. It takes more than the same code. You also need the exact training data, the same versions of the libraries and the same source of chance. Missing any one of the four can change the result by enough to confuse a comparison, or to hide a real problem.

## Random seed {#random-seed}
A random seed is a starting number that makes a program's "random" choices come out the same each time. Training uses chance in several places: shuffling the rows, choosing which rows go into the test set, and setting the starting values of the model's weights. A computer's random numbers are actually calculated from a seed. Fix the seed to 42, and the "random" sequence is identical on every execution.

## CSV {#csv}
CSV, comma-separated values, is a plain-text table in which each row is a line and the columns are split by commas. It is the most common format for exchanging training data, because every tool can read it. Its weakness for reproducibility: a CSV file has no version. Someone can overwrite `train.csv` with new rows, and nothing marks that the file is now different.

## Data versioning {#data-versioning}
Data versioning means taking snapshots of a training dataset in the way version control takes snapshots of code, so that you can go back and retrieve the exact rows a model was trained on. Without it, "the data" is whatever happens to be in the folder today.

## DVC {#dvc}
DVC, Data Version Control, is an open-source tool that versions large datasets and model files alongside your code repository. Big files are not stored in the repository itself. DVC keeps a small text file that records the file's fingerprint, and stores the real content in a separate place. A commit of the code then points to the exact data it used.

## A worked example {#example}
Maria trains a model on `train.csv` and gets an accuracy of 88.4%. A month later, she repeats it and gets 87.1%. What changed? Check the four factors.

| Factor | Question | Finding |
|---|---|---|
| Code | Same commit? | Yes |
| Environment | Same library versions? | Yes |
| Data | Same rows? | No: a colleague added 5,000 rows to `train.csv` |
| Chance | Same seed? | No: the seed was not set |

Two causes, and no way to tell how much each contributed. She fixes the seed:

```python
import random

random.seed(42)
first = [random.randint(1, 100) for _ in range(5)]
random.seed(42)
second = [random.randint(1, 100) for _ in range(5)]
print(first == second)  # True
```

The output is `True`: the same seed gives the same sequence. Training libraries have their own seed settings, and each must be set too.

Then she versions the data with DVC:

```bash
dvc add train.csv
git add train.csv.dvc
git commit -m "Track training data"
```

The file `train.csv.dvc` is small, and the commit now identifies the exact file. A month later, `git checkout <commit>` followed by `dvc checkout` brings back the data of that day. A rerun gives 88.4% again. The difference of 1.3 points (88.4 − 87.1) is now explained: it was the added rows and the unseeded chance, and she can test each separately.

Count the saving. Without versioning, finding the cause might take two days of asking colleagues and comparing files. With it, the answer takes minutes: check out the old commit and compare.

A limit to know: fixing the seed does not always give identical results across different hardware or library versions, since floating-point calculations can differ slightly. Aim for the same result within a small margin, and record the environment too.

## A common mistake {#mistake}
Overwriting the training file in place. The old version is gone, and no result can be reproduced. Save a new version. Another mistake is setting the seed in one place when the shuffle, the split and the model each use their own.

## Key points {#key-points}
- Results depend on code, data, environment and chance.
- A random seed makes "random" steps repeatable.
- A CSV file has no version history by itself.
- DVC versions data beside the code, so a commit identifies the exact data.
