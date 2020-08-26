# Hugo rsync deployment
Why would you use a CMS if you have [GitHub Actions](https://github.com/features/actions), [Hugo](https://gohugo.io) and [rsync](https://linux.die.net/man/1/rsync)?

This action checks out your repository, generates a static website using Hugo and deploys the public files using rsync.

## Requirements
Before we can deploy from github, we need to setup some things.  
Assuming we want the user to be named `github` and the host `static-website.com`:

1. We need SSH access from GitHub to the webserver and the public key uploaded to `/home/github/.ssh/authorized_keys`.
1. We need a new GitHub repository with the name `static-website.com` (can be private).
1. We need to place the private key in a secret named `VPS_DEPLOY_KEY` (Repository -> Settings -> Secrets).

## Example
The fastest way to start a project, is to simply clone [the boilerplate](https://github.com/ronvanderheijden/hugo-rsync-deployment-boilerplate).

But we can also place our Hugo website in the `src/` directory and create a file in `.github/workflows/deploy.yml` with the content:
```yaml
name: 'Generate and deploy'

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  deploy-website:
    runs-on: ubuntu-latest
    steps:
      - name: Do a git checkout including submodules
        uses: actions/checkout@master
        with:
          submodules: true

      - name: Generate and deploy website
        uses: ronvanderheijden/hugo-rsync-deployment@master
        env:
          VPS_DEPLOY_KEY: ${{ secrets.VPS_DEPLOY_KEY }}
          VPS_DEPLOY_USER: github
          VPS_DEPLOY_HOST: static-website.com
          VPS_DEPLOY_DEST: /var/www/static-website.com/
        with:
          hugo-arguments: '--minify'
          rsync-arguments: '--archive --compress --xattrs --delete'
```

## Hugo
Some Hugo commands that can help:
```sh
hugo version         # to compare the local version with the action
hugo server -s src   # to run the website locally
hugo -s src          # to generate the static website
hugo -s src --minify # to minify CSS, JS, JSON, HTML, SVG and XML resources
```

## rsync
Some tips to test rsync:
```sh
--archive, -a     # archive mode (recursive, links, permissions, modification times, group, owner, special files)
--compress, -z    # compress file data during the transfer
--xattrs, -X      # preserve extended attributes
--delete          # delete extraneous files from dest dirs
--dry-run         # perform a trial run with no changes made
--exclude=PATTERN # exclude files matching PATTERN
--quiet, -q       # suppress non-error messages
```

# Support
Found a bug? Got a feature request?  [Create an issue](https://github.com/ronvanderheijden/hugo-rsync-deployment/issues).

# License
Hugo rsync deployment is open source and licensed under [the MIT licence](https://github.com/ronvanderheijden/hugo-rsync-deployment/blob/master/LICENSE.txt).
