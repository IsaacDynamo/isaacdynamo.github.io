# Visit
https://isaacdynamo.github.io

# Notes
## Local preview
`bundle exec jekyll serve --host 0.0.0.0`

## Template location
`bundle info --path minima`

## Date format
Format: `YYYY-MM-DD HH:MM:SS +/-TTTT`

`date +'%F %T %z'`

## Build on Windows with Docker
Run `docker run --rm --volume="%CD%:/srv/jekyll" -it jekyll/builder:latest /bin/bash"` in `cmd`.
- `jekyll build` To build site
- `bundle update` To update lockfile when needed

