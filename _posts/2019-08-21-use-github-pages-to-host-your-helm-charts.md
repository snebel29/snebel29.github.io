---
layout: post
title: Use github pages to host your helm charts
description: Use github pages to host your helm charts publicly for free
---

Leveraging [the official helm chart repository](https://github.com/helm/charts) is a great way of publishing and sharing your helm charts with the world, however this comes with a price.

Changes to your chart will have to be approved and merged by someone else, which in my experience can take a long time...

For those case when you need autonomy and fast releasing you better host your own chart repository, essentially all you need is an object storage. There is many out there [Google Cloud Storage](https://cloud.google.com/storage/), [Amazon S3](https://aws.amazon.com/s3), [Azure blob storage](https://azure.microsoft.com/en-us/services/storage/blobs/), [Digital Ocean Spaces](https://www.digitalocean.com/products/spaces/) ... Most of them would be covered by their free tier so at the end of the month this would be for free.

However they may require you to open accounts adding your credit card even if the bill will be zero, and be harder to use, so why not use [github pages](https://help.github.com/en/articles/what-is-github-pages) to do so?

It has decent [usage limits](https://help.github.com/en/articles/what-is-github-pages#usage-limits) for "small" to "medium" size projects.

## How to do it?

1. Create a repository

> &#9888; avoid using `charts` since this may collide with the stable chart repo, so you won't be able to fork repositories with the name `chart` anymore!

2. [Setup GitHub Pages](https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages) pointing to the `/docs` folder. 


In your newly created repository go to `Settings` > `Github Pages` > `Source` and select `Master branch /docs folder`


From there, you can create and publish charts by pushing the helm package of a chart to the `/docs` directory, you could keep the source code in the root of the repository as well, follows an example.

```console
$ helm package kwatchman/
$ mv kwatchman-0.1.0.tgz docs/
$ helm repo index docs --url https://snebel29.github.io/snl-charts
$ git add --all
$ git commit -m "Adding kwatchman"
$ git push origin master
```

### Adding this repository on helm
To make it available for your helm installation, you have to add the repository locally.

```console
$ helm repo add snl-charts https://snebel29.github.io/snl-charts
```

You can find the full working example of my own repository [snl-chart](https://github.com/snebel29/snl-charts)
