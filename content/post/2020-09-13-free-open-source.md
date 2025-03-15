---
title: "Free and Open Source Blog"
date: 2020-09-13T21:22:30-04:00
draft: false
---

The blog is now truly free and open source.

----
<!--more-->

## Wait, my blog wasn't FOSS?

You'd think that, having taken a job at Red Hat [almost two and a half years ago](/post/2018-05-15-much-needed-updates/), I would have done
my due diligence and made sure my blog was free and open source.
After all, the [code](https://github.com/adambkaplan/adambkaplan-hugo) and
[content](https://github.com/adambkaplan/adambkaplan.github.io) are up on GitHub for the world to
see.
Doesn't that make it so?

Alas, my dear readers, you would have been mistaken.
Simply posting code in a public location is not enough to make my code _free_ - just open source.
Without a license, the code and generated content are all maximally copyrighted by me.
If someone wanted to fork my alterations to the Anake theme, or republish my blog entry, they would
need to get my permission.
And if I someone copied my blog or code without my permission, I could sue.

## The License

With this post and associated update, the site and Hugo template content is available under the
[Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/),
SPDX license identifier CC-BY-4.0. This license change is retroactive - all previous content is
hereby available under the license.

I must admit that it was a challenge determining if I should use the CC-BY-4.0 license for the
Hugo templates and Markdown that I publish on
[adambkaplan-hugo](https://github.com/adambkaplan/adambkaplan-hugo).
Some items, like the deploy.sh script, feel more like software than content.
Ultimately what swayed me was the Kubernetes docs
[website repo](https://github.com/kubernetes/website), which uses CC-BY-4.0 for everything.

## SPDX License Identifiers

Lately I've had to set up licenses and copyright notices for other open source projects I'm
involved in.
Some things I've done - and which I've carried to my blog - are a departure from what I accustomed
to seeing in the wild.
These new practices are based on the latest guidance from the Linux Foundation, which include:

1. No year in the [Copyright Notice](https://www.linuxfoundation.org/blog/2020/01/copyright-notices-in-open-source-software-projects/).
2. The usage of [SPDX License Identifiers](https://www.linuxfoundation.org/blog/2018/08/solving-license-compliance-at-the-source-adding-spdx-license-ids/) in place of license boilerplate.

The latter is a welcome change for compliance and reducing boilerplate in code.
Use them in your projects!
