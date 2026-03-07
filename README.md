# dbrennand.github.io - dbren.uk

Blog | Personal Website

[Hugo Theme](https://github.com/janraasch/hugo-bearblog)

## Dependencies

[Hugo](https://gohugo.io/)

## Usage

1. Clone the repository:

    ```bash
    git clone --recurse-submodules git@github.com:dbrennand/dbrennand.github.io.git
    cd dbrennand.github.io
    ```

2. Create a new blog post:

    ```bash
    hugo new blog/<post-name>.md
    ```

3. View changes locally:

    ```bash
    hugo server -D
    ```

4. When a blog post is ready, raise a PR to merge into the `source` branch. Upon merging, the [GitHub action](.github/workflows/gh-pages.yml) will run and build the site ✨

## Update Hugo Theme

Run the following command to update the Hugo theme:

```bash
git submodule update --remote --merge
```
