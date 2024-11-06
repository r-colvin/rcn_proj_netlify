---
title: HTTP Basic Auth with Netlify
sidebar_position: 30
---

## Intro
As Docusaurus builds to a set of static HTML, CSS and JavaScript files, it is not possible to add authentication, instead Docusuarus puts the burden of auth onto the hosting platform. This guide is how I setup HTTP Basic Auth for robcolvin.net using Netlify. 

Although Netlify has some great documentation (https://docs.netlify.com/) their docs on setting up HTTP Basic Auth, and more importantly how to do it securely are not straight forward or beginner friendly. It took me a few days to wrap my head around how it should work. Hopefully this page will help someone else in the future (or even me if I ever need to do this again).

I strongly suspect the complex way of implementing Basic Auth on a site in conjunction with the incomplete documentation is leading to security issues for many users (based on my own research). I have reported this to Netlify, but hopefully this page can fill the void for now.

_These instructions are correct as of writing (2022-08-31); If Netlify change anything though, they may be invalid_

## Use Case to Solve
robcolvin.net is my personal information repository - part of that is documenting my projects and sometimes I'm not ready to release the docs, they are works in progress or just for me personally.

With Docusaurus deployed to Netlify (or Amplify) via GitHub I could set up branches for each project, and only when I merge to master would it go to the public deployment, and for some use cases this might be fine, but generally speaking I would like to just go to robcolvin.net and pick up working on a project where I left off. Within a project some pages might be done and public and others are placeholders or work in progress. For this use case then fine grained access control is required, down the folder or page level. This is where Netlify has an advantage over AWS Amplify - Amplify will allow you to add HTTP Basic Auth but it is limited to a single user and is deployed site wide. In contrast, in addition to a site wide password, Netlify allows you to create multiple users, passwords and build an auth file controlling access to a folder or single page. Perfect!

## Pre-requisites
It is assumed that you have your site deployed in Netlify already and know your way around Netlify - deploying and building etc. It is also assumed that you have a plan with access to the *Basic authentication headers* feature (see [plans](https://www.netlify.com/pricing/?_gl=1*1v5farx*_gcl_aw*R0NMLjE2NTk0MzQwNjQuQ2p3S0NBandscU9YQmhCcUVpd0EtaGhpdERzc2FSZEw5bzNrVHY5RTh0cmNtaks4b2s5aEpVWENvdlBNRlpKQ2p3d3BlZ2pfTHdZX1J4b0N3SmtRQXZEX0J3RQ..&_ga=2.72408406.459288328.1661934267-1376030614.1659006159&_gac=1.222357993.1659434064.CjwKCAjwlqOXBhBqEiwA-hhitDssaRdL9o3kTv9E8trcmjK8ok9hJUXCovPMFZJCjwwpegj_LwY_RxoCwJkQAvD_BwE#features-basic-authentication-headers) to confirm).

Also I assume you have read the documentation [Custom username and password protection through basic authentication](https://docs.netlify.com/visitor-access/password-protection/#selective-protection-with-basic-authentication) and have paid attention to the warning regarding Sensitive Information.

I assume you know how to use `sed` [sed man page](https://linux.die.net/man/1/sed)

## How to Setup Basic Auth in Netlify
In order for Netlify to implement Basic Auth, it wants to see a file called `_headers` containing *Basic-Auth* rules. A Basic-Auth rule looks like the following

```
/folder_to_protect/*
  Basic-Auth: USERNAME_1:PASSWORD_1 USERNAME_2:PASSWORD_2 
```

In the above as you might expect the `*` wildcard will apply the rule to all files under `/folder_to_protect/`. You can alternatively add a specific file (page) if you only want to protect a certain file, or you can use a reg-exp such as `/folder_to_protect/*.md` to protect all Markdown files. I'm sure you can work out how to extend that to your needs! You can of course have as many entries in the `_headers` file as you need, so you can be as granular as you like.

The problem is that if you were to add users and passwords to a file called `_headers` and then push to Git, your usernames and passwords would be exposed. If your repository was private it would still be a problem, but at least contained to a group of people, however if your repository is public then you have exposed your site usernames and passwords to the world... it is a simple matter to search GitHub for Docusuarus pages with exposed usernames and passwords. 

__IF YOU HAVE EXPOSED YOUR CREDENTIALS IN A REPO, PLEASE FIX IT!__

read on for the _right_ way to configure Basic Auth in Netlify - which is to store the usernames and passwords in environment variables, safely protected inside Netlify's UI, and only access them inside the build environment. 

### Create netlify_headers
Create a file in the root of your project called `netlify_headers` and within this file create Basic-Auth rules as above. However instead of inserting actual usernames and password, use placeholders - for example:

```
/folder_to_protect/*
  Basic-Auth: USERNAME_PLACEHOLDER1:PASSWORD_PLACEHOLDER1 USERNAME_PLACEHOLDER2:PASSWORD_PLACEHOLDER2
```

### Create Environment Variables
We will store the actual usernames and passwords as environment variables within Netlify. These are only visible to those with access to the Netlify account, and are obfuscated on screen. Log into your Netlify account and select the site (read more on Environment variables in Netlify here - [Build environment variables](https://docs.netlify.com/configure-builds/environment-variables/?_gl=1%2acpnn8w%2a_gcl_aw%2aR0NMLjE2NTk0MzQwNjQuQ2p3S0NBandscU9YQmhCcUVpd0EtaGhpdERzc2FSZEw5bzNrVHY5RTh0cmNtaks4b2s5aEpVWENvdlBNRlpKQ2p3d3BlZ2pfTHdZX1J4b0N3SmtRQXZEX0J3RQ..&_ga=2.232116826.459288328.1661934267-1376030614.1659006159&_gac=1.16523844.1659434064.CjwKCAjwlqOXBhBqEiwA-hhitDssaRdL9o3kTv9E8trcmjK8ok9hJUXCovPMFZJCjwwpegj_LwY_RxoCwJkQAvD_BwE))

- In the top menu select _Deploys_ 
- Select _Deploy Settings_ 
- In the left, under _Build & deploy_ select _Environment_
- Ceate environment variables, one for every username and password you need to store. The environment variable name should match the placeholder you used earlier in the `netlify_headers` file. So if you have a placeholder called *USERNAME_PLACEHOLDER1* you should created an environment variable called `USERNAME_PLACEHOLDER1`. The value of the environment variable should be the chosen username or password.

### Create a build script
A build script will be used to take the `netlify_headers` file, replace the placeholders with the values from the environment variables and then create the `_headers` file Netlify is expecting. The `_headers` file is only used inside the Netlify hosting environment and so is not generally available keeping your credentials safe.

Although you *could* implement the build script inside the UI, I find it easier to use another special file in Netlify, `netlify.toml` which is a general config file to control your site build etc. You can read more about `netlify.toml` here - [Syntax for the Netlify configuration file](https://docs.netlify.com/routing/headers/#syntax-for-the-netlify-configuration-file)

- create a file in the root of your site called `netlify.toml`
- inside the file add a build-block

```
[build]
    publish = "build"
    command = "npm run build && cp netlify_headers build/_headers && sed -i \"s|USERNAME_PLACEHOLDER|${USERNAME_ENV_VARIABLE_NAME}|g\" build/_headers && sed -i \"s|PASSWORD_PLACEHOLDER|${PASSWORD_ENV_VARIABLE_NAME}|g\" build/_headers"
```

the above does a few things:
1. I am using `npm run build` to actually build my site files from code (I am using Docusaurus) - you should replace this with your own build command as appropriate
2. once the build is complete, the `netlify_headers` file is copied to the build folder (`/build`) and renamed to `_headers` as expected. 
3. `sed` is then used to replace the placeholders in the `_headers` file with the environment variable values set earlier. 

Notes: 
- you will need to have a ´sed` command for _each_ username and password you need configured
- yes! there are more efficient ways to write the build command, I am intentionally using a more verbose method to ease readability rather than show off my sed-fu!

If all goes to plan, you should now have a functional HTTP Basic Auth protection for folders and pages in your Netlify hosted site. If it _hasn't_ gone to plan, read on for how to troubleshoot

### Troubleshooting

#### NETLIFY_BUILD_DEBUG
The first step to troubleshoot will be to enable build debug logs. Create a new environment variable called `NETLIFY_BUILD_DEBUG` with the value `true` (see [Netlify configuration variables](https://docs.netlify.com/configure-builds/environment-variables/#netlify-configuration-variables) and also [Build troubleshooting tips](https://docs.netlify.com/configure-builds/troubleshooting-tips/))

#### Forum
#### Netlify Support

### Logout

## Advanced Use Cases
### Private Repositories
### Branching Strategies