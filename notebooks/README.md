# FFNN Text Classification Notebook

This project teaches text classification with a feed-forward neural network (FFNN) using Bag of Words (BOW) features. The main learning artifact is the notebook [v2-chap07_ffnn.ipynb](./v2-chap07_ffnn.ipynb), which trains a classifier on the AG News dataset.

This README is written for junior data scientists who want to understand both what the notebook does and why each step matters.

## What You Will Learn

By working through the notebook, you will learn how to:

- load and inspect a real text classification dataset
- clean raw text and prepare it for modeling
- tokenize text into word-level units
- build a vocabulary from training data
- convert text into sparse Bag of Words features
- train a simple neural network with PyTorch
- evaluate a classifier with accuracy, a classification report, and a confusion matrix

## Notebook Goal

The notebook predicts one of four news categories from article title and description text:

- World
- Sports
- Business
- Sci/Tech

## End-to-End Pipeline

```mermaid
flowchart TD
    A[Load AG News CSV files] --> B[Inspect labels and class balance]
    B --> C[Clean text and combine title plus description]
    C --> D[Tokenize text with NLTK]
    D --> E[Split into train and dev sets]
    E --> F[Build vocabulary from training tokens]
    F --> G[Convert tokens to BOW feature vectors]
    G --> H[Train FFNN with PyTorch]
    H --> I[Plot loss and accuracy]
    I --> J[Evaluate on test set]
    J --> K[Classification report and confusion matrix]
```

## Project Structure

```text
Gentle-NLP/
|-- README.md
|-- v2-chap07_ffnn.ipynb
|-- v2.md
`-- data/
    `-- ag_news_csv/
        |-- train.csv
        |-- test.csv
        `-- classes.txt
```

## Dataset

The notebook uses the AG News topic classification dataset stored in [data/ag_news_csv](./data/ag_news_csv).

Each example contains:

- a class index
- a title
- a description

The notebook combines the title and description into one lowercase text field, then replaces backslashes with spaces before tokenization.

## Core Ideas

### 1. Bag of Words

Bag of Words ignores word order and represents a document by counting how often each vocabulary term appears.

Example:

- text: `market falls as oil rises`
- tokens: `[market, falls, as, oil, rises]`
- BOW vector: counts for those words in the vocabulary

This is simple, fast, and still useful for baseline text classification.

### 2. Tokenization

Tokenization breaks text into smaller parts, usually words and punctuation.

```mermaid
flowchart LR
    A[Raw text] --> B[Tokenizer]
    B --> C[Word tokens]
```

In this notebook, tokenization is done with NLTK's `word_tokenize`.

### 3. Feed-Forward Neural Network

After text is converted into numeric BOW vectors, the model learns patterns that connect word counts to class labels.

```mermaid
graph LR
    A[Input BOW vector] --> B[Dropout]
    B --> C[Linear layer]
    C --> D[ReLU]
    D --> E[Dropout]
    E --> F[Linear output layer]
    F --> G[4 class scores]
```

The notebook uses:

- an input layer sized to the vocabulary
- one hidden layer with 50 units
- ReLU activation
- dropout for regularization
- an output layer with 4 scores, one per class

## Notebook Walkthrough

### 1. Initialization

The notebook imports PyTorch, NumPy, pandas, and tqdm, then sets a random seed for reproducibility.

It also picks a device automatically:

- `cuda` if a compatible GPU is available
- `cpu` otherwise

### 2. Load and Explore the Training Data

The training CSV is loaded into a pandas DataFrame. Then the class names from `classes.txt` are added so the labels are easier to interpret.

The notebook also plots label counts to confirm that the classes are balanced.

### 3. Clean the Text

The notebook creates a new `text` column by concatenating the title and description. It lowercases the text and replaces stray backslashes with spaces.

### 4. Tokenize the Text

The cleaned text is tokenized with NLTK.

The notebook now downloads `punkt_tab` and `punkt` automatically before tokenization, which helps avoid common NLTK resource errors in fresh environments.

### 5. Train and Dev Split

The dataset is split into:

- 80% training data
- 20% development data

The split is:

- reproducible through `random_state=seed`
- stratified by class label so class balance is preserved

### 6. Build the Vocabulary

The notebook counts tokens in the training set and keeps only tokens whose frequency is above a threshold.

This reduces noise and keeps the feature space manageable.

Special handling:

- `[UNK]` is used for unknown tokens not seen often enough in training

### 7. Convert Tokens to Feature Vectors

Each document is converted into a sparse dictionary of token counts. The custom PyTorch dataset then expands that sparse representation into a dense tensor when an item is requested.

### 8. Train the Neural Network

The model is trained with:

- `CrossEntropyLoss` for multi-class classification
- `Adam` optimizer
- batch size of `500`
- `5` epochs

```mermaid
flowchart TD
    A[Start epoch] --> B[Get batch]
    B --> C[Forward pass]
    C --> D[Compute loss]
    D --> E[Backpropagation]
    E --> F[Optimizer step]
    F --> G[Track loss and accuracy]
    G --> H{More batches?}
    H -- Yes --> B
    H -- No --> I[Run dev evaluation]
```

During training, the notebook stores:

- training loss
- development loss
- training accuracy
- development accuracy

These values are later plotted to help you see whether the model is learning or overfitting.

### 9. Evaluate on the Test Set

The notebook repeats preprocessing for the test set, runs the trained model, and reports:

- per-class precision, recall, and F1-score
- overall accuracy
- a normalized confusion matrix

These outputs help answer two questions:

- How well does the model perform overall?
- Which classes does it confuse most often?

## Recommended Run Order

Run the notebook from top to bottom.

```mermaid
flowchart TD
    A[1. Setup imports and random seed] --> B[2. Load train data]
    B --> C[3. Add labels and inspect balance]
    C --> D[4. Clean text]
    D --> E[5. Tokenize]
    E --> F[6. Split train and dev]
    F --> G[7. Build vocabulary]
    G --> H[8. Create features]
    H --> I[9. Define dataset and model]
    I --> J[10. Train]
    J --> K[11. Plot metrics]
    K --> L[12. Evaluate on test data]
```

If the kernel restarts, rerun the earlier cells before continuing because later steps depend on variables created above them.

## Environment Notes

The notebook currently expects these Python packages:

- torch
- numpy
- pandas
- matplotlib
- nltk
- tqdm
- scikit-learn

If `scikit-learn` is missing, install it in the notebook kernel.

If NLTK tokenization fails because tokenizer data is missing, the notebook already includes code to download the required resources.

The notebook also avoids the `ipywidgets` dependency path for tqdm, which makes it more reliable in lightweight notebook environments.

## What to Watch For as You Learn

As you run the notebook, pay attention to these questions:

- Does the development accuracy improve over epochs?
- Does development loss stop improving while training loss keeps dropping?
- How large is the vocabulary after thresholding?
- Which classes are easiest or hardest to separate?
- What information does BOW lose compared with more advanced text encoders?

## Good Follow-Up Exercises

Once you understand the notebook, try these extensions:

1. Change the vocabulary threshold and see how performance changes.
2. Increase or decrease the hidden layer size.
3. Add more epochs and compare train versus dev curves.
4. Replace BOW with TF-IDF features.
5. Compare this FFNN baseline with an embedding-based model.

## Summary

This notebook is a strong baseline project for learning practical NLP. It shows the full workflow from raw text to evaluation without hiding the mechanics behind a large framework.

For junior data scientists, that is the main value here: you can see each stage clearly, understand what data is flowing through the pipeline, and build intuition before moving on to more advanced models.
