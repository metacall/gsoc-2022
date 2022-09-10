<p align="center">
  <img width="700" src="https://user-images.githubusercontent.com/65965202/189445588-aabd3e70-a5b3-4494-a353-6ec73d487eba.jpeg">
</p>

<h1 align="center">GSOC 2022 with Metacall</h1>

## What the heck is metacall?

<p>MetaCall is a polyglot runtime environment that lets developers call functions and methods between different programming languages. For example, we can call a Python function from a Node.js file. It supports languages like NodeJS, C++, C, Ruby, Python, C# & Typescript.<p>

## Metacall FaaS

<p>The Metacall FaaS, which lets you deploy your functions into the Metacall dashboard, is another tool that Metacall offers.</p>

<p>For those who don't understand what FaaS is, Let me explain :- <p>

```
In simple words, As a developer, you don't need to worry about managing servers, devops, or anything else all you need is your functions (the application's entity), and Metacall FaaS will turn it into an API endpoint so you can use it anyway you like. So you can see how intriguing it is and how much time it will save you. Applications that previously took months to develop may now be created in as little as 4-5 hours because to the reduction in complex work.
```

Visit <a href="https://metacall.io/doc.html#/" >metacall.io</a> for more information on Metacal FaaS.

So, Earlier the graphical interface of <a href="dashboard.metacall.io">dashboard.metacall.io</a> can be used to access the Faas service from Metacall. Devs prefer terminals over graphical user interfaces, therefore as part of Google Summer of Code 2022, I set out to develop a command line tool to access dashboard functionality using native terminals like Git Bash.

You can find our work on github <a href="https://github.com/metacall/deploy">Deploy CLI</a>.

## How it works?

```
MetaCall deployments takes your code package, being inside a repository or zip, uploads it into a dedicated machine and runs selected modules, exposing to the internet ports that are opened, it also exposes exported functions from modules as serverless functions, and static files.
```

<p>Let me tell you how you can convert your JavaScript functions into an API endpoints although it supports many languages.</p>

<p>Steps to be followed :-</p>

`Step 1` :- Initialize NodeJS using `npm init -y`.

`Step 2` :- Create any .js extension file (ex - `sum.js`) and write functions and export them.

```js
export const sum = (a, b) => a + b;
```

Now that you are almost finished, all that is left to do is generate the `metacall.json` file, which you can do manually or on the fly as if you like. Below is an example of how to manually construct this file.

```json
{
  "language_id": "node",
  "path": ".",
  "scripts": ["sum.js"]
}
```

`Step 3` :- Install metacall-deploy package using `npm install --global @metacall/deploy`

`Step 4` :- In your terminal, execute - `metacall-deploy --workdir=PATH-TO-YOUR-APP-DIRECTORY`

<p>And your application will be deployed within no time deployed.</p>

<p> Your application will be launched quickly.</p>

## Here is the quick video explanation for the whole process.

[![asciicast](https://asciinema.org/a/520124.svg)](https://asciinema.org/a/520124)

## Work done for achieving all the things I mentioned above

<p>The major goals of my GSoC with the metacall were completed. These goals were -</p>

### 1. Implementing all the flags -

Metacall deploy provides 15 CLI flags to consume API endpoints provided in <a href="https://github.com/metacall/protocol/blob/master/src/protocol.ts">Metacall Protocol</a>. A list of all supported options can be <a href="https://github.com/metacall/deploy#supported-arguments-and-commands">found here</a>.

We will be adding more flags in future to ease deployment.

### 2. Pull requests for all the flags I added

| Flag                   | Pull Request                                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| `--email & --password` | <a href="https://github.com/metacall/deploy/pull/24">#24</a>                                        |
| `--help`               | <a href="https://github.com/metacall/deploy/pull/29">#29</a>                                        |
| `--inspect`            | <a href="https://github.com/metacall/deploy/pull/30">#30</a>                                        |
| `--delete`             | <a href="https://github.com/metacall/deploy/pull/39">#39</a>                                        |
| `--addrepo`            | <a href="https://github.com/metacall/deploy/pull/46">#46</a>                                        |
| `--listPlans`          | <a href="https://github.com/metacall/deploy/pull/97">#97</a>                                        |
| `--force`              | <a href="https://github.com/metacall/deploy/pull/77">#77</a>                                        |
| `--logout`             | <a href="https://github.com/metacall/deploy/pull/96">#77</a>                                        |
| `And a lot`            | <a href="https://github.com/metacall/deploy/pulls?q=is%3Apr+is%3Aclosed+author%3ACreatoon">More</a> |

### 3. Improving test coverage

Testing plays an important role in software development. I wrote all the neccessary integration tests and increaed the test coverage.

You can find the details of tests I wrote <a href="https://github.com/metacall/deploy/blob/master/src/test/cli.integration.spec.ts">here</a>

And many more bug fixes and improvements …
I wish I could explain all the features and fixes I have implemented during GSoC but the list is huge. Though you can find a detailed version of my work during GSoC <a href="https://github.com/metacall/deploy">here</a>.

## Conclusion

The ability to quickly and efficiently deploy your raw functions to the metacall faas via metacall deploy is a game-changer.

There are a lot of learnings that come along with experience and familiarity with the core team and community. Collaboration and team effort are key for the success of any large-scale project.

Community is really important. Never lose sight of the problem we're trying to solve, and constantly pay attention to the needs and worries community.

Following design patterns and modifying pre-existing code help keep the application's consistency. Always create code that is easy to scale, manageable, and loosely connected.

The most significant development in my career has definitely been GSoC and Metacall. I am really grateful to Google and Metacall for providing me with this fantastic platform. This opportunity has unquestionably had a profound effect on me. I've been able to advance from being an open-source contributor to an open-source maintainer because of it.
