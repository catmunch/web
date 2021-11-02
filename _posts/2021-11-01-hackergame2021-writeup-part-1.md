---
layout: post
title: "Hackergame 2021 Writeup Part 1(Entry level)"
categories: Writeup Hackergame
---

This part of writeup contains tasks that don't require experience in CTF&Linux.

## 签到

The task requires to find a page of the diary which is written during the contest, open the [link](http://202.38.93.111:10000/) and we'll find out that the page number in the URL params is exactly the [UNIX Timestamp](https://en.wikipedia.org/wiki/Unix_time) of the written time. So just find any converter then convert any time from 2021-10-23 12:00:00 to 2021-10-30 12:00:00 (UTC+8).

If you don't know the UNIX Timestamp before, you can also calculate manually.

## 进制十六——参上

In the screenshot, only the decoded part is pixelated, but remains all hex part. As it's encoded with ASCII (inferred from the previous characters), we can simply decode the hex numbers then we can get flag.

```python
Python 3.9.7 (default, Sep  3 2021, 12:37:55) 
[Clang 12.0.5 (clang-1205.0.22.9)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> (0x666C61677B5930555F5348305531445F6B6E30775F4830575F74305F43306E763372745F4845585F746F5F546578547D).to_bytes(48,'big')
b'flag{Y0U_SH0U1D_kn0w_H0W_t0_C0nv3rt_HEX_to_TexT}'
>>> 
```

## 去吧！追寻自由的电波

Task provides a pretty fast & high pitch voice file, and informs us it uses a method in radiotelephone field to distinguish similar pronunciations of letters.

First, we should search for that method, just google like "how to say letters in radiotelephone", then we'll find [NATO phonetic alphabet](https://en.wikipedia.org/wiki/NATO_phonetic_alphabet). So the voice will basically contain some English words.

Next, let's deal with the voice file. Fast & high pitch, that's normally the result of basic audio speed changing. You may ask, ah why when I speed up audios, the pitch doesn't change? It's just because modern editors such as Final Cut Pro / iMovie will automatically help you correct the pitch after changing speed. So we should find an editor which has the option to disable the auto-correction feature. As far as i know, Adobe Premiere Pro does. So let's take it as an example.

1. Drag the audio into timeline, right click it, and choose `Speed/Duration`

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101091135.png)

2. Make sure not to check the `Manage Audio Pitch` and slow down to ~50%

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101091319.png)

Then just play the audio and decode with [NATO phonetic alphabet](https://en.wikipedia.org/wiki/NATO_phonetic_alphabet). You'll get the flag.

## 猫咪问答 Pro Max

1. Use websites like [web.archive.org](https://web.archive.org) or stuff to visit sec.ustc.edu.cn then you'll find it.

2. The official writeup says you can check [here](https://lug.ustc.edu.cn/wiki/intro/), but during the contest it didn't update so you may count as 4 times. Alternatively, just enumerate the answer.

3. Search `LUG @ USTC` on Google Images or `活动室` on lug.ustc.edu.cn

4. Find that paper on [SIGBOVIK](http://sigbovik.org/2021/) and count it. (btw it's really funny)

5. Search `IETF Protocol Police` on google and you'll find the answer is `/dev/null`.![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101092314.png)

## 旅行照片

Note that there's a special blue KFC in the photo, so we can find that KFC first. Search `蓝色KFC` on Baidu, the first result is a KFC located in Qinhuangdao. We can then find it on Meituan to get more photos to ensure it's the one in the given photo.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101095430.png)

So that's it.

Search it on Baidu Map

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101095757.png)

We got the phone number.

Then we can use the street view feature to solve question 5.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211101095932.png)

Though the street view is old, we can infer that it's the same place from the pillar.

The question 1&2 can easily solve by KFC location. And for question 3, count the floor of the opposite building which has the same height with sea level line in picture.
