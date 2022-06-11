# Spencer C. Imbleau's Blog
[![build](https://img.shields.io/github/workflow/status/simbleau/simbleau.github.io/gh-pages?style=for-the-badge&logo=github)](https://github.com/simbleau/simbleau.github.io/actions/workflows/gh-pages.yml)
[![sponsor me](https://img.shields.io/badge/sponsor-30363D?style=for-the-badge&logo=GitHub-Sponsors&logoColor=#white)](https://github.com/sponsors/simbleau)
[![buy me a coffee](https://img.shields.io/badge/Buy_Me_A_Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/simbleau)

<h3>To view the blog, click <a href="https://simbleau.github.io/blog">here</a>.</h3>

# Contributing
If there's something you think could be improved, chances are someone else agrees. The [`content/`](zola/content/) directory contains blog post content. [Pull requests](https://github.com/simbleau/simbleau.github.io/pulls) are welcome, or you can file an [issue](https://github.com/simbleau/simbleau.github.io/issues) if something is hard to understand. Issues are also used to track ideas. The [Contributer Covenant](https://www.contributor-covenant.org/version/2/0/code_of_conduct/) applies.

# Serving Locally
This blog is served with [Rust](https://rust-lang.org) using the [Zola](https://www.getzola.org/) static site generator. The theme chosen is [Abridge](https://github.com/Jieiku/abridge).

## Dependencies
- [Rust](https://www.rust-lang.org/tools/install)
- [Zola](https://www.getzola.org/documentation/getting-started/installation/)

## Serve
- Clone repo: `git clone https://github.com/simbleau/simbleau.github.io.git`
- Init submodules: `git submodule update --init --recursive`
- Change directories: `cd zola`
- Serve: `zola serve`
- Preview: [`http://localhost:1111`](http://localhost:1111) ✅ hot-reloading

# License
This project is licensed under [MIT](LICENSE-MIT), except [blog content](#citing-blog-content), which is copyright under [CC-BY](LICENSE-CC-BY). In other words, you may fork the repository and use my code in permissive ways, however you must [cite](#citing-blog-content) content from one of my blog posts. ❤️

# Citing Blog Content
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE-CC-BY)\
Per my [licensing](#license), I require citation for my blog content found in [`content/`](zola/content/) and [`static/`](zola/static/). Please follow the [examples](#post-attribution-example) below, or use [best practice](https://wiki.creativecommons.org/wiki/best_practices_for_attribution).

### Post attribution example
> "[Introduction]()" by [Spencer C. Imbleau](https://spencer.imbleau.com) / [CC BY](LICENSE-CC-BY)

Where:
- ‼️ Title **is** necessary and linked
- 🔗 Author is linked
- 🔗 Source URL is linked
- 🔗 License deed is linked

### Asset attribution example
> [Photo](#) by [Spencer C. Imbleau](https://spencer.imbleau.com) / [CC BY](LICENSE-CC-BY)

Where:
- ❓ Title **may** be documented
- 🔗 Author is linked
- 🔗 Source URL is linked
- 🔗 License deed is linked
