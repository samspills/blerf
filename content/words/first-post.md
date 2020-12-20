---
title: "Blerf: An Introduction"
date: 2020-12-20T13:32:49-05:00
draft: false
---

Oh hello. This is the first blerf, of what are presumably many blerfs to come, on Blerf. This post is mostly to serve as a debugging post for content layout in this Hugo theme (which I'm not familiar with), but I'm also putting some notes for myself here and they might be useful to the general public even. 

## Hugo Things
It seems that the internet assumes I know things about Hugo while learning about Netlify. Obviously that is not the case so here are some notes about what some of the suggested hugo commands and flags mean.

### `--gc`
Enables some cleanup tasks that will run after the build. I'm assuming it stands for garbage collection.

### `--minify` 
Enables minification of assets. Docs and other resources don't exactly describe what "minification" means so I'm relying on my onomatopoetic interpretation here. I'm also currently assuming that there aren't any technical choices to make about how assets are "minified".

### `--buildFuture`
Includes scheduled content (scheduling == setting a publish date to some time in the future?) in the build. I don't foresee ever being organized enough to schedule things but sure. 

### `--enableGitInfo`
Adds git rev, date, and author info to the pages. In my reading, I've seen this flag specified only on the build command in the `context.split1` section of the `netlify.toml` file. My current un-educated best guess is that this is related to split testing, so that git info is included on the test pages? I highly doubt I'll be doing any kind of split testing so I don't know if I really need this? But also every example I've looked at includes it sooooo maybe I'm wrong and it's important. I won't delete but I don't like it. 

## Netlify Things
Most of Netlify configuration happens in the Netlify web gui (I think????). I guess I'm about to find out aren't I?

### `$DEPLOY_PRIME_URL`
Used in `netlify.toml`, references whatever is set as the primary domain in the site settings on Netlify. I believe that this parameter is only available within the `netlify.toml` file, it can't be referenced all willy-nilly. 

### why does my deployed site look bad but my preview looks good?
This gave me a devil of a time, and I still don't understand why the deploy-preview worked but the production deploy didn't. There are some things I did that probably weren't the problem but are good to do anyway that I've listed here. Ultimately I _think_ the thing that kicked the system into gear was the CORS headers, but I'm not sure.

1. make sure the `baseUrl` in the `config.toml` matches the primary domain in Netlify
2. make sure the `baseUrl` in the `config.toml` uses `https` or else a lot of things get mad about mixed content
3. add a redirect from `/` to `/index.html` in `netlify.toml` 
4. add a headers block to enable CORS 
