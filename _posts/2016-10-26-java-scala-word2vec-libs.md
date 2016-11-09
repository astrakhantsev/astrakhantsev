---
title: 'Java/Scala tools for working with word2vec models'
excerpt: "How to choose word2vec library for JVM language if you don't want to learn new models, but need to read already learned vectors and compute similarity."
permalink: /java-scala-word2vec-libs/
tags:
  - Tools
  - NLP
---

Since the invention of word2vec in 2013,
it managed to go all the way in NLP from funny toy (_hey, Putin+USA-Russia=Trump already!_)
to silver bullet (_look, we can find semantically similar words by word2vec and solve the whole task!_)
 and, finally, to mainstream (_ok, have you already tried word2vec for this task?_).

Word2vec becomes especially helpful,
when we work with small text data and face sparseness problem in its worst.
A popular way to cope with it is to train word2vec model on some huge data like Wikipedia dump
and use these good vectors for words of texts we actually want to process.

In this setting, we don't need to train model in our program or even ourselves;
there are [collections](https://github.com/3Top/word2vec-api#where-to-get-a-pretrained-models)
 of pre-trained models ready to download).
All we have to do is to load pre-trained model into some map
(dictionary, in case of Python) and to compute cosines between vectors.

~~Ok, we can write such code in an hour!~~
Ok, there should be a library doing exactly that. ~~Let's spend this hour choosing it!~~

Let's assume we write our app in some JVM language like Java or Scala
(in Python: simply use genism;
in C/C++: find original word2vec saved by [some guy in github](https://github.com/3Top/word2vec-api#where-to-get-a-pretrained-models)
after the shutdown of code.google).

As usual, implied requirements to the lib:
permissive license, little dependencies, readable code.

So, what are our options?

## Java libraries
The first one in google for "word2vec java" is [deeplearning4j](https://deeplearning4j.org) -
it is huge, requires a lot of stuff,
so we'd prefer to start with something more lightweight (but we'll return to it in the next post).

The next one is [Word2VecJava](https://github.com/medallia/Word2VecJava)

What catches our eyes? They use `double[]` for vectors storage; despite the fact that binary format stores floats, so there would be no information loss. 
And all potential loss in accuracy caused by operations under vectors of floats instead of vectors of doubles would be negligible compared to the loss caused by twice lower dimension of the vector (on condition of the same memory consumption).

Thus we have 1 more requirement for a lib: float vectors. Moreover, as float arrays, not Lists, because, we all know, Float weights as double double (sorry for bad wordplay).

The third hit leads to [Apache Spark](http://spark.apache.org/docs/latest/ml-features.html#word2vec). 

Again, too heavy, plus they don't have sim/cos;
it is not hard to implement, but what was the reason not to create it?
But worse, it still can't load pre-trained models of gensim or original word2vec,
see [old issue](https://issues.apache.org/jira/browse/SPARK-9484) unresolved yet.

There are also some abandoned non-English [repos](https://github.com/NLPchina/Word2VEC_java)
with strange code in Readme file.

Not so much. Let's google for Scala.

## Scala libraries
[read_word_vectors.scala](https://github.com/awhogue/word2vec-scala/blob/master/read_word_vectors.scala) -
ok, it is licensed under Apache 2.0, utterly simple one-file with no dependency.

But it is based on Scala's `List`, which compiles to Java's `List`, see corresponding [SO answer](http://stackoverflow.com/questions/2712877/difference-between-array-and-list-in-scala)

It doesn't matter already, but this lib doesn't contain sim/cos and hasn't been updated since the very 2013.

[word2vec-scala](https://github.com/trananh/word2vec-scala) - Apache License, Array of float, one-file with no dependency; sim/cos implemented.
The only minor note that it hasn't been updated since the very 2013.

Luckily, [Refefer](https://github.com/Refefer) updated it:
split into 2 classes (Reader and Model)
and rewrote Model so that it is based on spire and, hopefully, works faster.

I have chosen this library.

### Small war story

I checked the model provided with the code - unsurprisingly, everything worked.

I took the model trained by modern genism;
tried to load it and it failed with `EOFException` during model file reading.

Then ~~- hour for choosing library was coming to end -~~ I searched for exception message at stackoverflow,
read the [top answer](http://stackoverflow.com/questions/18451232/eofexception-how-to-handle) diagonally (is there such an idiom in English?) -
it advised to simply catch exception.
I copy-pasted this voiceless catch - the main mistake - and everything seemed to work fine.

However, I started to get many misses for completely common words.
So, I took a look at the [source code](https://github.com/Refefer/word2vec-scala/blob/b75b33201a1b073d5e47b6b48837ede905a9e301/src/main/scala/word2vec/Reader.scala#L98) a little more:

{% highlight scala linenos %}
def readVector(reader: VecBinaryReader, vecSize:Int, normalize: Boolean): (String, Array[Float]) = {
     // Read the word
    val word = reader.read[String]

    val vector = new Array[Float](vecSize)
    for((f, i) <- reader.read[Stream[Float]].take(vecSize).zipWithIndex) {
      vector(i) = f
    }

    // Eat up the next delimiter character
    reader.read[Byte]

    // Store the normalized vector representation, keyed by the word
    word -> (if (normalize) normVector(vector) else vector)
}
{% endhighlight %}

This line looks susceptible:

{% highlight scala %}
// Eat up the next delimiter character
reader.read[Byte]
{% endhighlight %}

Let's examine the excerpt from the model file:

```
ID/united_states 2dr?`C?>?{???>?`?>???>o>[?!???d?Y?>?,??q? ???"??%?????>?"=??l?=!	??#F=??$??8|??J(????>X(?p7>v???`????>??=??????>)%??u????>Hb?.?
??=?n????1???=??>yX?>?????1>???>???><?{?s???!-?>U????
>L??>~7?????V?H*>?P?C ?? ????>?bD=*.?/?M?	???>]w????l?:?"?tÑ>?9?Wo???.K>M?'?T??>?>??>w]5?D6??Q??)???U???3>XW;??>e*,?????I???J???Y???>B?C????[
?|W^???>n?????>%Rn>???ID/world_war_ii ???>??mMP=c????>?>???&???????K=?lY?>???K????o?yx?_?l)?b?;?p???>??P??'???:???????=?????Wu???>?K?=???=?u??+??o]=??`>????????d???+Nh?l??m??=??>??k?2?<i?>?(??????J???@???
?'??4?>?WZ???OVG>?#>?{?1r?A?>s3???>?R?>?H??????*?
=?Qc?V??.
>x;[?@5?>?t?U????>1?$?7?(??=@??????c?>??H??>??h??7w>S^y??????-?????????CJ=??{??	????>D???|?>??zK3=?z??d=??ID/association_football ???>???j8Q?ao)?2???>)?|???
```

If we look at the distance between 2 words `ID/united_states` and `ID/world_war_ii`, there are 400 bytes, thus there is no delimiter in new model.
Btw, it proves that each element weighs 4 bytes, i.e. it is float.

It is not a bug. If we look at the old model, it has a delimiter (and then, we have support for old models in dl4j, see below).
Obviously, the model format has changed; I don't know, who introduced that - maybe, gensim.

Of course, we can determine model format automatically, because we know word vectors size (it is written in the model file prefix),
but it is much simpler to set it up in parameter and, most probably, no one would use old models.

So this fix was implemented (and merged).
Btw, dl4j uses the same [workaround](https://github.com/deeplearning4j/deeplearning4j/blob/91a481ae8f5bcb4c9ff3463c1bba2df69d7325d2/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/models/embeddings/loader/WordVectorSerializer.java#L114).

## Compare deeplearning4j with word2vec-scala

Actual task I needed to accomplish with word2vec is to rank term candidates, 
i.e. to compute semantic similarity between some set of words/collocation and some set of keywords.

I performed this task with 2 libs (dl4j and word2vec-scala), 
keeping all other conditions the same, including word2vec model.
I didn't try to make precise evaluation, so there is no averaging of multiple launches, to say nothing about JVM warming-up.

I measured 2 times:

1. Time needed to load word2vec model: word2vec-scala requires about 60-70s, while dl4j -- 150-180s.

2. Time needed to compute semantic similarities.

Table below contains one row per dataset; 
2nd column shows count of similarity computation calls, when word2vec model contains both words;
3rd column shows count of calls for vectors, when word2vec model doesn't contain one word.

| Dataset  | Compute similarity | Only check presence | DL4j  | word2vec-scala |
|----------|--------------------|---------------------|-------|----------------|
| ACL2     |            404,307 |             266,893 |   12s |             1s |
| Patents  |            153,014 |             238,374 |   11s |             7s |
| GENIA    |          1,995,757 |           5,923,282 |   57s |             6s |
| Krapivin |         10,510,830 |          98,928,131 |  4.4m |            14s |
| FAO      |         18,860,475 |         114,602,307 |  7.5m |            15s |
| ACL      |         15,309,896 |         149,369,914 |  6.5m |            20s |
| Europarl |         14,721,164 |          98,334,404 | 11.9m |            12s |

As we can see, dl4j works significantly slower.

This may be caused by several reasons.
Probably, there is some flaw in experiment setup, so -- again -- take these results with a grain of salt.
Probably, dl4j works so slow because of missing native libraries on my machine; 
I tried to install different blas listed in [ND4j guide](http://nd4j.org/getstarted.html),
but it isn't trivial and I didn't invest much time into that.

However, note that deeplearning4j keeps evolving, for example their [new guide](https://deeplearning4j.org/native) looks much nicer;
maybe, Intel MKL, which dl4j recommends to use and which I didn't check, already speeds up this.

In sum, when you read this, dl4j may work faster, but be ready to spend time playing with setup.
Or, if all you need is read model and compute a few dot-products -- use word2vec-scala.
