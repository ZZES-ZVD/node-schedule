# Node Schedule

[![NPM version](http://img.shields.io/npm/v/node-schedule.svg)](https://www.npmjs.com/package/node-schedule)
[![Downloads](https://img.shields.io/npm/dm/node-schedule.svg)](https://www.npmjs.com/package/node-schedule)
[![Build Status](https://travis-ci.org/node-schedule/node-schedule.svg?branch=master)](https://travis-ci.org/node-schedule/node-schedule)
[![Join the chat at https://gitter.im/node-schedule/node-schedule](https://img.shields.io/badge/gitter-chat-green.svg)](https://gitter.im/node-schedule/node-schedule?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![NPM](https://nodei.co/npm/node-schedule.png?downloads=true)](https://nodei.co/npm/node-schedule/)

Node Schedule 是一个Node.js的灵活的类似cron又不类似的任务调度库.它允许你调度任务（任意函数）在特殊的日期执行，并循环执行。他只在在任何给定的时间里使用一个定时器（而不是每隔一秒/一分钟来重新判断将要执行的任务）

## 使用

### 安装

你可以使用 [npm](https://www.npmjs.com/package/node-schedule).

```
npm install node-schedule
```

### 概述

Node Schedule 是一个基于时间的调度，而不是基于区间的调度。你可以很容易的让他按照你的意思来干活，比如，你说“每五分钟来运行这个函数"，你将发现`setInterval`要更容易使用，也是更适合的。但是如果你想说"运行这个函数在每个月的第三个星期二每个小时的20分和50分"，你会发现你更想要Node Schedule组件。此外，Node Schedule 支持windows系统，不像cron并不支持。

注意 Node Schedule 是被设计来进行进程内调度，也就是说调度任务只能在你的脚本运行时才能有效以及调度将在执行成功后消失。如果你需要在你脚步 *不* 运行的时候调度任务，那就需要考虑使用cron.

### 任务和调度

每个在Node Schedule的计划任务都会被一个`Job`对象所代表，你可手动创建任务，然后执行 `schedule()`方法来应用一个计划，或者使用一个方便的方法`ScheduleJob()` 就像下面要说的。

`Job` 对象是 `事件触发器`,触发一个 `run` 事件在每次执行之后。
他们也触发一个`scheduled`事件，在每次他们调度运行的时候，
`canceled`事件可以让一个调用在它执行之前被取消（这两个事件都接受一个JavaScript日期对象作为一个参数). 注意这个任务会第一时间被调度，所以如果你使用 `scheduleJob()`这个方便的方法来创建一个任务，你将错过第一个`scheduled`事件，但是你能手动查询调用（下面会有）。也要注意 `canceled` 是单L美式拼写方法

### Cron风格的调度

cron的格式组成如下:
```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    |
│    │    │    │    │    └ 一周的星期 (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── 月份 (1 - 12)
│    │    │    └────────── 月份中的日子 (1 - 31)
│    │    └─────────────── 小时 (0 - 23)
│    └──────────────────── 分钟 (0 - 59)
└───────────────────────── 秒 (0 - 59, OPTIONAL)
```

cron格式的例子:

```js
var schedule = require('node-schedule');

var j = schedule.scheduleJob('42 * * * *', function(){
  console.log('生命，宇宙，一切的答案。。。!');
});
```

当分钟为42时，执行一个cron任务(例如 19:42, 20:42, etc.).

以及:

```js
var j = schedule.scheduleJob('0 17 ? * 0,4-6', function(){
  console.log('今天被ren认出来了!');
});
```

每五分钟执行一个cron任务 = `*/5 * * * *`

#### 不支持的cron特性

一般的, `W` (最近的工作日), `L` (一个月/星期的最后一天), 以及 `#` (月的第n个星期) 是不支持的. 大多数流行的cron特性应该都能工作。

[cron-parser] 用来解析crontab指令

### 基于日期的调度

就是说你特别想要一个函数在 2012年12月12日早上5:30执行。
记住在JavaScript中- 0 - 星期一, 11 - 十二月.（意思就是星期数和月份数都是从0开始计数的）

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);

var j = schedule.scheduleJob(date, function(){
  console.log('世界将在今天走向 结束.');
});
```

要在未来使用当前数据，你可以使用绑定:

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);
var x = 'Tada!';
var j = schedule.scheduleJob(date, function(y){
  console.log(y);
}.bind(null,x));
x = 'Changing Data';
```
当调度的任务运行时，这个将会打印出'Tada!'，而不是 'Changing Data'，
这个x会在调度后立即更改.

### 递归循环规则调度

你可以创建递归规则来指定任务在何时重新调用。举个例子，考虑这个规则，将在每个小时的第42分钟执行函数:

```js
var schedule = require('node-schedule');

var rule = new schedule.RecurrenceRule();
rule.minute = 42;

var j = schedule.scheduleJob(rule, function(){
  console.log('生命，宇宙，一切的答案。。。!');
});
```

你也可以使用数组来指定一个允许值的列表,`Range`
对象来指定一个系列的开始值和结束值，带有可选的步骤参数。举个例子，这个将在星期4，星期5，星期6和星期天的下午五点答应一个信息：

```js
var rule = new schedule.RecurrenceRule();
rule.dayOfWeek = [0, new schedule.Range(4, 6)];
rule.hour = 17;
rule.minute = 0;

var j = schedule.scheduleJob(rule, function(){
  console.log('今天我碰到klren了!');
});
```

#### 递归规则的属性

- `second`
- `minute`
- `hour`
- `date`
- `month`
- `year`
- `dayOfWeek`

> **注意**: 值得注意的时递归规则的默认的第一个属性是`null` (除了第二个,对于熟悉cron，知道默认为0). *如果我们之前没有明确地设定`minute`为0, 信息将会在下面时间打印 5:00pm, 5:01pm, 5:02pm, ..., 5:59pm.* 或许这不是你想要的.

#### 对象字面化语法

让事情变得简单一点，一个对象字面化语法也是支持的，就像这个例子，将会在每个星期天的下午两点半打印信息：

```js
var j = schedule.scheduleJob({hour: 14, minute: 30, dayOfWeek: 0}, function(){
  console.log('到了喝茶的时间!');
});
```

#### 设置开始时间和结束时间

这个例子中，它将在五秒后开始，然后十秒后结束.和之前一样支持规则。

```js
let startTime = new Date(Date.now() + 5000);
let endTime = new Date(startTime.getTime() + 5000);
var j = schedule.scheduleJob({ start: startTime, end: endTime, rule: '*/1 * * * * *' }, function(){
  console.log('到了喝茶时间!');
});
```

### 处理任务和任务调度

这儿有一些函数来从一个任务中获取信息以及处理任务和调度

#### job.cancel(reshedule)
你可以让任何任务失效，使用 `cancel()` 方法:

```js
j.cancel();
```
所有的计划调用将会被取消。当你设置 ***reschedule*** 参数为true，然后任务将在之后重新排列。

#### job.cancelNext(reshedule)
这个方法将能将能取消下一个计划的调度或者任务.
当你设置 ***reschedule*** 参数为true，然后任务将在之后重新排列。

#### job.reschedule(spec)
这个方法将取消所有挂起的调度，然后使用给定的规则重新注册任务.
将返回 true/false 来说明成功/失败.

#### job.nextInvocation()
这个方法返回一个日期对象为这个任务的下一次调用计划，如果没有调度安排，则返回null.

## 贡献

这个模块由 [Matt Patenaude] 最初开发, 现在由
[Tejas Manohar] 和 [其他贡献者] 维护.

我们非常希望得到你的贡献. 做出有意义的和有价值贡献的人，将会被给予贡献的权限在他们认为合适的地方做出贡献.

在跳过之前, 检查我们[贡献]向导页面!

## Copyright and license

Copyright 2015 Matt Patenaude.

Licensed under the **[MIT License] [license]**.


[cron]: http://unixhelp.ed.ac.uk/CGI/man-cgi?crontab+5
[Contributing]: https://github.com/node-schedule/node-schedule/blob/master/CONTRIBUTING.md
[Matt Patenaude]: https://github.com/mattpat
[Tejas Manohar]: http://tejas.io
[license]: https://github.com/node-schedule/node-schedule/blob/master/LICENSE
[Tejas Manohar]: https://github.com/tejasmanohar
[other wonderful contributors]: https://github.com/node-schedule/node-schedule/graphs/contributors
[cron-parser]: https://github.com/harrisiirak/cron-parser
