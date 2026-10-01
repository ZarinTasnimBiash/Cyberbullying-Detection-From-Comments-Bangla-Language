# Cyberbullying Detection in Bangla

Teaching a computer to recognise bullying in Bangla (Bengali) social-media comments, and to tell what kind of bullying it is.

> **Content warning:** this project works with real online comments. The notebooks and their saved outputs contain offensive, sexual, religious and threatening language in Bangla.

---

## Why this project exists

Bangla is one of the most widely spoken languages in the world, and more and more Bangla speakers are online every year. Online harassment has grown with them. Platforms already have good automatic tools for English, but there are very few for Bangla.

We wanted to answer two questions:

1. **Is this comment bullying or not?** (a yes/no question)
2. **If it is bullying, what kind is it?** (sexual, threatening, trolling, or religious)

## The data

We collected and hand-labelled **44,001 comments** from public Facebook pages of celebrities, politicians, athletes, singers and other public figures. Each comment is labelled with one of five categories:

| Label | What it means |
|---|---|
| **Not bully** | A normal comment with no harassment |
| **Sexual** | Sexually abusive or harassing language |
| **Threat** | Threatens harm to the person |
| **Troll** | Mocking, insulting or ridiculing the person |
| **Religious** | Attacks someone because of religion or belief |

The dataset is free to download from Mendeley Data: <https://data.mendeley.com/datasets/9xjx8twk8p>

It is described in detail, with charts and statistics, in our dataset paper:
**[Bangla Text Dataset and Exploratory Analysis for Online Harassment Detection](https://arxiv.org/abs/2102.02478)**

## How it works

The approach has three steps.

**1. Clean and prepare the text.**
Common filler words that carry little meaning (such as "and", "but" or "this" in Bangla) are removed. Each word is then turned into a number so the computer can process it.

**2. Train two neural networks.**
Both use the same design: a *convolutional* layer that picks up short word patterns, followed by an *LSTM* layer that reads the comment in order and remembers context.
- The **binary model** answers "bully or not bully?"
- The **multiclass model** picks one of the five categories.

**3. Combine their answers (stacking).**
Each comment now has two predictions, one from each network. A second, simpler classifier learns from both predictions together and makes the final call. We tried four of these: Random Forest, Support Vector Machine (SVM), k-Nearest Neighbours and Naive Bayes.

```
               ┌──────────────────────┐
  comment ───► │ Binary network       │ ──► bully / not bully ──┐
          │    └──────────────────────┘                         ├──► final classifier ──► final label
          └──► ┌──────────────────────┐                         │    (SVM, Random Forest, …)
               │ Multiclass network   │ ──► one of 5 labels ────┘
               └──────────────────────┘
```

The full method is described in our paper:
**[Cyberbullying Detection Using Deep Neural Network from Social Media Comments in Bangla Language](https://arxiv.org/abs/2106.04506)**

## Results

All models were tested on comments they had not been trained on (15% of the data held out for testing).

| Model | Task | Accuracy |
|---|---|---|
| Neural network (CNN + LSTM) | Bully vs. not bully | **87.9%** |
| Neural network (CNN + LSTM) | 5 categories | 80.3% |
| Neural networks + SVM | 5 categories | **85%** |
| Neural networks + Random Forest | 5 categories | 84% |
| Neural networks + k-Nearest Neighbours (k = 7) | 5 categories | 84.4% |
| Neural networks + Naive Bayes | 5 categories | 79% |

In plain terms, the binary model gets roughly 9 out of 10 comments right. Combining the two networks raises the five-category accuracy from about 80% to 85%.

The hardest category is **Threat**. It has far fewer examples than the others, so the models often miss threats.

## What's in this repository

| File | What it does |
|---|---|
| [`BinaryClassification.ipynb`](BinaryClassification.ipynb) | Trains and evaluates the bully / not-bully network, then saves its prediction for every comment |
| [`MulticlassClassification.ipynb`](MulticlassClassification.ipynb) | Trains and evaluates the five-category network, then saves its prediction for every comment |
| [`2_Model_Stacked.ipynb`](2_Model_Stacked.ipynb) | Combines both networks' predictions and trains the final classifiers (Random Forest, SVM, k-NN, Naive Bayes) |

The notebooks are saved with their outputs, so you can read the results, charts and confusion matrices on GitHub without running anything.

## Running it yourself

The notebooks were written for **Google Colab** with Python 3.6 and **TensorFlow 2.4**. They use `model.predict_classes()`, which was removed in TensorFlow 2.6. Use TensorFlow 2.4 or 2.5, or replace that call with `(model.predict(x) > 0.5).astype("int32")` (binary) or `np.argmax(model.predict(x), axis=-1)` (multiclass).

Main libraries: `tensorflow`, `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `openpyxl`, `tqdm`.

1. **Get the data.** Download the dataset from [Mendeley Data](https://data.mendeley.com/datasets/9xjx8twk8p). Save it as an Excel file named `without racism.xlsx`, with the data on `Sheet1`, in the same folder as the notebooks. The notebooks use the columns `comment`, `label`, `name`, `Category` and `Gender`.
2. **Run `MulticlassClassification.ipynb`.** It trains the five-category model and writes its predictions to `Prediction.xlsx`, in a column called `pred5`.
3. **Prepare the input for the binary notebook.** Make a file `Pred5.xlsx` with the `comment`, `label` and `pred5` columns.
4. **Run `BinaryClassification.ipynb`.** It trains the binary model, adds its predictions as a `pred2` column, and saves `AllPred.xlsx`.
5. **Run `2_Model_Stacked.ipynb`.** It reads `AllPred.xlsx` and trains and evaluates the final classifiers.

Some cells use Colab-specific paths (`/content/...`, `/content/drive/...`). If you run the notebooks somewhere else, change these paths.

## Citation

If you use this code or the dataset, please cite:

```bibtex
@article{ahmed2021cyberbullying,
  title   = {Cyberbullying Detection Using Deep Neural Network from Social Media Comments in Bangla Language},
  author  = {Ahmed, Md Faisal and Mahmud, Zalish and Biash, Zarin Tasnim and Ryen, Ahmed Ann Noor and Hossain, Arman and Ashraf, Faisal Bin},
  journal = {arXiv preprint arXiv:2106.04506},
  year    = {2021}
}

@article{ahmed2021bangla,
  title   = {Bangla Text Dataset and Exploratory Analysis for Online Harassment Detection},
  author  = {Ahmed, Md Faisal and Mahmud, Zalish and Biash, Zarin Tasnim and Ryen, Ahmed Ann Noor and Hossain, Arman and Ashraf, Faisal Bin},
  journal = {arXiv preprint arXiv:2102.02478},
  year    = {2021}
}
```

## Authors

Md Faisal Ahmed, Zalish Mahmud, Zarin Tasnim Biash, Ahmed Ann Noor Ryen, Arman Hossain, Faisal Bin Ashraf
