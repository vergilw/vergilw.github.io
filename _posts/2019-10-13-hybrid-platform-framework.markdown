---
layout: post
title: ReactNative and Flutter
date: 2019-10-13
---

如今跨平台框架日益完善，在今年经济形势不好的情况下，企业为了缩减开发成本也越来越多的采用移动端跨平台解决方案，了解跨平台解放方案的对企业技术选型尤为重要。

网上对于跨平台方案的技术比较文章很多，本文对于跨平台方案的讨论，仅从公司成本和产品层面出发，抛开了纯技术的问题。

如下最热门的跨平台技术莫属ReactNative和Flutter，Facebook的ReactNative先于Flutter发布几年，生态较成熟，正如RN官网所介绍的，保证了和原生App一致的交互体验，但用过RN的都知道，正因为RN的实现技术，将JS代码转成原生平台代码，对于大型App的特殊业务需求往往没有完美的兼容方案，这就导致使用RN框架最终变成了需要写三端代码。这时Google发布了Flutter跨平台方案，完美的解决了UI兼容性的问题，大量的RN开发者拥护Flutter是真正未来的跨平台解决方案，因为他们确实受够了RN的开发三端问题。但是正由于Flutter的实现方案，自有的绘制引擎，完全脱离了原生平台的渲染框架，导致了Flutter的UI体验差强人意，虽然性能上有所保障，但是原生平台框架十多年的发展，不是一个新生的Flutter框架能短期追赶的上的，正如那句话所说的，不是Flutter太弱，只是原生平台框架太强。特别是iOS系统对UI交互体验的优化细节，对于长期使用原生系统App的用户来说，一定会感觉到明显差异，这个交互的差异对于C端的产品是很难接受的。Flutter框架目前的发展还在解决Bug阶段，更不谈追赶原生平台的体验了。再加上Flutter生态不够成熟，很多技术都需要重新造轮子，这对于小团队来说成本太高。

所以，如果当下如果公司是为了降低开发成本，提高开发效率，首选还是ReactNative。除非你的团队不在乎时间成本或者人力成本，并且产品的交互设计允许放低标准，那可以尝试为Flutter未来的发展当当铺路石。但并不是说Flutter框架不好，至少目前短期来说，Flutter的发展还有很长的路要走。

