#让文本可搜索

第一个不得不解决的挑战是如何让文本变得可搜索。在传统的数据库中，一个字段存一个值，但是这对于全文搜索是不足的。想要让文本中的每个单词都可以被搜索，这意味这数据库需要存多个值。

支持一个字段多个值的最佳数据结构是反向索引，我们在。反向索引包含了出现在所有文档中唯一的值或词的有序列表，以及每个词所属的文档列表。


     Term  | Doc 1 | Doc 2 | Doc 3 | ...
     ------------------------------------
     brown |   X   |       |  X    | ...
     fox   |   X   |   X   |  X    | ...
     quick |   X   |   X   |       | ...
     the   |   X   |       |  X    | ...


[NOTE]
====
When discussing inverted indices, we talk about indexing _documents_ because,
historically, an inverted index was used to index whole unstructured text
documents.  A _document_ in Elasticsearch is a structured JSON document with
fields and values.  In reality, every indexed field in a JSON document has its
own inverted index.
====

The inverted index may hold a lot more information than the list
of documents that contain a particular term. It may store a count of the number of
documents that contain each term, the number of times a term appears in a particular
document, the order of terms in each document, the length of each document,
the average length of all documents, and more.  These statistics allow
Elasticsearch to determine which terms are more important than others, and
which documents are more important than others, as described in
<<relevance-intro>>.

The important thing to realize is that the inverted index needs to know about
_all_ documents in the collection in order for it to function as intended.

In the early days of full-text search, one big inverted index was built for
the entire document collection and written to disk.  As soon as the new index
was ready, it replaced the old index, and recent changes became searchable.

[role="pagebreak-before"]
==== Immutability

The inverted index that is written to disk is _immutable_: it doesn't
change.((("inverted index", "immutability"))) Ever.  This immutability has important benefits:

* There is no need for locking. If you never have to update the index, you
  never have to worry about multiple processes trying to make changes at
  the same time.

* Once the index has been read into the kernel's filesystem cache, it stays
  there, because it never changes.  As long as there is enough space in the
  filesystem cache, most reads will come from memory instead of having to
  hit disk.  This provides a big performance boost.

* Any other caches (like the filter cache) remain valid for the life of the
  index. They don't need to be rebuilt every time the data changes,
  because the data doesn't change.

* Writing a single large inverted index allows the data to be compressed,
  reducing costly disk I/O and the amount of RAM needed to cache the index.

Of course, an immutable index has its downsides too, primarily the fact that
it is immutable! You can't change it.  If you want to make new documents
searchable, you have to rebuild the entire index. This places a significant limitation either on the amount of data that an index can contain, or the frequency with which the index can be updated.


