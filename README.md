# kNN Movie Recommender

A study of the k-nearest-neighbors algorithm as a movie recommender, on the MovieLens
100K dataset. Four kNN distance metrics are compared for recommendation quality, and a
hybrid item-based recommender with a graphical interface layers popularity and a
weighted-rating rule on top of kNN to cover the cases kNN handles poorly. The work
includes an IEEE-style paper.

## Authors and who built what

This was a three-person academic project at Columbus State University (TSYS College of
Computer Science).

- **Ashley Bedford**: the recommendation and evaluation code, the four kNN models, the
  sparse-matrix pipeline, and the model-comparison method in `evaluation/`.
- **Luis Barriga**: the graphical recommender application and the popularity and
  weighted-rating layer in `gui/`.
- **Aida Hashemlou**: contributions to the paper.

The paper was written by the authors covering their respective parts of the work.

## What is here

    evaluation/   Ashley's notebook: builds four kNN models and compares them
    gui/          Luis's tkinter app: an item-based recommender with a popularity and
                  weighted-rating hybrid
    paper/        the IEEE-style write-up
    requirements.txt

The evaluation notebook loads the ratings, pivots them into a movies-by-users matrix,
converts it to a compressed sparse row (CSR) matrix, and fits four kNN models that
differ only in their distance metric: Euclidean, cosine, Minkowski, and Manhattan. It
then scores each model with a modified Pearson-correlation threshold method and reports
accuracy and recall with confusion matrices.

The GUI app is the product side: it recommends similar movies with item-based kNN, and
because kNN cannot reason about a movie a user disliked or serve a brand-new user, it
falls back to a popularity score and an IMDB-style weighted rating to handle those
cold-start and negative-feedback cases.

## About the results, read this first

The notebook reports accuracy and recall near 1.0 for the Euclidean, Manhattan, and
Minkowski models. That number is not a claim that the recommender is perfect. It is a
consequence of how the evaluation is set up, and the paper is explicit about this.

kNN has no built-in way to represent a rating a user would dislike, so the evaluation
only ever produces two of the four confusion-matrix outcomes: true positives and false
negatives. Every evaluated user is one whose own high ratings average at or above the
threshold of 3, and the metric checks whether the movies the model recommended also
average at or above 3. When they do, it counts as a true positive. So an accuracy of
1.0 means the recommended movies were, on average, rated highly by users, not that the
model classified anything with perfect precision. The honest reading is that these three
metrics reliably surface well-liked movies on this dataset, and that a fuller evaluation
would need a way to handle negative feedback. The paper lists that as future work and
leaves the precision and F1 formulas in the code for whoever picks it up.

Cosine similarity scored slightly lower, missing on four users, which the paper notes
may reflect cosine suiting larger, higher-dimensional datasets better than this one.

## Running it

Get the data first. The code uses MovieLens latest-small (about 100K ratings). Download
`ml-latest-small` from https://grouplens.org/datasets/movielens/ and place `ratings.csv`
and `movies.csv` in the folder you run from. The dataset is not committed here, since
GroupLens asks that it not be redistributed.

The evaluation notebook:

    pip install -r requirements.txt
    # place ratings.csv and movies.csv in evaluation/
    jupyter notebook evaluation/kNN_movie_recommendation_evaluation.ipynb

Running it fits the four models, writes their pickled versions, and prints the metric
results and confusion matrix for each.

The GUI app:

    # place ratings.csv and movies.csv in gui/
    python gui/main.py

The app needs a display, since it is a desktop tkinter window.

## Method notes

The CSR matrix is what makes kNN practical here. The ratings are extremely sparse, most
users have rated a tiny fraction of movies, so storing only the non-zero entries keeps
both memory and the similarity search efficient. Item-based filtering was chosen over
user-based because the goal is to recommend movies, and item-based avoids the cold-start
problem that user-based filtering has before a new user has rated anything.

The four distance metrics trade off differently: Euclidean and Manhattan measure
straight-line and grid distance between rating vectors, cosine measures the angle
between them and is often better for high-dimensional sparse data, and Minkowski
generalizes Euclidean and Manhattan through an exponent parameter. On this dataset the
three distance-based metrics behaved almost identically, and cosine trailed slightly.

## Limitations

Single dataset and single size. The team tried the 25M-rating version and found it too
large to load into the dense matrices this approach builds, and settled on the
100K-rating small set for fast iteration. Singular value decomposition would be the path
to the larger set and is noted as future work.

No negative-feedback handling in kNN, which is why the evaluation is bounded as
described above, and why the GUI layers popularity and weighted-rating rules on top.

Non-rated movies are filled with zero and ignored, which nudges unrated titles toward
the low end; the paper suggests imputing an average rating such as 2.5 as an
improvement.

## Reference

Barriga, L., Bedford, A., and Hashemlou, A. In the Neighborhood: Revisiting kNN as a
Tool for Movie Recommendation Systems. TSYS College of Computer Science, Columbus State
University. See `paper/`.

Dataset: F. Maxwell Harper and Joseph A. Konstan. 2015. The MovieLens Datasets: History
and Context. ACM Transactions on Interactive Intelligent Systems 5, 4.
