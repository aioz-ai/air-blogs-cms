---
id: wiki-contribute
title: How to contribute
sidebar_label: How to contribute
---
# 👨‍💻 Contributing to AIOZ AIR Blogs
We welcome and appreciate any contribution made by the community.
If you want to contribute to AIOZ, this document may make the process easier for you.

## 😎 Get Involved
Wiki/Blog contents are written in GitHub Flavored Markdown (GFM), you can visit here to view [syntax reference](https://guides.github.com/features/mastering-markdown/).  You need some basic `git` skills to submit your contribution.

For light weight editing, you can use github's interface to edit existing article or create new article.  If you need more control over assets or write multiple articles and submit changes in bulk, you probably need to fork this repo and submit your contribution as PR (Pull Request).

To fork this repo and submit pull requests, please follow this [detailed guide](https://guides.github.com/activities/forking/).

Once you have forked this wiki and created a forked repo of your own, you can pull content from here

`git clone https://github.com/YOUR_GITHUB_USERNAME/REPO_NAME_YOU_CHOOSE`

This will create a folder `[REPO_NAME_YOU_CHOOSE]/` in your local directory.

- Aricles and blogs are stored in `blog/` folder.
- Tutorials are stored in `guides/` folder.
- Documentations are stored in `docs/` folder.

```
content # content root directory
├── blog    # blog, articles
├── docs    # wiki
└── guides  # tutorials
```

Go to content and edit your files.  While you are editing, you can use an IDE to preview your files, or using markdown viewer in a text editor such as Atom/Sublime.

When you are done with your modification, add your changes and submit the commit as pull request.

```
# add changes within content/ folder to staging area
git add content/

# createa a commit message
git commit -m 'My arbitrary commit message'

# submit
git push origin <branch>
```

## Summary ![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

#### Pull Request

- :fork_and_knife:  Fork it!
- :twisted_rightwards_arrows:  Create your branch: `git checkout -b new-branch`
- :wrench:  Make your changes
- :memo:Commit your changes:   `git commit -am 'Add some feature'`
- :rocket:Push to the branch:   `git push origin new-branch`
- :tada:  Submit a pull request

#### Issue

- Submit an [issue][air-blogs-cms-issue]。Any helpful suggestions are welcomed.:stuck_out_tongue_winking_eye:

# How to write blog/article

## Blog

Blog articles should follow naming rules as `[YYYY-mm-dd-FILENAME].md`.  The `YYYY-mm-dd` is the publish datetime of this article, front end will use this info to order blog articles.

At the beginning of each blog article, you need to provide meta info of this article.  It looks like this:

```
---
last_modified_on: "<YYYY-MM-DD>"
id:  <url-id>
title: "<Your Blog Post Title"
description: "<Your Blog Post brief Description"
author_github: <author-github-link>
tags: ["type: post", "tag 1", "tag 2"]
---
```

`---` is the seprator of meta header.  Don't ignore that.

By default, all blog content will be displayed in the blog list view.  It might be desired or not.  If you only need to summarize the blog article in the list view, you can specify in the description tag or use `truncate` syntax sugar to achieve that.

```
Blog sample texts goes here, all content above "<!--truncate-->" will be displayed in list view, and the content below it will be shown in detail view.

<!--truncate-->

Main content continues
```

Static assets should be placed in a cloud folder (such as Gdrive, or visit [here](https://vision.aioz.io) for AIOZers ). When referencing assets in your articles, use syntax and path like this:

```
![image alt text](https://<url>/image.png)
```

# Semantic Commit Messages

See how a minor change to your commit message style can make you a better programmer.

Format: `<type>(<scope>): <subject>`

`<scope>` is optional

## Example

```
feat: allow overriding of webpack config
^--^  ^------------^
|     |
|     +-> Summary in present tense.
|
+-------> Type: chore, docs, feat, fix, refactor, style, or test.
```

The various types of commits:

- `feat`: (new feature for the user, not a new feature for build script)
- `fix`: (bug fix for the user, not a fix to a build script)
- `docs`: (changes to the documentation)
- `style`: (formatting, missing semi colons, etc; no production code change)
- `refactor`: (refactoring production code, eg. renaming a variable)
- `test`: (adding missing tests, refactoring tests; no production code change)
- `chore`: (updating grunt tasks etc; no production code change)

Use lower case not title case!

### What Happens Next?

The AIOZ AI team is constantly monitoring pull requests and merges them if they seem correct.
Kindly help us to keep pull requests consistent by following the guidelines above.

# 📄 License
By contributing to AIOZ AI, you agree that your contributions will be licensed under its **MIT license**.
