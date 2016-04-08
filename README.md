# Siamese CBOW

## Overview

Siamese CBOW is a neural network architecture for calculating word embeddings
optimised for being averaged across sentences to produce sentence
representations.

Siamese CBOW compares sentence vectors to each other.
A sentence can be a *positive example*, which means that it is supposed to be
semantically similar to the sentence it is compared to, or a
*negative example*, when it is not assumed to be semantically similar.

A sentence embedding is obtained by averaging the word embeddings of the words
in a sentence.
The cosine similarity between sentence pairs is calculated, and the network
tries to give a high cosine similarity to positive examples, and a low cosine
similarity to negative examples.

_IMAGE_

A paper has been written about Siamese CBOW, which is currently under review.

## Installing

Before Siamese CBOW can be run, the following reporitories need to be cloned
too:

* tokenizationUtils <https://bitbucket.org/TomKenter/tokenizationutils>
* ppdbUtils <https://bitbucket.org/TomKenter/ppdbutils>
* InexUtils <https://bitbucket.org/TomKenter/inexutils>
* torontoBookCoprusUtils <https://bitbucket.org/TomKenter/torontobookcorpusutils>

They should be installed next to the Siamese CBOW directory:

    $ ls
    siamese-cbow
    tokenizationutils
    ppdbutils
    inexutils
    torontobookcorpusutils

## Usage

Several input formats are supported, each of them corresponding to a slightly
different use.
In general, Siamese CBOW is called like:

    $ python siamese-cbow.py [OPTIONS] DATA OUTPUT_DIR 

### Required arguments

<table>
<tr>
<td valign="top">DATA</td>
<td>File (in PPDB case) or directory (in Toronto Book
           	     Corpus and INEX case) to read the data from. NOTE that
                        the program runs in aparticular input mode
                        (INEX/PPDB/TORONTO) which is deduced from the
                        directory/file name)
</td></tr>
<tr><td valign="top">OUTPUT_DIR</td>
<td> A file to store the final and possibly intermediate
 word embeddings to (in cPickle format)
</td></tr></table>

### Optional arguments

<table>
<tr><td valign="top">-h, --help</td><td valign="top">show this help message and exit</td></tr>
<tr><td valign="top"> -w2v FILE</td><td valign="top">A word2vec model can be used to initialize the weights for words in the vocabulary file from (missing words just get a random embedding). If the weights are not initialized this way, they will be trained from scratch.</td></tr>
<tr><td valign="top"> -vocab FILE</td><td valign="top">A vocabulary file is simply a file, SORTED BY FREQUENCY of frequence<SPACE>word lines. You can take the top n of these (which is why it should be sorted by frequency). See -max_nr_of_vocab_words.</td></tr>
<tr><td valign="top"> -max_nr_of_vocab_words INT</td><td valign="top">Maximum number of words considered. If this is not specified, all words are considered</td></tr>
<tr><td valign="top"> -max_nr_of_tokens INT</td><td valign="top"> Maximum number of tokens considered per sentence. Default: 50</td></tr>
<tr><td valign="top"> -epochs INT</td><td valign="top">Maximum number of epochs for training. Default: 10</td></tr>
<tr><td valign="top"> -learning_rate FLOAT</td><td valign="top">Learning rate. Default: 1.0</td></tr>
<tr><td valign="top"> -gradient_clipping_bound FLOAT</td><td valign="top">Gradient clipping bound (so gradients will be clipped to [-FLOAT, +FLOAT]).</td></tr>
<tr><td valign="top"> -momentum FLOAT</td><td valign="top">Momentum, only applies when 'nesterov' is used as update method (see -update). Default: 0.0</td></tr>
<tr><td valign="top"> -batch_size INT</td><td valign="top">Batch size. Default: 1</td></tr>
<tr><td valign="top"> -neg INT</td><td valign="top">Number of negative examples. Default: 1</td></tr>
<tr><td valign="top"> -embedding_size INT</td><td valign="top">Dimensionality of the word embeddings. Default: 300</td></tr>
<tr><td valign="top"> -share_weights</td><td valign="top">Turn this option on (a good idea in general) for the embedding weights of the input sentences and the other sentences to be shared.</td></tr>
<tr><td valign="top"> -last_layer LAYER</td><td valign="top">Last layer is 'cosine' or 'sigmoid'. NOTE that this choice also determines the loss function (binary cross entropy or negative sampling loss, respectively). Default: cosine</td></tr>
<tr><td valign="top"> -update UPDATE_ALGORITHM</td><td valign="top">Update algorithm. Options are 'adadelta', 'adamax', 'nesterov' (which uses momentum) and 'sgd'. Default: 'adadelta'</td></tr>
<tr><td valign="top"> -regularize</td><td valign="top">Use l2 normalization on the parameters of the network</td></tr>
<tr><td valign="top"> -dont_lowercase</td><td valign="top">By default, all input text is lowercased. Use this option to prevent this.</td></tr>
<tr><td valign="top"> -store_at_batch INT</td><td valign="top">Store embeddings every INT batches.</td></tr>
<tr><td valign="top"> -store_at_epoch INT</td><td valign="top">Store embeddings every INT epochs (so 1 for storing at the end of every epoch, 10 for for storing every 10 epochs, etc.).</td></tr>
<tr><td valign="top"> -start_storing_at INT</td><td valign="top"> Start storing embeddings at epoch number INT. Default: 0. I.e. start storing right away (if -store_at_epoch is on, that is)</td></tr>
<tr><td valign="top"> -dry_run</td><td valign="top">Build the network, print some statistics (if -v is on) and quit before training starts.</td></tr>
<tr><td valign="top"> -d</td><td valign="top">Debugging mode</td></tr>
<tr><td valign="top"> -v</td><td valign="top">Be verbose</td></tr>
<tr><td valign="top"> -vv</td><td valign="top">Be very verbose</td></tr>
</table>

### Data formats

Three data formats are supported: PPDB, INEX and Toronto Book Corpus.

#### PPDB

The PPDB corpus is a corpus of paired short phrases which are explicitely
marked for semantic similarity.
The corpus can be downloaded from <http://www.cis.upenn.edu/~ccb/ppdb/>.

To run Siamese CBOW on the XL version of the corpus, run this command:

    $ THEANO_FLAGS=floatX=float32 python siamese-cbow.py -v -share_weights \
      -vocab /path/to/ppdbVocabFile.txt -epochs 5 -neg 2 -embedding_size 100 \
      /path/to/PPDB/xl/ppdb-1.0-xl-phrasal myPpdbOutputDir

Running this command will results in a file with embeddings being written to
`myPpdbOutputDir/cosine_sharedWeights_adadelta_lr_1_noGradClip_epochs_5_batch_1_neg_2_voc_37532x100_noReg_lc_noPreInit.pickle`.

We can inspect these embeddings by loading them in Python:

    >>> import wordEmbeddings as we
    >>> oWE_PPDB = we.wordEmbeddings("./myPpdbOutputDir/cosine_sharedWeights_adadelta_lr_1_epochs_5_batch_100_neg_2_300d_noReg_lc_noPreInit_vocab_65535.end_of_epoch_5.pickle")

    >>> oWE_PPDB_epoch_5.most_similar('bad')
    [(u'evil', 0.80611455), (u'naughty', 0.80259472), (u'mess', 0.78487486), (u'mal', 0.77596927), (u'unhappy', 0.75018817), (u'silly', 0.74343985), (u'wrong', 0.74152184), (u'dirty', 0.73908687), (u'pissed', 0.73823059), (u'terrible', 0.73761278)]
    >>> oWE_PPDB_epoch_5.most_similar('good')
    [(u'nice', 0.8981635), (u'genuinely', 0.88940138), (u'truly', 0.88519835), (u'good-looking', 0.87889814), (u'swell', 0.87747467), (u'delicious', 0.87535363), (u'adorable', 0.8678599), (u'charming', 0.86676568), (u'lovely', 0.86469257), (u'rightly', 0.8637867)]
    >>> oWE_PPDB_epoch_5.most_similar('controls')
    [(u'controlling', 0.97573024), (u'inspections', 0.97478831), (u'checks', 0.97122586), (u'surveillance', 0.96790159), (u'oversight', 0.96275693), (u'supervision', 0.95519221), (u'supervises', 0.95428753), (u'supervise', 0.95306516), (u'monitoring', 0.95126694), (u'monitors', 0.9501)]
    >>> oWE_PPDB_epoch_5.most_similar('love')
    [(u'loves', 0.92758971), (u'likes', 0.9195503), (u'apologize', 0.91565502), (u'adore', 0.89775938), (u'apologise', 0.89251494), (u'oh', 0.89192468), (u'ah', 0.88041437), (u'aw', 0.87821764), (u'dear', 0.8773343), (u'wow', 0.87290406)]

#### INEX

The INEX dataset is in fact a Wikipedia dump from November 2012.
It was released as a test collection for the INEX 2013 tweet contextualisation track (_Report on INEX 2013_, P. Bellot, et al, 2013).

The dataset can be downloaded from <http://inex.mmci.uni-saarland.de/tracks/qa/> but please note that a password is required (which is available to INEX participants).
If the dataset can not be obtained, probably another Wikipedia dump will do just as well.

In order to use the data, we first need to preprocess it. and we need to have a vocabulary of all terms occurring in the corpus.
Please see the subsections below.

For now, let's assume the pre-processed data is in `/path/to/inex_paragraph_output_dir/` and we have vocabulary `INEX.vocab/txt`.<br>
Now we can call:

    $ THEANO_FLAGS=floatX=float32 python siamese-cbow.py -v -vocab INEX.vocab.txt \
     -max_nr_of_vocab_words 65535 -share_weights -epochs 2 -batch_size 100 \
     -neg 2 /path/to/inex_paragraph_output_dir/ word_embeddings_output/

##### Preprocessing INEX data

The preprocessing consists of parsing the Wikipedia XML, tokenizing the text with NLTK `tokenizers/punkt/english.pickle`, replacing some utf-8 characters, normalizing the amounts of spaces, etc.

Also, we want make use of the structure in Wikipedia, in particular the paragraph bounds, by only considering sentences to be positive examples if they are next to eachother AND are within the same paragraph.
Therefore, when we parse the Wikipedia XML, we keep track of paragraphg bounds.
We assume that by doing things this way, more semantic coherence is captured.
The downside is, however, that there are many 1- and 2-sentence paragraphs, all of which are ignored. 

Preprocessing is done by running the following script:

    $ cd /path/to/inexutils
    $ ./tokenizeInexFiles.sh /path/to/INEXQAcorpus2013 "*" \
       /path/to/inex_paragraph_output_directory 5 -paragraph_mode

The penultimate argument (5 in the example above) controls the number of processes to run in parallel.<br>
For a further explanation of arguments, run `tokenizeInexFiles.sh` without any arguments.

The INEX Wikipedia dump comes as one file per Wikipedia page.
All these files are stored in 1000 directories.
The original directory structure of the Wikipedia dump is maintained in the output directory.

##### Generating INEX vocabulary 

NOTE that you should __run this on the pre-processed files__ (see step above).

We first make a vocabulary for every __pre-processed__ file:

    $ ./makeAllVocabs.sh 5 /path/to/inex_paragraph_output_directory /path/to/dir_with_all_vocabularies

Agian, the 5 indicates that 5 processes will (at max) be running simultaneously.

We now have one directory with very many vocabulary file.
Now we can merge all of these in one big file:

    $ python combineVocabs.py /path/to/dir_with_all_vocabularies INEX.vocab.txt

#### Toronto Book Corpus

The Toronto Book Corpus contains 74,004,228 already pre-processed sentences in total, which are made up of 1,057,070,918 tokens, originating from 7,087 unique books (_Aligning books and movies: Towards story-like visual explanations by watching movies and reading books._, Zhu et al., 2015).

The corpus can be downloaded from http://www.cs.toronto.edu/~mbweb/.

As no paragraphs boundaries are present in this corpus, we simply treat all triples of consecutive sentences as positive examples (NOTE that these may even cross book boundaries, though very rarely of course).

For efficient handling of the Toronto Book Corpus (and to get rid of some UTF-8 encoding issues), some pre-processing needs to be done (see below)).
If we assume the pre-processing is done, and the vocabulary has been generated and is stored in `toBoCo.vocab.txt`, we can call:

    $ THEANO_FLAGS=floatX=float32 python siamese-cbow.py -v -vocab toBoCo.vocab.txt \
     -max_nr_of_vocab_words 315643 -share_weights -epochs 1 -batch_size 100 \
     -neg 2 /path/to/toBoCo_preprocessed_dir word_embeddings_output/

The vocabulary size here is chosen to allow only for words appearing 5 times or more.

##### Preprocessing Toronto Book Corpus

The Toronto Book Corpus comes in two files: `books_large_p1.txt` and `books_large_p2.txt`.
The first one contains some utf8 characters that were of no importance, but which caused trouble.
To get rid of them, run:

    $ cd /path/to/torontobookcorpusutils
    $ python replaceWeirdChar.py /path/to/books_large_p1.txt > /path/to/books_large_p1.corrected.txt

As we need random access to these rather large files, we first compute the character offsets of every sentence in them (the files come as one sentence per line).
These character offsets enable us to extract random sentences from the corpus in an efficient way.
They are assumed to be stored in two .pickle files.
To produce these files, in the torontobookcorpusutils directory, run:

    $ python file2positions.py /path/to/toronto_book_corpus/books_large_p1.corrected.txt \
       /path/to/toronto_book_corpus/
    $ python file2positions.py /path/to/toronto_book_corpus/books_large_p2.txt \
       /path/to/toronto_book_corpus
 
Please note that, to keep things simple, the software expects the .txt files and the .pickle files to be in the same directory (as reflected in the example above).

##### Generating Toronto Book Corpus vocabulary

To generate a vocabulary file of the entire Toronto Book Corpus, run the following scripts:

    $ python makeVocab.py /path/to/toronto_book_corpus/books_large_p1.corrected.txt > \
       books_large_p1.corrected.vocab.txt
    $ python makeVocab.py /path/to/toronto_book_corpus/books_large_p2.txt > \
       books_large_p2.vocab.txt
    $ python ../inexutils/combineCountWords.py > toBoCo.vocab.txt
