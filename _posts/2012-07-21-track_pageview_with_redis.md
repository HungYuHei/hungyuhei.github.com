---
layout: post
title: 利用 Redis 记录 PageView 并实时查询数据
---

如果只是网站管理员，对于 PageView 的跟踪管理使用 Google Analytics 已经足够强大了，完全没有必要花人力实现另一套工具出来。

不过本文的需求是基于：为用户提供跟踪其发布的某个页面的 PageView (PV) 功能，并可以实时地按小时、日、周、月查询数据。如果使用关系数据库（如 PostgreSQL），为了实现前面提到的查询条件，必须编写复杂的 SQL 查询语句，复杂的 SQL 查询语句意味着需要消耗服务器一定的计算量，效果不够好，最后经过推敲，决定使用 Redis 来实现，理由如下：

* 内存数据库，高性能
* key-value 数据库，基于时间日期来构建 key，能够非常方便实现复杂查询
* 支持复杂的数据特性，比如 Hash, List, Set 等

<br>

## 最终效果

文字是抽象的，程序更是抽象的，所以先放出最终实现出来的效果：

![最终效果](http://ww4.sinaimg.cn/large/a74ecc4cjw1dv2nwni798j.jpg)

## 数据建模

即便是 NoSQL 也和关系数据库差不多，就是首先要建立好数据模型，不过对于 key-value 数据库有点不同的是，建模最重要一个方面就是决定好方便查询的 key。具体到 Redis，除了传统的 key-value 结构，它还支持 Hash, List, Set 等数据结构，所以也需要考虑合适的数据结构。

接上面提到的功能要求是：「按时、日、周、月查询某个页面的 PV」，所以 key 是必须包含 **页面 ID** 和**日期**，所以 key 应该大致是这样：`pageviews:post:ID:DATETIME`，比如我要查询 id 为 12 的 post 在 2012-07-18 下午3点的数据（redis 命令）：

    redis> GET pageviews:post:12:2012-07-18:15
    1763
    
非常直观的实现方式，完全摆脱了复杂的 SQL 语句。

其实就这样使用最基础的 key-value 结构已经实现出来了，不过根据 Redis 官网给出的这篇[内存优化指南](http://redis.io/topics/memory-optimization)中提到的「Use hashes when possible」，还是决定使用 Hash 来保存数据，而且使用 Hash 的还有另一好处是更好组织数据。

按照最初方案，很容易就可以改造成使用 Hash 的方案。Hash 其实就是在一个 key 中可以储存多个 field，具体实现起来无非就是将原来 key 中的日期标识改成 field 就可以了，伪代码大致是这样：

    "pageviews:post:12" => { '2012-07-18:15' => 357 }
    
另需要注意考虑的一点是数据储存的颗粒度大小，因为最精确的查询要求是按小时，所以按小时来区分数据是必需的，至于按日、按周、按月等，是单独储存数据还是通过基础数据计算出来（空间换时间 or 时间换空间），需要按自己的实际情况来考虑。这里的实现是按小时和日期来储存数据，月和周通过日期的数据计算出来。例如：**id 为 12 的 post 在 2012-07-18 整天和15点、21点的** 数据大致的数据会是这样：

    "pageviews:post:12" => { '2012-07-18' => 1731, '2012-07-18:15' => 357, '2012-07-18:21' => 241 }
    
<br>

## 实现（基于 Rails 3.1+）

首先安装 `redis` 这个 Gem 来实现通过 Ruby 操作 Redis  
在 Gemfile 中添加： `gem 'redis', '~> 3.0.1'`，配置请参考 [https://github.com/redis/redis-rb](https://github.com/redis/redis-rb)

### Model

    class PageView
      BASE_KEY = 'pageviews:post:'

      attr_reader :date, :datetime, :key

      def initialize(post_id)
        now = Time.current
        @date = now.strftime('%F') # => 2012-07-18
        @datetime = now.strftime('%F:%H') # => 2012-07-18:15
        @key = BASE_KEY + post_id.to_s # => pageviews:post:12
      end
    end
    
马上通过 `Rails c` 进行冒烟测试：

    1.9.3p194 :001 > pv = PageView.new(12)
    1.9.3p194 :002 > pv.key
     => "pageviews:post:12"
    1.9.3p194 :003 > pv.date
     => "2012-07-18" 
    1.9.3p194 :004 > pv.datetime
     => "2012-07-18:02"
     
`PageView#key` 就是对应的 `redis 的 key`，`PageView#date` 和 `PageView#datetime` 就是对应的 `redis 的 field`

### Controller

在 Controller 实现一个 `after_filter`， 只要运行 `show` 这个 action 就将对应的 PV 值增加：

    class PostsController < ApplicationController
      after_filter :increase_pv, :only => [:show]

      private
        def increase_pv
          PageView.new(@ppost.id).incr
        end
    end

然后在 `PageView` Model 上实现 `incr` 这个方法：

    def incr(increment = 1)
      $redis.multi do
        $redis.hincrby(key, date, increment)
        $redis.hincrby(key, datetime, increment)
      end
    end
    
`incr` 这个方法实际上是将当天和当前小时的 PV 值都加 1  
另外留意一下 `$redis.multi`，实际上 Redis 也有类似关系数据事务的概念。reference: [http://redis.io/topics/transactions](http://redis.io/topics/transactions)

### 查询数据
 
到此为止，已经实现了当用户访问指定页面就会将增加对应的 PV 值，接下来就是实现查询数据，以实现查询日期为例子。
通过 `redis` 这个 gem 扩展的 `mapped_hmget` 方法可以很方便地查询出数据（[详细 API 说明](http://rubydoc.info/github/redis/redis-rb/Redis#mapped_hmget-instance_method)）。
回到 `PageView` Model，实现 `fetch` 方法查询指定日期范围的 PV 值
 
    def fetch(begin_at, end_at)
      $redis.mapped_hmget(key, *date_list(begin_at, end_at))
    end

    private
      def date_list(begin_at, end_at)
        begin_at.upto(end_at).reduce([]) { |days, day| days << day.to_s }
      end

马上通过 `Rails c` 进行冒烟测试
    
    1.9.3p194 :028 >   now = Time.current; begin_date = (now - 3.days).to_date; end_date = now.to_date
     => Fri, 20 Jul 2012 
    1.9.3p194 :029 > pv = PageView.new(1210).fetch(begin_date, end_date)
     => {"2012-07-17"=>"3", "2012-07-18"=>"1", "2012-07-19"=>"6", "2012-07-20"=>"8"}
     
查看执行结果 ` {"2012-07-17"=>"3", "2012-07-18"=>"1", "2012-07-19"=>"6", "2012-07-20"=>"8"}` 可以得到我们想要的日期和对应的 PV 值。

下一步创建 `PageViewsController`，在命令行执行 `rails g controller page_views show`，编辑 PageViewsController：

    class PageViewsController < ApplicationController
      def show
        now = Time.current
        begin_date = (now - 6.days).to_date
        end_date = now.to_date

        @page_views = Redis::PageView.new(@post.id).fetch(begin_date, end_date)
      end
    end

文章开始的那个最终效果图需要依赖 `Morris.js` 这个 JavaScript 库来生成，具体参考：[http://oesmith.github.com/morris.js/](http://oesmith.github.com/morris.js/)

按照 `Morris.js` 对数据格式的要求，我们需要修改一下 `PageView` Model 返回的数据，将原来的 `fetch` 方法修改成如下：

    def fetch(begin_date, end_date)
      days = date_list(begin_date, end_date)

      $redis.hmget(key, *days) do |values|
        if values.kind_of?(Array)
          days.zip(values).reduce([]) do |array, item|
            array << { :k => item[0], :v => (item[1] || 0).to_i }
          end
        else
          values
        end
      end
    end
    
修改后，`fetch` 方法将会返回类似这个的数据 `[{"k":"2012-07-19","v":6},{"k":"2012-07-20","v":8}]`。
到此为止，`MVC` 结构中的 `M` 和 `V` 都实现得差不多了，接下来就是实现 `V`，即 `View`

### View

先准备好  `Morris.js` 以及其依赖的文件，然后新建一个 `page_views.js`，并加入以下代码：

    //= require jquery
    //= require raphael
    //= require morris
    //= require prettify

接着新建 view 文件 `app/views/page_views/show.html.erb`，并编辑其内容：

    <div id="pv"></div>
    <%= javascript_include_tag "page_views" %>
    <script type="text/javascript">
      $(function () {
        Morris.Line({
          element: 'pv',
          data: #{@page_views},
          xkey: 'k',
          ykeys: ['v'],
          ymin: 'auto[0]',
          ymax: 'auto[8]',
          labels: ['浏览量'],
          parseTime: false,
          lineWidth: 2
        });
      });        
      prettyPrint();
    </script>
    
<br>

## Last but not least...

如果还有进一步的需求，例如唯一身份访问者等，配合 Redis 的 Set，实现起来也不是难事。

如果你想基于关系数据库来实现，可以考虑一下使用 `impressionist` 这个 Gem [https://github.com/charlotte-ruby/impressionist](https://github.com/charlotte-ruby/impressionist)

除了本文实现的功能外，我在项目中也将 Redis 用于实现排行榜、SNS 关系、缓存等。相比较关系数据库，通过 Redis 的一些特性，可以更方便地实现也这些功能。往后时间，我将会整理出文章。

以 Redis、Memcached (key-value)  为代表的 NoSQL 与关系数据库其实不应该是谁取替谁的关系，相反，NoSQL 应该是关系数据库的一个很好的补充，尤其在高并发、低一致性需求 Web 领域。
