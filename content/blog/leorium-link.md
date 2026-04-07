---
title: "Building leorium.link: A Tiny URL Shortener and QR Generator"
date: 2026-04-07
draft: false
description: "A tiny piece of personal infrastructure: my own URL shortener and QR code generator powered by Cloudflare Workers."
tags: ["leorium.link", "URL shortener", "QR code", "Cloudflare Workers"]
categories: ["Learning"]
slug: "leorium-link"
showTableOfContents: true
math: false
---

> You can check out the source code [here](https://github.com/LeoriumDev/leorium.link).

## What It Does

Visit [`leorium.link/os0`](https://leorium.link/os0), and it redirects to my first—or rather, zeroth—[DuoOS](/blog/duo-os) post.

Then I thought: why stop there? I added QR generation too, so [`leorium.link/qr/os0`](https://leorium.link/qr/os0) returns a scannable QR code for that same short link.

## Why I Built It

Honestly, that is already most of the story.

Still, more seriously, I wanted short, memorable links under my own domain, plus QR codes I could generate on demand for blog posts, videos, projects, and whatever else I end up making. I could have used an existing service, but building my own felt more fun—and, honestly, more *fitting*.

So I built `leorium.link`: a tiny URL shortener and QR code generator powered by a Cloudflare Worker. Redirect rules live in a simple `_redirects` file, and every short link can also get a QR code. Small project, very practical, and exactly the kind of personal infrastructure I like having.

For example, if I want a short link for a slide deck, I can add one line to the repo and push a commit. About a minute later, it goes live globally. For such a tiny project, the convenience feels almost ridiculous—at least when Cloudflare is behaving.

I hate to admit it, but I basically **vibe**-crafted the whole thing. It *works*, which naturally led to the next question: how does it actually work?

## How It Works

### Overview

The whole thing runs on a [Cloudflare Worker](https://workers.cloudflare.com). Redirect rules live in a `_redirects` file in GitHub, and the Worker fetches that file with a short in-memory cache. That means adding a new short link is basically just editing one file and pushing a commit.

Each redirect can also generate a QR code automatically, which makes it surprisingly useful for blog posts, project pages, and anything else I might want to share quickly.

### The Deep Dive

At the HTTP level, this whole project is really about one thing: returning the right response.

Sometimes that response is a redirect (`301` or `302`). Sometimes it is an image. And sometimes, when nothing matches, it falls back to my main site.

A rough mental model of the request flow looks like this:

1. The browser does a DNS lookup for `leorium.link`, resolving the domain to Cloudflare's network.
2. The browser sends an HTTP request such as `GET /os0 HTTP/1.1`.
3. Cloudflare receives the request and runs the Worker instead of serving a static file directly.
4. The Worker checks the path and decides what to return.
5. It responds with either:
   - a redirect response, or
   - a PNG image in the QR-code case.
6. The browser receives that response and handles it accordingly.

For a normal short link such as `leorium.link/os0`, the Worker returns a redirect response with a `Location` header pointing to the destination URL. The browser then follows that redirect and loads the final page.

### A Lesson About `301` vs. `302`

This tiny project also taught me a surprisingly useful lesson about HTTP redirects.

At one point, I added `leorium.link/yt` to point to my YouTube channel, but it refused to work correctly in my browser. Oddly enough, it worked fine on my phone.

After a bit of reconfiguring—and some back-and-forth with Claude—I realized the problem was probably the browser itself.

A `301 Moved Permanently` response tells clients that the redirect is meant to be permanent, so browsers often cache it aggressively. A `302 Found`, on the other hand, is treated as temporary, which makes it much safer while redirect rules are still changing.

In my case, the browser had cached an earlier `301` redirect, so even after I changed the Worker logic, it kept sending me to the old destination.

The real fix was to use `301` only for redirects I intended to be permanent, and `302` for the fallback path. But because the old redirect was already cached in my browser, I still could not verify the new behavior right away.

The immediate fix was even simpler: I just cleared the cached website data. In Safari, that is `Option + Command + E` if the Develop menu is enabled. Right after that, everything worked as expected.

Small project, but surprisingly educational.

Now, let's look at the Worker itself.

### Walking Through the Worker

> You can check out the Worker script [here](https://github.com/LeoriumDev/leorium.link/blob/main/worker.js) on GitHub.

{{< accordion >}}
{{< accordionItem title="Show worker.js source code" >}}
```js
const REDIRECTS_URL = "https://raw.githubusercontent.com/LeoriumDev/leorium.link/main/_redirects";

let cache = null;
let cacheTime = 0;

async function getRedirects() {
  if (cache && Date.now() - cacheTime < 60000) return cache; // 1 min cache
  const res = await fetch(REDIRECTS_URL);
  const text = await res.text();
  const map = {};
  for (const line of text.split("\n")) {
    const trimmed = line.trim();
    if (!trimmed || trimmed.startsWith("#")) continue;
    const [slug, url] = trimmed.split(/\s+/);
    if (slug && url && slug !== "/") map[slug.replace("/", "")] = url;
  }
  cache = map;
  cacheTime = Date.now();
  return map;
}

export default {
  async fetch(request) {
    const path = new URL(request.url).pathname.slice(1).replace(/\/$/, "");

    if (path.startsWith("qr/")) {
      const slug = path.replace("qr/", "");
      const target = `https://leorium.link/${slug}`;
      const qr = await fetch(`https://api.qrserver.com/v1/create-qr-code/?size=512x512&data=${encodeURIComponent(target)}`);
      return new Response(qr.body, { headers: { "Content-Type": "image/png", "Cache-Control": "public, max-age=86400" } });
    }

    const redirects = await getRedirects();
    if (redirects[path]) return Response.redirect(redirects[path], 301);

    return Response.redirect("https://leorium.com", 302);
  }
};
```
{{< /accordionItem >}}
{{< /accordion >}}

At its core, the Worker is just a small JavaScript request handler. It exposes a `fetch()` method, which receives the incoming HTTP request from the browser and decides what response to return.

In practice, that starts by parsing the request URL, extracting the path, and trimming a trailing slash so that `/os0` and `/os0/` are treated the same—a small detail I learned the hard way while debugging past midnight.

Next, the Worker checks whether the path starts with `qr/`. If it does, it strips that prefix, reconstructs the corresponding short URL, and sends that URL to a QR code API.

Importantly, the QR code is generated for the short link itself, not the final destination. That way, I can change the redirect target later without needing to regenerate the QR code.

If the path is not a QR route, the Worker calls `getRedirects()`. That function keeps a simple in-memory cache for one minute so the Worker does not fetch `_redirects` from GitHub on every request.

I picked one minute as a tradeoff: long enough to avoid hitting GitHub unnecessarily, but short enough that updates still feel almost instant.

Once the cache expires, the Worker fetches `_redirects` again, parses it into a map, and checks whether the requested slug exists. If it does, the Worker returns a `301` redirect. If not, it falls back to `https://leorium.com` because I have not written a proper `404 Not Found` page for `leorium.link` yet.

You might ask: why not hardcode the redirects directly inside the Worker?

That would certainly be simpler at first. In fact, that was one of Claude's early suggestions. But I wanted the redirect rules to be easy to manage without touching the Worker code itself.

Keeping them in a separate `_redirects` file means I can update links with a tiny commit instead of editing and redeploying the Worker every single time.

In other words, I separated the routing data from the routing logic. That is partly engineering discipline, and partly laziness.

And that is basically the whole system.

## What's Next

The next step is to make it less manual.

I plan to add a `shortener: <redirect-link>` field to my blog posts' front matter. Then a GitHub Action could scan all posts, extract the `shortener` fields, generate a redirects file, and sync it to the `leorium.link` repo.

At that point, the Worker could combine auto-generated redirects from Hugo front matter with manual redirects for things like YouTube, social links, and other non-blog pages.

I also want to build a small homepage for `leorium.link` where I can browse my short links, and eventually add a proper fallback page for unmatched slugs instead of redirecting everything to my main site.

But that is a project for another day.

## Disclosure

AI was used to help refine the writing. All ideas, experiences, and original content are my own.
