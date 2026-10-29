# Reproducibility and data versioning

**Covers:** reproducibility, random seed, data versioning, DVC, CSV

## Getting the same answer twice {#overview}
Suppose you change something in an experiment and compare the result. If the result also changes when you change nothing, comparison is impossible. Machine learning is more sensitive to this than ordinary software, because its results depend on four things at once: the code, the data, the environment and chance. This article shows how to pin each one down.

## Reproducibility {#reproducibility}
Reproducibility means you can get the same result from an experiment again later. The same code is not enough. You also need the exact training data, the same versions of the libraries and the same source of chance. If any one of the four slips, the result can change enough to confuse a comparison or hide a real problem.

## Random seed {#random-seed}
Let's take chance first. Training uses it in several places: shuffling the rows, choosing which rows go into the test set and picking the starting values of the model's weights. A computer's random numbers are actually calculated from a starting number, the random seed. Fix the seed to 42, and the "random" sequence is identical every time.

## CSV {#csv}
Now the data. CSV, comma-separated values, is a plain-text table in which each row is a line and commas separate the columns. It is the most common format for exchanging training data, because every tool can read it. Its weakness for reproducibility is that a CSV file has no version. Someone can overwrite `train.csv` with new rows, and nothing marks that the file is now different.

## Data versioning {#data-versioning}
Data versioning takes snapshots of a training dataset the way version control takes snapshots of code, so you can go back and retrieve the exact rows a model was trained on. Without it, "the data" means whatever happens to be in the folder today.

## DVC {#dvc}
DVC, Data Version Control, is an open-source tool that versions large datasets and model files alongside your code repository. The big files are not stored in the repository itself. DVC keeps a small text file that records the file's fingerprint, and it stores the real content elsewhere. A commit of the code then points to the exact data it used.

## A worked example {#example}
Maria trains a model on `train.csv` and gets an accuracy of 88.4%. A month later she repeats it and gets 87.1%. What changed? Let's check the four factors.

| Factor | Question | Finding |
|---|---|---|
| Code | Same commit? | Yes |
| Environment | Same library versions? | Yes |
| Data | Same rows? | No: a colleague added 5,000 rows to `train.csv` |
| Chance | Same seed? | No: the seed was not fixed |

There are two causes, and no way to tell how much each contributed. She starts by fixing the seed:

```python
import random

random.seed(42)
first = [random.randint(1, 100) for _ in range(5)]
random.seed(42)
second = [random.randint(1, 100) for _ in range(5)]
print(first == second)  # True
```

The output is `True`: the same seed gives the same sequence. Training libraries have their own seed options, and you must fix each of them too.

Then she versions the data with DVC:

```bash
dvc add train.csv
git add train.csv.dvc
git commit -m "Track training data"
```

The file `train.csv.dvc` is small, and the commit now identifies the exact data file. A month later, `git checkout <commit>` followed by `dvc checkout` brings back the data of that day, and a second attempt gives 88.4% again. The gap of 1.3 points (88.4 − 87.1) is now explained: it came from the added rows and the unfixed chance, and she can test each one separately.

Let's count the saving. Without versioning, finding the cause might take two days of asking colleagues and comparing files. With it, the answer takes minutes: check out the old commit and compare.

One limit to know: fixing the seed does not always give identical results across different hardware or library versions, because floating-point calculations can differ slightly. Aim for the same result within a small margin, and record the environment as well.

## A common mistake {#mistake}
A common slip is overwriting the training file in place. The old version is gone, and no earlier result can be reproduced. Save a new version instead.

Another slip is fixing the seed in one place when the shuffle, the split and the model each use their own.

## Key points {#key-points}
- Results depend on code, data, environment and chance.
- A random seed makes "random" steps repeatable.
- A CSV file has no version history by itself.
- DVC versions data beside the code, so a commit identifies the exact data.
