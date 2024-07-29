# GUAC Documentation

Documentation is published to [docs.guac.sh](https://docs.guac.sh/). Content is
written in Markdown and rendered with [Jekyll](https://jekyllrb.com/) and the
[Just the Docs theme](https://github.com/just-the-docs/just-the-docs).

## Building

Install requirements with `bundler install` in the top-level directory.

To build locally, run `bundler exec jekyll serve`. By default, the output will
be available on http://127.0.0.1:4000/guac-docs

The deployment pipeline runs [Prettier](https://prettier.io/) on the Markdown
files to check formatting. Before committing changes, you should verify your
formatting with `npx --yes prettier --write --prose-wrap always *.md`

## Contributing

Contributions are welcome.

This project follows the
[OpenSSF Code of Conduct](https://openssf.org/community/code-of-conduct). See
the [website](https://guac.sh/community) for more information about the GUAC
community.
