---
layout: post
title: Rails CSRF 兼容 Angular
---

早期在实现前后端分离时使用了 Angular，那里前后端分离才刚刚兴起。
大概先吃螃蟹的坏处之一就是坑多，例如 Angular 跟 Rails 的 CSRF 保护机制冲突。

刚好在 Stack Overflow 看到有人问类似问题，于是我就顺手回答了，
想不到一段时间过去后，得到了不少赞同，得瑟一下，哈

[https://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on/15761835](https://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on/15761835#15761835)

更有趣的是有人根据我的回答写了一个 Gem，哈哈，真好玩

[https://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on/20572280#20572280](https://stackoverflow.com/questions/14734243/rails-csrf-protection-angular-js-protect-from-forgery-makes-me-to-log-out-on/20572280#20572280)

github 上也有相关讨论

[https://github.com/rails/rails/pull/9405](https://github.com/rails/rails/pull/9405)

