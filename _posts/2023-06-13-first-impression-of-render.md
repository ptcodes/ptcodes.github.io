---
layout: post
title: First Impression of Render.com
description: First impression of Render.com - setting up a Jekyll static site.
categories: [Amazon, AWS, Amlify, Render.com, Jekyll]
image: /images/social/render.png
---

[![Render.com](/images/render.png)](/first-imression-of-render/)


I have extensively utilized Amazon AWS for various applications, ranging from operating an EKS cluster for our [Televet product](https://cooperpetcare.com/) at Cooper Pet Care, to hosting my personal blog on [Amazon Amplify](https://www.ptimofeev.com/static-blog-with-jekyll-and-amazon-amplify/).

Regrettably, deployments to my blog on Amazon Amplify ceased functioning unexpectedly. Instead of troubleshooting the issue, I opted to try out [Render.com](https://render.com/), a relatively new hosting platform I've frequently heard about.

One advantage of Render is its free hosting for static websites, in contrast to the annual cost of $20-30 I've been incurring on Amazon Amplify.

And the first impression of Render was pretty good.

<!-- more -->

## Setting Up on Render

The setup process on Render was remarkably efficient and user-friendly. Within a mere five minutes, I managed to sign up, authenticate via Github, select my static blog's repository, and deploy a fresh instance of the Jekyll site. Render adroitly identified the command required to convert Jekyll assets into static files. Following successful deployment to a Render subdomain, my remaining task was a simple DNS settings update for my domain.

One notable feature is Render's automatic creation of SSL certificates. This further simplifies the process, contributing to a seamless user experience.

The user interface in Render is notably user-centric, providing an intuitive, easy-to-navigate layout. This stands in sharp contrast to the more convoluted interface of AWS, which can at times be daunting.

One specific Render feature I found useful was the capability to add a cron job without necessitating a standalone server. However, be aware that this is a paid feature and necessitates upfront credit card details.

As such, this blog is now proudly powered by Render.

## Conclusion

I'm certainly pleased with using Render for my personal projects and pet endeavors. However, for larger-scale or more critical tasks, I'd prefer to reserve judgment and assess Render's performance over a longer duration.
