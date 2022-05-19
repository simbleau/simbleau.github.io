+++
title = "Blogging like a rustacean"
date = 2022-05-17T00:00:00+00:00
description = "Insights into the technologies used to deliver this blog with Rust, and with better performance, SEO, accessibility, and security than its competitor, Jekyll."

[taxonomies]
categories = ["Rust"]
tags = ["rust", "web"]
[extra]
toc = true
+++

# Overview
Look around! This blog is generated with Rust and no Javascript. This post will be explorating how this blog is generated with better performance, accessibility, and SEO than its competitior, Jekyll. We will also show, with proveable evidence, how our proposed tech stack supersedes those aspects.

# Jekyll?
[Jekyll](https://jekyllrb.com/) was written in Ruby and is the most popular static site generator (SSG) in use. The SSG ingests [Markdown](https://markdown-it.github.io/) formatted text and uses templates to generate a static website. This is ideal for those who need to quickly publish thoughts with nice formatting. [GitHub Pages](https://pages.github.com/) is *powered* by Jekyll[^ghpages_poweredby], and GitHub will use Jekyll to build your site by default[^github_rec]. These reasons and special privilege make Jekyll a ~~safe~~ popular option for most users.

The general workflow is to install Jekyll, create a project, build it, and host it on GitHub. If you create a GitHub repository named `<username>.github.io`, GitHub will automagically host this with GitHub Pages for free[^ghpages_quickstart]. You **can** use alternative static site generators with GitHub Pages' hosting, which we will explore later.

# Technologies
It's time to introduce the components that get taped together to present what you see now and de-obfuscate the magic. We install [Zola](https://www.getzola.org/) as a Rust-alternative SSG to Jekyll. Contrary to Jekyll, Zola makes 0 assumptions about your content, and requires a "*theme*" to tell it how to use generate static sites, providing flexibility can benefit from. We pair *Jieiku*'s "[Abridge](https://github.com/Jieiku/abridge)" theme with Zola for its maturity and design in blogging. We also will explain how to get free hosting through [GitHub Pages](https://pages.github.com/), offer a continuous integration strategy through [GitHub Actions](https://github.com/features/actions), and additional tools. The technologies are named below.

| Name                                                    | Purpose                                     |
| ------------------------------------------------------- | ------------------------------------------- |
| [Zola](https://www.getzola.org/)                        | SSG written in Rust                         |
| [Abridge](https://github.com/Jieiku/abridge)            | Zola theme (No JS)                          |
| [GitHub Pages](https://pages.github.com/)               | Free hosting for static sites               |
| [GitHub Actions](https://github.com/features/actions)\* | CI/CD for Zola                              |
| [LetsMarkdown](https://letsmarkdown.com/)\*             | Markdown collaboration tool written in Rust |

\* Optional

# Zola's opportunities
Let's clear the air on Zola - **It's not because of WASM**. Rust's renowned ability to transpile into Web Assembly (WASM) may be hip and trendy, but this is not the reason Zola is chosen to substitute Jekyll. The real reasons are simpler. Zola is a mature SSG with all of the necessities - live-reload previewing, scalability, markdown, and more. But more importantly, Zola makes 0 assumptions, allowing us to make a static website exactly how we want it.

## Operability
We, as developers, enjoy the ability to fix issues if they arise. Since Jekyll was written in Ruby, this presents a language barrier to rustaceans who otherwise have no interest in Ruby. You probably shouldn't ever *need* to write Ruby as a blogger, content is written in markdown, however you should *want* the ability to fix or contribute something to your SSG.

## Lighthouse metrics
Jekyll is one of the best simple, blog-aware solutions. It does a great job at optimizing search engine optimization (SEO), accessibility, and performance out of the box using best practice. But is it *the best*? [Google Lighthouse](https://web.dev/measure/) provides expert analysis on what is *the best*, so it will serve as our analytic framework. When measuring several blogs with Jekyll, off-nominal results were recorded. For evidence, I will pick on [Raph Levien's blog](https://raphlinus.github.io). He is a friend and active publisher in the Rust community, well known for his work on [Xi](https://xi-editor.io/), [Druid](https://github.com/linebender/druid), and [Piet](https://github.com/linebender/piet). Measuring his Jekyll blog is a good example, as there are several points missing from a perfect score. Since Zola makes no content assumptions, we can attack these last few points in a Zola theme and abuse the finer control given to us.

<div align="center">
{{ img(path="./raph_seo.png", alt="Raph's Lighthouse metrics") }}
</div>

*[Click here for the full report](raphlinus.github.io_2022-05-15_10-46-19.html)*

## Security
One metric our previous analytic framework does not measure is security. Thankfully Mozilla cared to create the [Mozilla Observatory](https://observatory.mozilla.org/) as an analytic security framework for websites. We find that Jekyll fails to take steps to improve security for common hosting solutions. Generally speaking, web security guarantees are provided by the web server, and Jekyll leaves up to you, which is impossible if you don't control the hosting servers. When measuring various Jekyll blogs, the average score has always been graded **F**. This oversight is an opportunity for improvement using Zola, which we will dig into.

<div align="center">
{{ img(path="./raph_security.png", alt="Raph's Observatory metrics") }}
</div>

# Abridge
Zola has an [index of themes](https://www.getzola.org/themes/) for generating varying static sites, such as documentation sites, blogs, or portfolios. In a journey to find the most mature blogging theme, I began with [Adidoks](https://www.getzola.org/themes/adidoks/). As time went on, I had to replace it[^del_adidoks], citing poor support for tagging, pagination, RSS/Atom, and more.

Enter Abridge. Abridge is a semantic classlight theme designed to be a modern emulation of Jekyll; fast, light, perfect. With caution in avoiding similar mistakes, I did my research. Abridge has support for these taxonomies, and most importantly, no javascript ([demo](https://jieiku.github.io/abridge-demo/)).

<details>
    <summary>How did Abridge compare to other themes?</summary>

Abridge had some issues at first. For weeks I contributed to make it more accessible to a wider audience and iron out the shortcomings of Jekyll. [Some of my contributions](https://github.com/Jieiku/abridge/pulls?q=is%3Apr+is%3Aclosed) include asynchronous deferral of font loading to improve the first contentful paint, fontawesome support, more social icons, SEO optimizations, color skins, and config options. Now I am happy to say Abridge makes all of the right tradeoffs, and is the most mature theme for blogging. Now we will prove these bold claims...
</details>


## Improvements over Jekyll
Abridge is, in my opinion, ready for wide adoption. Specifically, this theme is meant be a better Jekyll with improved performance, accessibility, SEO, and security. In fact, if Abridge does get a perfect scorecard on Google Lighthouse, it is a bug. [Please file an issue](https://github.com/Jieiku/abridge/issues/new).

<table>
<tr>
<th>Abridge's Google Lighthouse Score</th>
<th>Statistics</th>
</tr>
<tr>
<td>
<div align="center">
{{ img(path="./abridge_seo.png", alt="Abridge's Lighthouse score") }}
</div>

*[Click here for the full report](jieiku.github.io_2022-05-17_18-56-57.html)*
</td>
<td>

<!-- Statistics -->
| Audit          | Jekyll | Abridge | Gain  |
| -------------- | ------ | ------- | ----- |
| Performance    | 95     | 100     | 5.00% |
| Accessibility  | 97     | 100     | 3.00% |
| Best Practices | 100    | 100     | Same  |
| SEO            | 99     | 100     | 1.00% |

</td>
</tr>
</table>

Security was also improved for common free hosting. One of the complex issues with security is that you need to implement server-side rules, which isn't possible as an end-user of, say, GitHub Pages, Cloudfare Pages, Netlify, or other alternatives. Abridge mitigates this issue by inlining some common security metadata for you, which makes a difference. While Jekyll scores an **F**, Abridge scores a **B-**, subject to improve further.

<table>
<tr>
<th>Abridge's Mozilla Observatory Summary</th>
<th>Statistics</th>
</tr>
<tr>
<td>
<div align="center">
  {{ img(path="./my_security.png", alt="My Observatory score") }}
</div>
</td>
<td>

<!-- Statistics -->
| Audit        | Jekyll | Abridge | Gain   |
| ------------ | ------ | ------- | ------ |
| Grade        | F      | B-      | N/A    |
| Score        | 20/100 | 65/100  | 45.00% |
| Tests Passed | 6/11   | 8/11    | 18.18% |

</td>
</tr>
</table>


# GitHub Pages
If you are adventurous and fun, so there are many options to freely host your blog. I settled on GitHub Pages. If you create a GitHub repository named `<username>.github.io`, or convert your repository to a Pages deployment, GitHub will graciously host static content for free, at that address, and treat the code contained as the source code for Jekyll. *More on creating a GitHub Pages site [here](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)*.

There are some common pitfalls I would like to point out.

- GitHub Pages requires some ceremony to opt-out of Jekyll, for any static site generator. GitHub requires a [`.nojekyll`](https://github.com/simbleau/simbleau.github.io/blob/main/.nojekyll) file in the root level of the repository, since GitHub defaults you into using Jekyll[^github_rec]. This is your way of telling GitHub, "*Hey, I'm doing cooler things here!*".
- If you have a custom domain name like me, you'll need to stage a [`CNAME`](https://github.com/simbleau/simbleau.github.io/blob/main/zola/static/CNAME) file in the [`static/`](https://github.com/simbleau/simbleau.github.io/tree/main/zola/static) directory of your zola repository.

*If you're considering a migration to GitHub Pages, the Zola guide is [here](https://www.getzola.org/documentation/deployment/github-pages/)*.

# GitHub Actions
My favorite aspect of this tech stack is the autonomy. GitHub Actions is able to verify and deploy your Zola website with ease. If you push to your master branch and it's all good - you should see the changes on your deployment quickly! Otherwise, your blog is safe. The authors of Zola have included a very easy GitHub Action to help you deploy, batteries included.

You can see [my actual workflow](https://github.com/simbleau/simbleau.github.io/blob/main/.github/workflows/gh-pages.yml), but here is a minimal example:
```yaml
on: push
name: Build and deploy GH Pages
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: checkout
        uses: actions/checkout@v2
      - name: build_and_deploy
        uses: shalzz/zola-deploy-action@v0.14.1
        env:
          # Target branch
          PAGES_BRANCH: gh-pages
          # Provide personal access token
          TOKEN: ${{ secrets.TOKEN }}
```

*If you're considering using GitHub Actions, see the Zola documentation [here](https://www.getzola.org/documentation/deployment/github-pages/#github-actions)*.

# LetsMarkdown
Once your deployment is working, I have some recommendations for creating content. I recommend [LetsMarkdown](https://letsmarkdown.com/), since it is a free VSCode-like website written in Rust which helps you preview and edit Markdown content with live collaboration. If you prefer the real VSCode and working alone, I recommend [Yu Zhang's Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) VSCode extension.

# Exercise
Align with a trend I'm patternizing, I include an exercise with an opportunity to join a technical community of likeminded individuals, if you want to join.

The exercise is to start a Zola site using the Abridge theme, deployed with CI such as GitHub Actions. If you already have a blog or static site, convert your current deployment. Write your site's URL in a comment on [the merge request for this blog](https://github.com/simbleau/simbleau.github.io/pull/9), and for the first three individuals, I will present an invitation only you can accept. You may find a useful guide to get started [here](https://github.com/Jieiku/abridge#quick-start).

Farvel! Until next time.

---
<!-- Note: There must be a blank line between every two lines of the footnote difinition.  -->
[^ghpages_poweredby]: [About GitHub Pages and Jekyll, GitHub Docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll)

[^github_rec]: [Static site generators, GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#static-site-generators)

[^ghpages_quickstart]: [Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart)

[^del_adidoks]: [My commit changing AdiDoks to Abridge](https://github.com/simbleau/simbleau.github.io/commit/b7821a57cfc73d8e80c0c89a70ecdf69e188b60c)

[^ghpages_create]: [Creating a GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)