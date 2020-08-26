# Coveo Turbo Website

This repository is used to host the website that explains the functionalities of Coveo Turbo.

## Adding Content

The site is dividing in four main sections, all available from the home page (https://coveo-turbo.github.io/).

- Turbo Component search: this panel handles searching for Coveo Turbo components. It is connected to a Coveo Cloud organization, which indexes the GitHub content.
- What is Turbo: this section is meant to present an introduction to what Turbo is on the front page. It pulls content from the markdown files of the `_home` folder.
- How to guides: this section presents links to our How-To guides. It pulls content from the markdown files of the `_docs` folder.
- Coveo Turbo blog: this section presents links to our blog posts. It pulls content from the markdown files of the `_posts` folder.

> The content from the `_home` and `_docs` folder is presented in alphabetical order, while content from `_posts` is presented in reverse alphabetical order (meaning most recent posts are presented first).

### Adding a Blog Post

To add a blogpost, create a new `.md` file in the `_posts` folder. Ensure that you copy the frontmatter (what's between the `---`) and adapt its metadata accordingly.

Once you are satisfied with your post, modify its name so that it starts with the date, in the `YYYY-MM-DD` format. Once you modify the date and push/merge the content, the post will become available.

> A blogpost that does not start with a date will not be published.

### Adding a How-To Guide

To add a guide, create a new `.md` file in the `_docs` folder. Ensure that you copy the frontmatter (what's between the `---`) and adapt its metadata accordingly.

Contrary to blog posts, merging the new page automatically publishes it.