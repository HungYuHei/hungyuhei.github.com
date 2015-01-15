---
layout: post
title: HTTP 缓存知识点整理
---

> There are only two hard things in Computer Science: cache invalidation and naming things.

前段时间在优化 HTTP 缓存策略，以减轻一下 API 的压力。这段时间空闲一些就做个简单整理归纳。参考资料主要是 O'REILLY 的《HTTP 权威指南》和网上的一些文章。

# 缓存控制策略

服务器可以 [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expires) 和 [`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) 两个 Header 来设置缓存过期时间。

`Expires` 比较简单，就只能设置一个绝对日期，而后来发布的 `Cache-Control` 自然更强大一些，除了能指定缓存有效期之外还能更进一步指定缓存控制策略。

正因为 `Expires` 过于简单，所以它的局限也比较大。例如因为其使用绝对日期，所以必须保证浏览器的时钟跟服务器的时钟同步，《HTTP 权威指南》更倾向于使用 `Cache-Control` 也是基于这个理由。

不过 `Expires` 特别适用于设置静态文件缓存，因为通常这些文件很少有变化。

另外，当 `Cache-Control` 设置了 max-age 值时，`Expires` 会被忽略。

# 缓存再验证 (Revalidate)

通过条件请求（Conditional GET）可以实现缓存再验证。HTTP 定义了 5 个条件请求 Header，对于缓存再验证来说最有用的 2 个是 [`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since) 和 [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) 。

##If-Modified-Since

`If-Modified-Since` 可以与服务器返回的 [`Last-Modified`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified) 配合工作。

当请求中带有 `If-Modified-Since` 时，如果服务器会通过返回 `200 OK` 来新的响应内容或者返回 `304 Not Modified` 来表示缓存未过期。

## If-None-Match

有些情况下仅使用最后修改日期进行再验证是不够的。

* 有些文档可能会被周期性地重写（比如，从一个后台进程中写入），但实际包含的数据常常是一样的。 尽管内容没有变化，但修改日期会发生变化。

* 有些文档可能被修改了，但所做修改并不重要，不需要让大范围内的缓存都重装数据（比如对拼写或注释的修改）
* 有些服务器无法准确地判定其页面的最后修改日期。

* 有些服务器提供的文档会在毫秒间隙发生变化（比如，实时监视器），对这些服务器来说，以一秒为粒度的修改日期可能就不够用了。

所以 HTTP 引入 [`ETag`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag) 。

 `ETag` 可能包含响应内容的序列号或版本号，或者是它的校验和指纹信息。`Last-Modified` 精确度比 `ETag` 低。

当请求中带有 [`If-None-Match`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match) 时，如果服务器会通过返回 `200 OK` 来新的响应内容或者返回 `304 Not Modified` 来表示缓存未过期。

`If-Modified-Since` 当与 `If-None-Match` 一同出现时，`If-Modified-Since` 会被忽略掉。

# 小结

**浏览器**通过 `Expires` 和 `Cache-Control` 来判断是从缓存中读数据，还是发出 HTTP 请求读数据，好处是可以减少 HTTP 请求数。

在后续的缓存再验证时，**服务器**根据 `If-Modified-Since` 和 `If-None-Match` 来判断是返回 200 还是 304 状态，好处只是减少网络传输量，但不能减少 HTTP 请求数。
