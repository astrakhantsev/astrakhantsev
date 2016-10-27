---
title: 'Java/Scala tools for working with word2vec models'
permalink: /java-scala-word2vec-libs/
categories:
  - Tools
  - NLP
---

Since the invention of word2vec in 2013,
it managed to go all the way in NLP from funny toy (_hey, dollar+Russia-USA=63 already!_)
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
The first one in google for "word2vec java" is [deeplearning4j](https://deeplearning4j.org):
it is huge, requires a lot of stuff,
so we'd prefer to start with something more lightweight (but we'll return to it in the next post).

The next one is [Word2VecJava](https://github.com/medallia/Word2VecJava)
What catches our eyes? They use `double[]` for vectors storage; despite the fact that binary format stores floats, so there would be no information loss. And all potential loss in accuracy caused by operations under vectors like cosine in case of float instead of doubles would be negligible compared to the loss caused by twice lower dimension of the vector (on condition of the same memory consumption).

Thus we have 1 more requirement for a lib: float vectors. Moreover, as float arrays, not Lists, because, we all know, Float weights as double double (sorry for bad wordplay).

The third hit leads to [Apache Spark](http://spark.apache.org/docs/latest/ml-features.html#word2vec). 
Again, too heavy, plus they don't have sim/cos;
it is not hard to implement, but what was the reason not to create it?
But worse, it still can't load pre-trained models of gensim or original word2vec
(see [old issue](https://issues.apache.org/jira/browse/SPARK-9484) unresolved yet).

There are also some abandoned non-English [repos](https://github.com/NLPchina/Word2VEC_java)
with strange code in Readme file.

Not so much. Let's google for Scala.

## Scala libraries
[read_word_vectors.scala](https://github.com/awhogue/word2vec-scala/blob/master/read_word_vectors.scala) -
ok, it is licensed under Apache 2.0, utterly simple one-file with no dependency.
But it is based on Scala's `List`, which compiles to Java's `List` (see corresponding [SO answer](http://stackoverflow.com/questions/2712877/difference-between-array-and-list-in-scala)
It doesn't contain sim/cos and hasn't been updated since the very 2013.

[word2vec-scala](https://github.com/trananh/word2vec-scala) - Apache License, Array of float, one-file with no dependency; sim/cos implemented.
The only minor not that it hasn't been updated since the very 2013.
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

If we look at the distance between 2 words ` ID/united_states` and ` ID/world_war_ii`, there are 400 bytes, thus there is no delimiter in new model.
Btw, it proves that each element weights 4 bytes, i.e. float.

It is not a bug. If we look at the old model, it has a delimiter (and then, we have support for old models in dl4j, see below).
Obviously, the model format has changed; I don't know, who introduced that - maybe, gensim.

Of course, we can determine model format automatically, because we know word vectors size (it is written in the model file prefix),
but it is much simpler to set it up in parameter and, most probably, no one would use old models.

So this fix was implemented (and merged).
Btw, dl4j uses the same [workaround](https://github.com/deeplearning4j/deeplearning4j/blob/91a481ae8f5bcb4c9ff3463c1bba2df69d7325d2/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/models/embeddings/loader/WordVectorSerializer.java#L114).


Next time we'll compare this lib with DL4J and check if the burden of dl4j dependencies is worth it.
