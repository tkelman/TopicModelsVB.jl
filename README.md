# TopicModelsVB.jl
A Julia Package for Variational Bayesian Topic Modeling.

Topic Modeling is concerned with discovering the latent low-dimensional thematic structure within corpora of documents.  Modeling this latent thematic structure is done using either Markov chain Monte Carlo methods, or variational Bayesian methods.  The former approach is slower but unbiased, in that given infinite time, the desired model will be fit exactly.  The latter method is faster (often much faster), but is biased.  This package takes the latter approach to topic modeling.

# Datasets
Included in TopicModelsVB.jl are three datasets:

1. National Science Foundation Abstracts 1989 - 2003:
  * 128804 documents
  * 25319 lexicon

2. CiteULike Science Article Database:
  * 16980 documents
  * 8000 lexicon
  * 5551 users

3. Computer Magazine Archive Article Collection 1975 - 2014:
  * 330577 documents
  * 16020 lexicon

# Dependencies
```julia
Pkg.add("Distributions.jl")
```

# Install
```julia
Pkg.add("TopicModelsVB")
```

# Corpus
Let's begin with the Corpus datastructure.  The Corpus datastructure has been designed for maximum ease-of-use.  Datasets must still be cleaned and put into the appropriate format, but once a dataset is in the proper format, corpora can be easily molded and modified to meet the user's needs.

Any useful corpus needs a non-empty collection of documents.  The document file should be a plaintext file containing lines of delimited numerical values.  Each document is a block of lines, the number of which depends on the amount of information one has about the document.  Since a document is essential a list of terms, each document *must* contain at least one line containing a list of delimited positive integer values corresponding to the terms from which it is composed.  The lines for a particular document block (if they are present) must come in the following order:

1. This line is mandatory, and is a delimited list of positive integers corresponding to the terms which make up the document.

2. A line of delimited positive integers equal in length to the first line, corresponding to the number of times a particular term appears in a document.

3. A line delimited positive integers corresponding to the readers which have the corresponding document in their library.

4. A line of delimited positive integers equal in length to the third line, corresponding to the rating each reader gave the corresponding document.

5. A numerical value in the range ```[-inf, inf]``` denoting the timestamp of the document.

```julia
readcorp(;docfile, lexfile, userfile, titlefile, delim::Char, counts::Bool, readers::Bool, ratings::Bool, stamps::Bool)
```

The file keywords are all strings indicating the path where the file is located.

Even once the files are in the correct format and are read into a corpus, it's still often the case that the files are not sufficiently cleaned and formatted to be usable by the models.  Thus it's **important** that the user always runs one of the following
```julia
fixcorp!(corp; kwargs...)
```
or
```julia
padcorp!(corp; kwargs...)
fixcorp!(corp; kwargs...)
```
or
```julia
cullcorp!(corp; kwargs...)
fixcorp!(corp; kwargs...)
```
first, and then runs ```fixcorp!``` afterwards.  Padding a corpus before fixing it will insure that any documents which contain lexkeys or userkeys not in the lex or user dictionaries attached to the corpus are not removed.  Instead generic lex and user keys will be added to the lex and user dicionaries (resp.).

On the other hand, culling a corpus prior to fixing it will remove those documents which contain bogus lex or user keys not contained in the lex and user dictionaries (resp.)

# Models
The available models are as follows:
```julia
LDA(corp, K)
# Latent Dirichlet Allocation model with K topics.

fLDA(corp, K)
# Filtered latent Dirichlet allocation model with K topics.

CTM(corp, K)
# Correlated topic model with K topics.

fCTM(corp, K)
# Filtered correlated topic model with K topics.

DTM(corp, K, delta, pmodel)
# Dynamic topic model with K topics and ∆ = delta.

CTPF(corp, K, pmodel)
# Collaborative topic Poisson factorization model with K topics.
```

Notice that both ```DTM``` and ```CTPF``` have a ```pmodel``` argument.  It is *highly* advisable that you prime these models with a pretrained model from one of the first four topic models, otherwise learning for these models may take a prohibitively long amount of time.

## Tutorial
Let's begin our tutorial with a simple latent Dirichlet allocation (LDA) model with 8 topics, trained on the first 5000 documents from the NSF Abstracts corpus.
```julia
using TopicModelsVB

nsfcorp = readcorp(:nsf)
nsfcorp = nsfcorp[1:5000]
fixcorp!(nsfcorp)

# Notice that the post-fix lexicon is considerably smaller after removing all but the first 5000 docs.

nsflda = LDA(nsfcorp, 8)
train!(nsflda, iter=150)

# training...

showtopics(nsflda)
```

One thing we notice is all the words which would be considered informative to a generic corpus, but which are effectively stop words in a corpus of science article abstracts.  These words will be missed by most stop word lists, and can be a pain to pinpoint and individually remove.  Thus let's change our model to a filtered latent Dirichlet allocation (fLDA) model.
```julia
nsfflda = fLDA(nsfcorp, 8)
train!(nsfflda, iter=150)

# training...

showtopics(nsfflda)
```

Now we see can see that many of the most troublesome corpus-specific stop words have been automatically filtered out of the topics, and those that remain are those which tend to belong to their own, more generic, topic.

For our final test with the NSF Abstracts corpus, let's upgrade our model to a filtered *correlated* topic model (fCTM)
```julia
nsffctm = fLDA(nsfcorp, 8)
train!(nsffctm, iter=150)

# training...

showtopics(nsffctm)
```
Not only have corpus-specific stop words been removed, but we can see that the topics are significantly more well defined and consistent than in the non-correlated LDA model.  Let's take a look at the correlations between topics
```julia
model.sigma
```
