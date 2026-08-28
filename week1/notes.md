## Assignment 1 - Word Vectors

### ⭐ The Big Picture
If you remember only this, you should be in good shape:
```
                    NLP WEEK 1

                 Training corpus
                       │
                       ▼
              Word co-occurrences
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
       TruncatedSVD             GloVe
            │                     │
            ▼                     ▼
      Word vectors          Word vectors
            │                     │
            └──────────┬──────────┘
                       ▼
              Compare vectors
                       │
              Cosine similarity
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Similarity   Analogies      Bias
          │            │            │
          ▼            ▼            ▼
    Similar contexts  v₁-v₂+v₃   Training data
```

And the three biggest concepts behind almost all of your questions are:
1. **Distributional hypothesis**: words used in similar contexts tend to have similar vectors.
2. **Static embeddings**: one word gets one vector, so multiple meanings get mixed together.
3. **Embeddings learn statistical patterns, not human concepts**: that's why antonyms can be close, analogies can fail, and biases in training text can appear in the vectors.


### 1. Co-occurrence Matrix
A co-occurrence matrix records how often words appear near each other in a corpus.

### 2. SVD
SVD = Singular Value Decomposition

A matrix \(M\) can be decomposed as:
\[
M = U\Sigma V^T
\]\(U\) = left singular vectors
\(\Sigma\) = singular values
\(V^T\) = right singular vectors
The singular values indicate how much information/structure each component captures.

### 3. TruncatedSVD
Instead of keeping every component, we keep only the top \(k\):
\[
M \approx U_k\Sigma_kV_k^T
\]

This gives us a lower-dimensional representation.

For example:

```
10000-dimensional
       ↓
 TruncatedSVD
       ↓
100-dimensional
```

#### Why use it?
- Reduce dimensionality
- Compress the co-occurrence matrix
- Capture the strongest patterns
- Obtain word embeddings

#### Important
TruncatedSVD finds **mathematical patterns**, but those dimensions aren't necessarily human-interpretable.
You might interpret a dimension as "animals" or "food," but SVD itself doesn't know those concepts.

### 4. Normalizing Word Vectors

Suppose a word vector is:
\[
v=[3,4]
\]

Its length is:
\[
\|v\|=\sqrt{3^2+4^2}=5
\]


Normalize it:
\[\frac{v}{\|v\|}=
[0.6,0.8]
\]

Now its length is 1.
Therefore, all normalized vectors lie on the unit circle in 2D:
\[
x^2+y^2=1
\]

Why normalize?

It makes comparisons focus on direction rather than magnitude.
This is important for cosine similarity.

### 5. Cosine Similarity vs. Cosine Distance
#### Cosine similarity
Measures how similarly two vectors point:
\[
\cos(\theta)
\]

Similarity	Meaning
1	Same direction
0	Perpendicular
-1	Opposite direction


#### Cosine distance
Usually:
\[
\boxed{1-\text{cosine similarity}}
\]Therefore:
Higher cosine similarity = lower cosine distance.

For example:
\[
\text{similarity}=0.8
\]means:
\[
\text{distance}=0.2
\]

### 6. GloVe
GloVe = Global Vectors for Word Representation

GloVe creates word embeddings using global word co-occurrence statistics.

Conceptually:
```
Corpus
  ↓
Co-occurrence statistics
  ↓
GloVe
  ↓
Word vectors
```

GloVe learns word and context vectors that explain the co-occurrence statistics.

A simplified version of its objective is:
\[
w_i^T\tilde w_j+b_i+\tilde b_j
\approx \log X_{ij}
\]where \(X_{ij}\) is the co-occurrence count.

#### GloVe vs. TruncatedSVD
Both use co-occurrence information, but differently:
- TruncatedSVD: Factorize/compress the co-occurrence matrix.

- GloVe: Learn vectors by optimizing an objective based on co-occurrence statistics.

Therefore, they can produce different embeddings and different plots even from the same corpus.

### 7. Why can Antonyms be Close?
You might expect:
```
happy ↔ cheerful     close
happy ↔ sad          far
```

But embeddings can sometimes produce:
```
happy ↔ sad          close
happy ↔ cheerful     farther
```

Why?
Because embeddings capture contextual/distributional similarity, not necessarily semantic equivalence.

happy and sad can occur in very similar contexts:

She was happy about the result.

She was sad about the result.

\[
\boxed{\text{similar context} \neq \text{same meaning}}
\]

This is why antonyms can sometimes have high cosine similarity.


### 8.  Word Analogies
A common analogy is:
\[
\text{man}:\text{woman}::\text{grandfather}:?
\]The vector arithmetic is:
\[
\boxed{g-m+w}
\]where:
\(m\) = man
\(g\) = grandfather
\(w\) = woman

We are essentially doing:
\[
g+(w-m)
\]The vector:
\[
w-m
\]represents something like the male → female direction.
Then we apply that direction to grandfather.
Expected result:
\[
grandmother
\]


### 9.  How Can We Mitigate Bias?
Remove the bias direction from the vectors of words that should be neutral.

For gender, you might estimate a gender direction using:

\[
g = man-woman
\]

Then remove the component of a neutral word's vector that points in that direction.

If \(d\) is the vector for doctor:

\[
projection=
\frac{d\cdot g}{g\cdot g}g
\]Then:
\[
\boxed{
d_{\text{debiased}}
=
d-projection
}
\]

Intuition
```
Original doctor
       ↗
      /
     /  ← gender component
    /
   →

Remove gender component

doctor → neutralized vector
```
