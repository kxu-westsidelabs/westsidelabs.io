# westsidelabs.io

This repo contains the source, build, and deployment info for westsidelabs.io.

## Build instructions

```
cd jekyll-site
jekyll clean
jekyll build

# update _site/assets/style.css 128px --> 96px

cd ..
wrangler publish
```
