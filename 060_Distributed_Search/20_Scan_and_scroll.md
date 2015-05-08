##扫描和滚屏

`scan（扫描）`搜索类型是和`scroll（滚屏）`API一起使用来从Elasticsearch里高效地取回巨大数量的结果而不需要付出深分页的代价。

___scroll（滚屏）___

一个滚屏搜索允许我们做一个初始阶段搜索并且持续批量从Elasticsearch里拉取结果直到没有结果剩下。这有点像传统数据库里的_cursors（游标）_。

滚屏搜索会及时制作快照。这个快照不会包含任何在初始阶段搜索请求后对index做的修改。它通过将旧的数据文件保存在手边，所以可以保护index的样子看起来像搜索开始时的样子。



___scan（扫描）___

深度分页代价最高的部分是对结果的全局排序，但如果禁用排序，就能以很低的代价获得全部返回结果。为达成这个目的，可以采用`scan（扫描）`搜索模式。扫描模式让Elasticsearch不排序，只要分片里还有结果可以返回，就返回一批结果。

为了使用_scan-and-scroll（扫描和滚屏）_，需要执行一个搜索请求，将`search_type` 设置成`scan`，并且传递一个`scroll`参数来告诉Elasticsearch滚屏应该持续多长时间。


``` js
GET /old_index/_search?search_type=scan&scroll=1m (1)
{
    "query": { "match_all": {}},
    "size":  1000
}
```
（1）保持滚屏开启1分钟。

这个请求的应答没有包含任何命中的结果，但是包含了一个Base-64编码的`_scroll_id（滚屏id）`字符串。现在我们可以将`_scroll_id` 传递给`_search/scroll`末端来获取第一批结果：

``` js
GET /_search/scroll?scroll=1m      (1)
c2Nhbjs1OzExODpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzExOTpRNV9aY1VyUVM4U0 <2>
NMd2pjWlJ3YWlBOzExNjpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzExNzpRNV9aY1Vy
UVM4U0NMd2pjWlJ3YWlBOzEyMDpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzE7dG90YW
xfaGl0czoxOw==
```
--------------------------------------------------
(1) 保持滚屏开启另一分钟。

(2) `_scroll_id` 可以在body或者URL里传递，也可以被当做查询参数传递。

注意，要再次指定`?scroll=1m`。滚屏的终止时间会在我们每次执行滚屏请求时刷新，所以他只需要给我们足够的时间来处理当前批次的结果而不是所有的匹配查询的document。

这个滚屏请求的应答包含了第一批次的结果。虽然指定了一个1000的`size` ，但是获得了更多的document。当扫描时，`size`被应用到每一个分片上，所以我们在每个批次里最多或获得`size * number_of_primary_shards（size*主分片数）`个document。

> ####注意：

> 滚屏请求也会返回一个_新_的`_scroll_id`。每次做下一个滚屏请求时，必须传递前一次请求返回的`_scroll_id`。

如果没有更多的命中结果返回，就处理完了所有的命中匹配的document。

> ####提示：

> 一些[Elasticsearch官方客户端](http://www.elasticsearch.org/guide)提供_扫描和滚屏_的小助手。小助手提供了一个对这个功能的简单封装。


<!--
[[scan-scroll]]
=== scan and scroll

The `scan` search type and the `scroll` API((("scroll API", "scan and scroll"))) are used together to retrieve
large numbers of documents from Elasticsearch efficiently, without paying the
penalty of deep pagination.

`scroll`::
+
--
A _scrolled search_ allows us to((("scrolled search"))) do an initial search and to keep pulling
batches of results from Elasticsearch until there are no more results left.
It's a bit like a _cursor_ in ((("cursors")))a traditional database.

A scrolled search takes a snapshot in time. It doesn't see any changes that
are made to the index after the initial search request has been made. It does
this by keeping the old data files around, so that it can preserve its ``view''
on what the index looked like at the time it started.

--

`scan`::

The costly part of deep pagination is the global sorting of results, but if we
disable sorting, then we can return all documents quite cheaply. To do this, we
use the `scan` search type.((("scan search type"))) Scan instructs Elasticsearch to do no sorting, but
to just return the next batch of results from every shard that still has
results to return.

To use _scan-and-scroll_, we execute a search((("scan-and-scroll"))) request setting `search_type` to((("search_type", "scan and scroll")))
`scan`, and passing a `scroll` parameter telling Elasticsearch how long it
should keep the scroll open:

[source,js]
--------------------------------------------------
GET /old_index/_search?search_type=scan&scroll=1m <1>
{
    "query": { "match_all": {}},
    "size":  1000
}
--------------------------------------------------
<1> Keep the scroll open for 1 minute.

The response to this request doesn't include any hits, but does include a
`_scroll_id`, which is a long Base-64 encoded((("scroll_id"))) string. Now we can pass the
`_scroll_id` to the `_search/scroll` endpoint to retrieve the first batch of
results:

[source,js]
--------------------------------------------------
GET /_search/scroll?scroll=1m <1>
c2Nhbjs1OzExODpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzExOTpRNV9aY1VyUVM4U0 <2>
NMd2pjWlJ3YWlBOzExNjpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzExNzpRNV9aY1Vy
UVM4U0NMd2pjWlJ3YWlBOzEyMDpRNV9aY1VyUVM4U0NMd2pjWlJ3YWlBOzE7dG90YW
xfaGl0czoxOw==
--------------------------------------------------
<1> Keep the scroll open for another minute.
<2> The `_scroll_id` can be passed in the body, in the URL, or as a
    query parameter.

Note that we again specify `?scroll=1m`.  The scroll expiry time is refreshed
every time we run a scroll request, so it needs to give us only enough time
to process the current batch of results, not all of the documents that match
the query.

The response to this scroll request includes the first batch of results.
Although we specified a `size` of 1,000, we get back many more
documents.((("size parameter", "in scanning")))  When scanning, the `size` is applied to each shard, so you will
get back a maximum of `size * number_of_primary_shards` documents in each
batch.

NOTE: The scroll request also returns  a _new_ `_scroll_id`.  Every time
we make the next scroll request, we must pass the `_scroll_id` returned by the
_previous_ scroll request.

When no more hits are returned, we have processed all matching documents.

TIP: Some of the http://www.elasticsearch.org/guide[official Elasticsearch clients]
provide _scan-and-scroll_ helpers that provide an easy wrapper around this
functionality.((("clients", "providing scan-and-scroll helpers")))

-->
