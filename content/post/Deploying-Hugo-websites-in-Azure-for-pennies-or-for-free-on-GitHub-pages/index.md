---
title: "Deploying Hugo websites in Azure for pennies or for free on GitHub pages"
description: "Learn how to deploy your Hugo websites to a Azure Static Web App, Azure Blob storage container or GitHub pages"
date: 2020-10-05T21:04:10+01:00
draft: true
---

- [What is this Hugo thing?](#what-is-this-hugo-thing)
- [Prerequisites](#prerequisites)
- [Azure Static Web Apps](#azure-static-web-apps)
- [Azure Blob storage](#azure-blob-storage)
- [GitHub pages](#github-pages)

I will be touching on a little about Hugo and why I like it. I am also going to show you three ways to deploy your Hugo websites: with Azure Static Web Apps (preview), Azure Blob storage and GitHub pages.

## What is this Hugo thing?

Hugo is a static site generator. 

_continues to stare blankly at the screen_

Yeah, I did too. 

With static site generators (like Hugo), you forego caring about databases and server-side code (like PHP). Instead, your HTML pages are generated either each time a user visits your website, or in Hugo's case, the HTML pages are generated each time you create or update content.

Instead of writing your pages and blog posts in HTML files or in feature-rich WYSIWYG editors on some bloated content management system (WordPress), you write them in markdown (at least you do with Hugo). Then you invoke the process to generate the HTML files, using your new markdown content as source. The generated HTML files land in a very particular directory (`/public`). After that, all you need to do is get that directory on a hosting solution as your web root. Done.

When it comes to styling, themes are mostly plug and play, too. Fancy a new theme? No problem. [Download one](https://themes.gohugo.io/) and drop it in the `/themes` directory, update `config.toml` a little. Donezo.

If no runtime is needed to run your website, other than something which is capable of offering HTML files over HTTP, not only does this opens up your hosting opportunities, performance is also another great benefit. You do not need a hosting package that sits on nginx or Apache, running PHP or whatever. For example, you can host on [Azure Blob storage](https://azure.microsoft.com/en-gb/services/storage/blobs/) or [GitHub pages](https://pages.github.com/).

GitHub pages is free. It also lets you use your own domain and offers free SSL certificates via LetsEncrypt. Azure Blob Storage isn't quite free but it is pennies. It is £0.0144 per GB (in UK South) of storage and the first 5GB of bandwidth from this zone is free. You can do this on many more platforms too, such as Amazon S3, Netlify, Heroku, GitLab Pages, Google Cloud Storage and more.

I am all-for ditching WordPress. I really do not want to pay for hosting any more. Over the last few years I have grown more comfortable with Git and working on Azure and GitHub. If you relate to that or are enticed by any of the benefits listed above, I highly recommend you at least give it a go!

Here are some resources I used to learn Hugo:

- [Hugo - Static Site Generator | Tutorial](https://www.youtube.com/watch?v=qtIqKaDlqXo&list=PLLAZ4kZ9dFpOnyRlyS-liKL5ReHDcj4G3) YouTube series
- [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/) official documentation

## Prerequisites

Before we get started, I am going to assume some things:

- You have an Azure subscription
- You have a GitHub account and repository
- The repository contains either only your static site content, or the whole Hugo directory with your latest generated content in the `/public` directory
- You get by with Git enough to be able to do things like committing/pushing - doesn't have to be command line, GitHub Desktop is totally fine!

## Azure Static Web Apps

Let us start with deploying Hugo to an Azure Static Web App. [Earlier this year Microsoft announced Azure Static Web Apps](https://techcommunity.microsoft.com/t5/apps-on-azure/introducing-app-service-static-web-apps/ba-p/1394451) and it's currently in preview. While it's in preview, this resource is free.

It boasts "deep GitHub integration", which is true. When you create the resource and associate a GitHub repository with it, it creates a GitHub Actions workflow YAML file in your repository. It also stores a secret in the repository. The secret is used by the workflow to authenticate to Azure. This build process ships everything that's in the `/public` directory up to a Blob container in Azure, creates a static website and offers you free LetsEncrypt SSL certificates.

1. Log in to [Azure portal](https://portal.azure.com)
2. Create a new Static Web App resource
   1. Fill out the typical information ie resource group, name and region
   2. Sign in to GitHub
   3. Thereafter you will be able to choose organisation, repository and branch
   4. After that, you can configure build details. Ensure you choose Hugo and that you have the `App artifact location` set to `public`

![Static Web App resource creation in Azure Portal](images/staticwebapp-01.jpg "Some text") ![Creation of GitHub Actions workflow file by Azure](images/staticwebapp-02.jpg)

- https://docs.microsoft.com/en-us/azure/static-web-apps/publish-hugo

The good thing about using Azure Static Web Apps is that you essentially get Azure Blob storage and Azure Functions bundled in to one resource. This enables you to leverage the speed and flexibility of static site generators, while still being able to implement some dynamic abilities in to your website by rolling your own API via Azure Functions.

## Azure Blob storage

- Cost
- GitHub actions

## GitHub pages
