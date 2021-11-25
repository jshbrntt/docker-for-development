---
theme: uncover
class: invert
---

<!--

- Basic TODO API Example
    - MongoDB

- Hypochondriac View
    - Which version of Yarn did you use?
    - Which version of Node.js did you use?
    - What operating system are you using?
    - What architecture running on?
        - x64
        - x86_64
        - aarch64 (Apple M1)
    - Is Node.js in your path?
    - How is it installed are you managing it with nvm?
    - What does Node.js depend on?
        # Linux
        ```bash
        $ which node
        /home/joshua/.nvm/versions/node/v16.13.0/bin/node

        $ ldd /home/joshua/.nvm/versions/node/v16.13.0/bin/node
            linux-vdso.so.1 (0x00007ffc1773c000)
            libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4aa8293000)
            libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f4aa80b2000)
            libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f4aa7f63000)
            libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f4aa7f48000)
            libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f4aa7f25000)
            libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4aa7d33000)
            /lib64/ld-linux-x86-64.so.2 (0x00007f4aa82ac000)
        ```

        # Windows
        ```cmd
        C:\Users\Joshua>where node
        C:\Program Files\nodejs\node.exe

        C:\Users\Joshua>dumpbin /dependents "C:\Program Files\nodejs\node.exe"
        Microsoft (R) COFF/PE Dumper Version 14.29.30133.0
        Copyright (C) Microsoft Corporation.  All rights reserved.


        Dump of file C:\Program Files\nodejs\node.exe

        File Type: EXECUTABLE IMAGE

        Image has the following dependencies:

            dbghelp.dll
            WS2_32.dll
            IPHLPAPI.DLL
            PSAPI.DLL
            USERENV.dll
            ADVAPI32.dll
            USER32.dll
            CRYPT32.dll
            bcrypt.dll
            KERNEL32.dll
            WINMM.dll

        Summary

            2D9000 .data
            D0000 .pdata
            2647000 .rdata
            1D000 .reloc
            25000 .rsrc
            1118000 .text
                1000 _RDATA
        ```
    - Which version of libstdc++ is it using?
    - What NPM registry are you using? Just the default?
    - Which DNS server are you using?
    - What ISP are you using?
    - What's your internet speed like?

- You and your friend want to work on the project together.
    - He's a good developer but doesn't know how to Linux/Windows he wants to use what he's most familiar and productive with e.g. macOS.

- Enter Docker
    - ./assets/it-works-on-my-machine.jpg
    - We'll ship your file system*
    - Lets you target Kuberenetes as your deployment target.
        - You want Kubernetes because it makes scaling your application and infrastructure easy.

- Okay how to we Dockerize this application

    - Covers production use case, not development.
        - https://snyk.io/blog/10-best-practices-to-containerize-nodejs-web-applications-with-docker/
        - https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
        - https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md
    - Covers development.
        - https://www.docker.com/blog/keep-nodejs-rockin-in-docker/

- Dockerfile
    - FROM node (nothing else)

Okay this works fine but where is our code?

Bind mount, okay we have our code.

Wait why doesn't our node_modules work?

- C++ addons
https://nodejs.org/api/addons.html

Although our code is JavaScript it like Node.js is dependent on Node modules which depend on C++ addons.

Often because there's something already written in C++ we want to make use of.

Or because the use case is computationally intensive (crypto) and needs to be as fast as possible.

- rm -rf node_modules
npm install

Wait this is really slow why?
- macOS

Okay well lets install it in the parent.

or try symlinking.

- Benefits of installing node_modules as part of the development image.

- We have to do it on the CI anyway so why not store these node_modules in a Docker image and use it as a cache.

- This means if our project has been tested and ran in the CI environment our only bottle neck is downloading the image

(no compilation or module resolution required)

We want to add a database.

- What is a Docker image?
    - It's just a tar archive with some magic around it.

- Dockerfile
    - Multi-stage builds

- Installing node_modules on host filesystem is slow on macOS, fine on Linux, meh on Windows.


# - Best Case Scenario
#
# So maybe you developing your application and hosting it on the same box.
#
# Questionable choice (for reasons I won't go into right now, but cool you do you).
#
# - Power cut at your house your application goes down and you loose market share and revenue because of it.
#
# Okay maybe having it on my home computer was a bad idea.
#
# Lets pack up my computer and send it to the nearest data center.
#
# They'll never have a power cut right?
#
# Nope, they're probably sensible have have things like Uninterruptible Power Supplies (UPS).
#
# Oh wait OVHcloud burns down, your server with it and any source code you have.
#
# https://www.reuters.com/article/us-france-ovh-fire-idUSKBN2B20NU
#
# Okay no worries we've backed up the server.
#
# Great, so now you find a new hosting provider and you redeploy your site.
#
# You repoint your DNS, great only 1 day of downtime.
#
# Maybe you even deploy it to a different server provider as well in advance for backup.
#
# A couple months later someone posts your application to Hacker News, and it gets upvoted to the top for a day.
#
# Suddenly your TODO app is a viral hit and on that same day same day your website is responding slower and slower to new visitors, until it stops responding all together.
#
# - https://www.forbes.com/sites/davidthier/2016/07/08/pokemon-go-servers-down-again/?sh=4dba14c229c9
# - https://www.vox.com/2016/7/16/12207552/pokemon-go-server-outage
#
# You phone up SuperCheapServers Inc. and request if you can rent some more servers in the rack.
#
# They come back with sorry we don't have any surplus servers at the moment but we can order some in might get here in 2-3 days?
#
# Not good enough, you want to catch the wave of good press coming your way?
#
# So where do you turn to next? The CLOUD!
#
# Now you are only an button press away from getting a new machine but you still have to setup unzip your project and set it up.
#
# Oh whoops turns out you missed the wave and now you're overpaying on servers when your demand doesn't match.
#
# Doing this setup and teardown is a pain lets automate it.
#
# So maybe you're using Ubuntu 20.04 to develop your application and deploying your application on a Ubuntu 20.04 EC2 instance in AWS?
#
# Maybe you're company mandates you use Ubuntu 20.04 everywhere including development environments.


-->

# <!-- fit --> Docker for development<br>:whale: :electric_plug: :technologist:

---

# :beach_umbrella: New Project Honeymoon

<!-- <style scoped>
section {
    display: grid;
    grid-template-columns: 50% 50%;
    grid-template-rows: 20% 80%;
}

h1 {
    grid-column-end: span 2;
}
/* Wraps unordered lists into columns */
/* section > * {
    flex: 1 0 auto;
} */
</style> -->

<!--
So let's imagine you start a new project.

You've decided to make a TODO app, because the thousands of existing ones do not fit your specific needs.

In the begunning there's all this possibility space yet to be realized.

You might have a bunch of questions that you answered without even thinking twice about.
-->

<!-- <iframe src="http://localhost:3001/" title="Awesome TODO" height="100%"></iframe> -->

<!-- ```shell
$ yarn init
yarn init v1.22.17
question name (awesome-todo):
question version (1.0.0):
question description: Better than the rest?
question entry point (index.js): src/main.js
question repository url: ¯\_(ツ)_/¯
question author: Me
question license (MIT): WTFPL
question private: true
success Saved package.json
Done in 57.59s.

# --snip--
```

* Let's build a TODO app! ✅
* I'm a bullet point
 -->
