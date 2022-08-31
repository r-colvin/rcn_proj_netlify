---
title: HTTP Basic Auth with Netlify
sidebar_position: 30
---

# Intro
As Docusaurus builds to a set of static HTML, CSS and JavaScript files, it is not possible to add authentication, instead Docusuarus puts the burden of auth onto the hosting platform. This guide is how I setup HTTP Basic Auth for robcolvin.net using Netlify. 

Although Netlify has some great documentation (https://docs.netlify.com/) their docs on setting up HTTP Basic Auth, and more importantly how to do it securely are not straight forward or beginner friendly. It took me a few days to wrap my head around how it should work. Hopefully this page will help someone else in the future (or even me if I ever need to do this again).

# Use Case to Solve
robcolvin.net is my personal information repository - part of that is documenting my projects and sometimes I'm not ready to release the docs, they are works in progress or just for me personally.

With Docusaurus I could set up branches for each project, and only when I merge to master would it go to the public deployment, and for some use cases this might be fine, but generally speaking I would like to just go to robcolvin.net and pick up working on a project where I left off. Within a project some pages might be done and public and others are placeholders or work in progress. For this use case then fine grained access control is required, down the folder or page level. This is where Netlify has an advantage over AWS Amplify - Amplify will allow you to add HTTP Basic Auth but it is limited to a single user and is deployed site wide. In contrast Netlify allows you to create multiple users and build an auth file controlling access to a folder or single page. Perfect!

# How to Setup Basic Auth in Netlify

# References