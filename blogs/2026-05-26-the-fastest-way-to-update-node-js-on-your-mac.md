---
title: The fastest way to update Node.js on your Mac
url: https://forem.com/alikarbasicom/the-fastest-way-to-update-nodejs-on-your-mac-18ke
date: '2026-05-26'
author: Ali Karbasi
feed_url: https://forem.com/feed
---
Posted on Jul 22, 2023 1 min read 🤖 AI summary: This article outlines a quick method for updating Node.js on macOS by utilizing the n package. After verifying your current version, you can install this package globally via the command line. Once installed, the n utility allows you to effortlessly update to the stable, latest, or LTS releases of Node.js using simple terminal commands, ensuring your development environment is compatible with new JavaScript libraries. If you are using any new JavaScript libraries in the NPM, it is important to ensure that you have the latest version of Node.js installed. Keeping your Node.js up to date is crucial. Now, let’s explore the simplest and fastest method to accomplish this on your macOS! Type this command in terminal to check your Node.js version: node -v Here’s what the output should look like: v x.x.x If you haven’t installed the n package yet, use npm to install it: sudo npm install -g n After installing the n package, you can proceed to update your Node.js to the desired version. Use this command to install the stable node release: sudo n stable Use this command to install/update the latest node release: sudo n latest Use this command to install/update the latest LTS node release: sudo n lts That’s all! You can confirm the successful update by checking your Node.js version once again. Happy coding :D
