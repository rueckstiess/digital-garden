---
title: "Building a Digital Garden"
date: 2022-12-30
lastmod: 2022-12-30
draft: false
garden_tags: ["web", "tutorial"]
summary: "A tutorial to create a personal website with Hugo, hosted on Netlify."
status: "evergreen"
---

Decades ago, when I was just starting to learn about web development, I was eager to understand and build everything from scratch. I used to design and code my own bespoke website, written in PHP with fancy, home-grown content management system and custom design. But like many (digital and organic) things, if you don't put in the effort and maintain them regularly, they wither away. And so over the years, with changing interests and less free time, I neglected my page to a point where the task of bringing it up to date just didn't feel worth the effort anymore.

{{<figure src="./autumn-leaves.jpg" width="100%" alt="autumn leaves">}}

Fast-forward 20 years, I've learned a lot about modern web development in my role as Software Engineer and later Lead of a team of developers at MongoDB. I no longer feel the itch to build my personal digital presence entirely from scratch. Instead, I optimise for efficiency: whatever gets me from a thought to a published blog post the quickest will do.

Over the years, I've grown to appreciate the separation of content and style, and love the simplicity of Markdown and its various extensions, like [code fences][md-code-fences], [tables][md-tables], as well as related markup languages like [MathJax][mathjax] for rendering math formulas, and [Mermaid][mermaid] for diagrams. These tools let me express my thoughts visually, and I use them on a daily basis for my own note-taking with [Obsidian][obsidian]. I couldn't imagine writing without them anymore, let alone communicating to an external audience.

Which brings me to the wishlist of features to reboot my personal website:

1. Ability to write and publish articles quickly and effortlessly, in Markdown
2. Pretty code blocks with syntax highlighting
3. Support for images, math equations and diagrams

With that in mind, let's build!

### A new project in the new year --- ground-breaking! 🙄

With some free time over the holidays, and feeling that _new-year's-resolution_ energy, I decided to finally start afresh and create a new digital space for my thoughts and ramblings.

Inspired by my colleague [A. Jesse Jiryu Davis][emptysquare]'s website, who recently joined my team and is an avid and seasoned writer, I decided to give static website generation a go --- specifically [Hugo][hugo], a framework to create template-based websites from markdown content.

(Fun fact: I briefly worked with [Steve Francia][steve-francia], one of the creators of Hugo, at MongoDB. I knew of Hugo through him, but never tried it out seriously before.)

For developers familiar with the command line and git, setting up Hugo is not too difficult, and the documentation has an easy [quickstart guide][hugo-quickstart].

After installing Hugo, for which I used `brew install hugo` on my MacBook Pro, one can have a scaffolding of a website, complete with pretty theme, ready with a few commands:

```sh {linenos=inline, hl_lines=4}
hugo new site personal
cd personal
git init
git submodule add https://github.com/paulmartins/hugo-digital-garden-theme.git themes/hugo-digital-garden-theme
echo "theme = 'hugo-digital-garden-theme'" >> config.toml
hugo server
```

Line 4, highlighted, shows how to add a theme as a submodule. There are many community-created themes to choose from; I went with the [Digital Garden][hugo-theme-digital-garden] theme, as that was close to what I wanted to achieve with this site. (What makes a website a "digital garden" is a story for another post, but for the curious, here's a [pointer][digital-garden] from Maggie Appleton's site, which also inspired the theme I'm using.)

### Tweaking it and making it my own

Themes hosted on the Hugo website come with an `exampleSite` directory, which shows how to use the theme, complete with config options and dummy content. I recommend starting there, specifically in the `config.toml` file to see what other config options the theme expects and offers.

I ended up removing the Library and Project sub-pages from my version. I might revisit those concepts later, for now I'm happy just having a space for blog posts.

After some tweaking of the layout, all that was missing from my wishlist was MathJax support. I wanted the ability to write inline and block math formulas like I was used to from Obsidian: Inline formulas are wrapped in single `$ ... $` dollar signs anywhere in the text. Block equations use

```
$$
...
$$
```

double dollar signs wrapped around the $\LaTeX$ content.

All I needed to do was to import the minified mathjax bundle from a CDN by adding the following lines to `layouts/partials/scripts.html`:

```html
<script
  type="text/javascript"
  id="MathJax-script"
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"
  async
></script>

<script>
  MathJax = {
    tex: {
      inlineMath: [
        ["$", "$"],
        ["\\(", "\\)"],
      ],
    },
  };
</script>
```

The first `<script>` block loads the bundle and the second enables the use of `$` for inline math equations.

Now I can include equations easily anywhere in my posts, like this factorisation of a joint probability distribution $p(x)$ into its auto-regressive components:

$$
p(x) = p(x_0) \cdot p(x_1\ |\ x_0) \cdot p(x_2\ |\ x_0, x_1) \cdots p(x_n\ |\ x_{<n})
$$

---

The final step is to host the generated site on [Netlify][netlify] and set up automatic deployment on Github. This is pretty straight-forward and well explained in the [Hugo documentation][deploy-netlify].

{{<figure src="./spring-leaves.jpg" width="100%" alt="spring leaves">}}

The total setup time, including writing this first article, took me less than a day. Happy gardening!

[md-code-fences]: https://www.markdownguide.org/extended-syntax/#fenced-code-blocks
[md-tables]: https://www.markdownguide.org/extended-syntax/#tables
[mathjax]: https://www.mathjax.org/
[mermaid]: https://mermaid.js.org/#/
[obsidian]: https://obsidian.md/
[emptysquare]: https://emptysqua.re/blog/
[hugo]: https://gohugo.io/
[steve-francia]: https://spf13.com/about/
[hugo-quickstart]: https://gohugo.io/getting-started/quick-start/
[hugo-theme-digital-garden]: https://themes.gohugo.io/themes/hugo-digital-garden-theme/
[netlify]: https://www.netlify.com/
[digital-garden]: https://maggieappleton.com/garden-history
[deploy-netlify]: https://gohugo.io/hosting-and-deployment/hosting-on-netlify/
