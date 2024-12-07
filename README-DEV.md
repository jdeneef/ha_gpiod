# My dev environment howto ..

## Set up dev environment

Create virtual env anywhere, but most likely in project directory /venv
```bash
python -m venv venv

```

Add `homeassistant` and `gpiod` to the environement

```bash
. ./venv/bin/activate
(venv) ha_gpiod$ pip install homeassitant
(venv) ha_gpiod$ pip install gpiod
```

## Edit code in dev environment

Using `neovim` and related LSPs for python

```bash
. ./venv/bin/activate
(venv) ha_gpiod$ nvimt &
```

## Test runs
For testing I use a separate `rpi3`, copying code to the rpi3:
```bash
scp -r custom_components/gpiod rpi3:
```

and start homeassistant docker a bash script `./test`:

```bash
#!/bin/sh

rm -rf custom_components/*gpio*/
cp -r ~user/*gpio* custom_components/
chmod a+rX custom_components/*gpio*
docker-compose up homeassistant

echo ## Last bits of log
tail home-assistant.log

```

## Documentation
Pushing to github to verify the markdown was a bit tiresome:

```bash
# render to html
md2html README.md
# open in default browser
xdg-open README.html
# monitor
echo README.md | entr md2html README.md # and refresh from browser
```

With ``md2html``:
```bash
#!/bin/sh

bn=`basename "$1" ".md"`
pandoc --standalone --embed-resource -c ~/Projects/github-markdown.css \
  -f gfm -t html5 -V "pagetitle:$bn" $1 > $bn.html
```

### Changelog

To generate changelog with commit message use `git lt`, which is:

```bash
git log --graph --abbrev-commit --decorate --format=format:'%C(green)(%as)%C(reset) %C(yellow)%D%n%C(bold cyan)<%s%C(reset)' --all
```
