---
layout: post
title: "Hackergame 2021 Writeup Part 2(Easy level)"
categories: Writeup Hackergame
katex: True
---

This part of writeup contains tasks that require a bit programming & linux experience.

## 卖瓜

It's obvious that, you can't get exactly 20 with some 6 and 9. So there must be some magic that can help us achieve it.

As the task said, if the result becomes a float it will be invalid. But how can we turn it to a float?

I tried to put 3.3333 * 6 to the scale by changing the network packet but it didn't work. So it may be something related to integer overflow.

Let's try a $2^{63}$ level integer, $10^{18}$.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102213220.png)

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102213252.png)

Surprisingly, after adding it twice, we turned it to a float.

So looks like if the weight on the scale exceeds the maximum 64-bit signed integer, it will turn to a float.

But if we type a bigger number like $10^{19}$ which already exceeded the limit, it will add nothing to the scale.

What about to type exactly the limit $2^{63}-1$? In the 6 jin field, it won't work. But if in 9 jin field...

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102214817.png)

BOOOOOOM!

We make it overflow to negative.

And the absolute value of it can't divide by 3. So after carefully increasing it back to positive, we can get a value which has a remainder of 1 when is divided 3.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102215643.png)

Doing this again then we'll get a remainder of 2.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102215814.png)

Then just add a 6 and a 9, we'll get 20.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102215853.png)

## 透明的文件

Open the file, you'll see something like `[39m[19;61H[9;12H`. 

The format is quite similar to the thing we used to change linux terminal text color. Google it, and you'll know it's [ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code). 

So let's add `\033` before each `[`, which can be done with vim command `:s/\[/\\033\[/g`. And replace all invisible space character to other things like `#`.(can be done with `:s/\ /#/g` in vim)

Finally print it with echo -e, and you'll get the flag.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211102221411.png)

## 图之上的信息

## 加密的U盘

## 阵列恢复大师