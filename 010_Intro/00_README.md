# 入门

Elasticsearch是一个实时分布式搜索和分析引擎。他让你可以以前所未有的速度处理大数据成为可能。

它用于全文搜索、结构化搜索、分析以及将这三者合并：

* 维基百科使用Elasticsearch提供全文搜索并高亮关键字，并提供**输入即时搜寻(search-as-you-type)**和**搜索纠错(did-you-mean)**等搜索建议功能。

* 英国卫报使用Elasticsearch综合用户日志和社交数据提供实时的反馈给他们的编辑，以便及时获得公众反馈。

* StackOverflow将全文搜索与地理位置和相关信息进行结合，以提供**more-like-this**功能来找到相关问题的答案。

* Github使用Elasticsearch检索1300亿行的代码。

But Elasticsearch is not just for mega-corporations. It has enabled many
startups like DataDog and Klout to prototype ideas and to turn them into
scalable solutions. Elasticsearch can run on your laptop, or scale out to
hundreds of servers and petabytes of data.

No individual part of Elasticsearch is new or revolutionary. Full text search
has been done before, as have analytics systems and distributed databases. The
revolution is the combination of these individually useful parts into a
single, coherent, real-time application. It has a low barrier to entry for the
new user, but can keep pace with you as your skills and needs grow.

If you are picking up this book, it is because you have data, and there is no
point in having data unless you plan to *do something* with it.

Unfortunately, most databases are astonishingly inept at extracting actionable
knowledge from your data. Sure, they can filter by timestamp or exact values,
but can they perform full-text search, handle synonyms and score documents by
relevance?  Can they generate analytics and aggregations from the same data?
Most importantly, can they do this in real-time without big batch processing
jobs?

This is what sets Elasticsearch apart: Elasticsearch encourages you to explore
and utilize your data, rather than letting it rot in a warehouse because it is
too difficult to query.

Elasticsearch is your new best friend.
--
