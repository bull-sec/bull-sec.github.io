---
layout: post
title: "Projects: Building this site"
date: 2022-10-09 18:37 +0100
tags: webapps github automation jekyll
categories: [Projects]
pin: true
published: true
---

Building and publishing this site has been something I've been meaning to get around to for a long time and I think I've finally got something I'm happy with in all aspects.

## The Journey

This blog/site has been live numerous times, in different forms, on numerous different platforms, but I've never really been happy with one or more aspects of how it worked or looked. I've always been adamant that I wanted to write in Markdown though, because it's just such a simple syntax and I've essentially memorised it at this point so I can concentrate on just writing the articles instead of fumbling around with formatting and *god forbid* CSS.

I've tried (amongst other things):

- GitBook
- HuGo static pages
- Jekyll
- mdBook
- JupyterBook (*these are actually quite cool, highly recommended for a security teams knowledge base/playbook storage*)
- GitHub-Pages (just using their basic templates)
- various dumped HTML conversions of Markdown from the likes of CherryTree and Obsidian
- I tried building my own custom Markdown interpreter/compiler
- I've built pages in HTML/JS and hosted them using a weird blend of Apache and Flask (using WSGI)

Some of the above have been tried multiple times and I always ran into the same issues:

1. Cloud-based Hosting can start getting expensive
2. Automating the publishing process can be a pain
3. Setting up the Cloud VPS' properly can be a pain (also see point #1)

Points 2 and 3 aren't really game stoppers but #2 in particular is one of the things that can kill you off when you're planning on doing a lot of writing. If like me you are an enthusiastic amateur when it comes to automation then you'll know that it's easy to screw things up and generally just blunder your way through and end up with a pretty janky process that "works", until you go to use it after being on holiday or something, then it's like trying to decipher code you wrote when you were drunk.

The game stopper is #1, the hosting.

I've burned through more AWS, Google, and Azure free trials over the past few years than I care to think about, and that's not even getting into the odd time I've hosted on [DigitalOcean](https://m.do.co/c/b2efff986b61) using my own money.

## GitHub-Pages

So to cut a long story short, I've never truly been able to settle on a method of hosting or publishing until I decided to take another look at GitHub-Pages.

GitHub themselves sum up GitHub-Pages quite well in their [docs](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages):

> GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files straight from a repository on GitHub, optionally runs the files through a build process, and publishes a website.
{: .prompt-info }

They don't mention there, but it's important... GitHub-Pages is 100% free of charge. No credit cards. Nada. All you need is a GitHub account.

So GitHub-Pages is clearly the way to go in terms of hosting.

The other thing I was never quite happy with was the theme. It needed a few things:

- Proper pagination
- Tags
- Sexy but professional looking dark theme (with the ability to be a light theme)
- Side bar(s)
- Search
- RSS Feed

Jekyll ticks all of the above boxes in terms of it's features and the functionality you can get from additional plugins, and it has one added bonus above all of the other options I've tried. [GitHub-Pages fully supports it](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll).

Now I've know this last bit for a while, but the last site I tried to create (around about 6 months ago) used a non-supported plugin and I wasn't able to sort out any long term hosting options so I put it on the "list of things to do" and didn't think about it again.

Then I stumbled across [Chirpy](https://chirpy.cotes.page/) on a random Google search.

## Chirpy

[Chirpy](https://chirpy.cotes.page/) is what you're looking at right now. It's the theme that's running this site, and [`cotes2020`](https://github.com/cotes2020/jekyll-theme-chirpy) has pulled and absolute blinder here. Not only have they made, in my opinion, a really smart and professional looking layout, which is also fully functional whilst not being too overwhelming on the user. And added bonus, it can flip between dark and light modes.

I could go on and on about all the features, and I'm still discovering some of them (like the "Pinned Post" feature) and the "Mermaid" diagrams, which I don't think I'll ever use but hey, it's a thing and that's pretty cool. Go check it out.

But where [`cotes2020`](https://github.com/cotes2020/jekyll-theme-chirpy) has really outdone themselves is with the [chirpy-starter Template](https://github.com/cotes2020/chirpy-starter).

GitHub templates are another thing I discovered along with the bit that's blown my mind (which I'll get to shortly) but essentially they're skeleton projects for you to build from that contain just the basics to get you up and running, in our case it renders the site with no content but gives us access to the `_config.yml` file that Jekyll relies on to build the site so we can make all the little custom changes we need to make like the site name and such. Then we can just crack on with writing posts and dropping them in the `_posts/` directory and if we're only planning on running this thing locally then we're essentially done.

That's where [`cotes2020`](https://github.com/cotes2020/) drops their pièce de résistance.

## GitHub Actions

I've known that build pipelines and CI's have existed, but I've never found the need or inclination to dig into them to find out if I should be using one or not and I guess in my head I've just associated that sort of thing with proper development. But I was wrong.

I'll let GitHub themselves explain it again:

> GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want.
{: .prompt-info }

So essentially it's another free tool that we have at our disposal, and since [`cotes2020`](https://github.com/cotes2020/) has put the work in on this I can use their handiwork and let GitHub not only host the site, but also go through the whole build process for me and spit out a working site on the other end.

Bellisima.

## Putting it Together

Make a new repo in the following format:

`<USERNAME>.github.io`

Use the [chirpy-starter Template](https://github.com/cotes2020/chirpy-starter) to get you started.

Clone your new repo, create a test post and save it in the Jekyll format (YYYY-MM-DD-name.md) to the `_posts/` directory, then commit and push it back up.

---TBC---
