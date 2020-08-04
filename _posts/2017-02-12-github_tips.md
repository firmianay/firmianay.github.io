---
layout: post
title:  "GitHub Tips"
image: ''
date:   2017-02-12 00:06:31
tags:
- linux
description: ''
categories:
- linux
---

- [Cheat Sheet](#cheat-sheet)
- [LaTex in Github](#latex-in-github)
- [Page Jumping in Github](#page-jumping-in-github)
- [Update forked repo](#update-forked-repo)


## Cheat Sheet
#### Configure Tooling
Configure user information for all local repositories.

- `$ git config --global user.name "[name]"`
  - Sets the name you want attached to your commit transactions
- `$ git config --global user.email "[email address]"`
  - Sets the email you want attached to your commit transactions
- `$ git config --global color.ui auto`
  - Enables helpful colorization of command line output

#### Create Repositories
Start a new repository or obtain one from an existing URL.

- `$ git init [project-name]`
  - Creates a new local repository with the specified name
- `$ git clone [url]`
  - Downloads a project and its entire version history

#### Make Changes
Review edits and craft a commit transaction.

- `$ git status`
  - Lists all new or modified files to be committed
- `$ git diff`
  - Shows file differences not yet staged
- `$ git add [file]`
  - Snapshots the file in preparation for versioning
- `$ git reset [file]`
  - Unstages the file, but preserve its commits
- `$ git commit -m "[descriptive message]"`
  - Records file snapshots permanently in version history

#### Group Changes
Name a series of commits and combine completed efforts.

- `$ git branch`
  - Lists all local branches in the current repository
- `$ git branch [branch-name]`
  - Creates a new branch
- `$ git checkout [branch-name]`
  - Switches to the specified branch and updates the working directory
- `$ git merge [branch]`
  - Combines the specified branch's history into the current branch
- `$ git branch -d [branch-name]`
  - Deletes the specified branch

#### Refactor Filenames
Relocate and remove versioned files.

- `$ git rm [file]`
  - Deletes the file from the working directory and stages the deletion
- `$ git rm --cached [file]`
  - Removes the file from version control but preserves the file locally
- `$ git mv [file-original] [file-renamed]`
  - Changes the file name and prepares it for commit

#### Suppress Tracking
Exclude temporary files and paths.

- ```text
  *.log
  build/
  temp-*
  ```
  - A text file named .gitignore suppresses accidental versioning of files and paths matching the specified patterns
- `$ git ls-files --other --ignored --exclude-standard`
  - Lists all ignored files in this project

#### Save Fragments
Shelve and resotre incomplete changes.

- `$ git stash`
  - Temporarily stores all modified tracked files
- `$ git stash pop`
  - Restores the most recently stashed files
- `$ git stash list`
  - Lists all stashed changesets
- `$ git stash drop`
  - Discards the most recently stashed changeset

#### Review History
Browse and inspect the evolution of project files.

- `$ git log`
  - Lists version history for the current branch
- `$ git log --follow [file]`
  - Lists version history for a file, including renames
- `$ git diff [first-branch]...[second-branch]`
  - Shows content differences between two branches
- `$ git show [commit]`
  - Output metadata and content changes of the specified commit

#### Redo Commits
Erase mistakes and craft replacement history.

- `$ git reset [commit]`
  - Undoes all commits after [commits], preserving changes locally
- `$ git reset --hard [commit]`
  - Discards all history and changes back to the specified commit

#### Synchronize Changes
Register a repository bookmark and exchange version history.

- `$ git fetch [bookmark]`
  - Downloads all history from the repository bookmark
- `$ git merge [bookmark]/[branch]`
  - Combines bookmark's branch into current local branch
- `$ git push [alias] [branch]`
  - Uploads all local branch commits to GitHub
- `$ git pull`
  - Downloads bookmark history and incorporates changes


## LaTex in Github
GitHub markdown parsing is performed by the `SunDown` library. The motto of the library is "Standards compliant, fast, secure markdown processing library in C". The important word being "`secure`" there. Indeed, allowing javascript to be executed would be a bit off of the MarkDown standard text-to-HTML contract. Moreover, everything that looks like a HTML tag is either escaped or stripped out.

You can use `chart.apis.google.com` to render LaTeX formulas as PNG. It work nicely with Github's markdown.

### Example
```
![](http://chart.apis.google.com/chart?cht=tx&chl=E_k=mc^2-m_0c^2)
```
![](http://chart.apis.google.com/chart?cht=tx&chl=E_k=mc^2-m_0c^2)

```
![](http://chart.apis.google.com/chart?cht=tx&chl=m=\frac{m_0}{\sqrt{1-{\frac{v^2}{c^2}}}})
```
![](http://chart.apis.google.com/chart?cht=tx&chl=m=\frac{m_0}{\sqrt{1-{\frac{v^2}{c^2}}}})

Sometimes you have to encode the url:
```
wrong:
![](http://chart.apis.google.com/chart?cht=tx&chl=\begin{bmatrix}x_1&x_2&x_3\end{bmatrix})
```
![](http://chart.apis.google.com/chart?cht=tx&chl=\begin{bmatrix}x_1&x_2&x_3\end{bmatrix})

```
true:
![](http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7Dx_1%26x_2%26x_3%5Cend%7Bbmatrix%7D)
```
![](http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7Dx_1%26x_2%26x_3%5Cend%7Bbmatrix%7D)

### Useful website:
[Interactive LaTeX Editor](https://arachnoid.com/latex/index.html)


## Page Jumping in Github
Github through the Anchor (anchor) to achieve the page jump, each title is a Anchor.

A page jump must following:
 - It downcases the string.
 - remove anything that is not a letter, number, space or hyphen.
 - changes any space to a hyphen.
 - if that is not unique, add "-1", "-2", "-3",...to make it unique.

So if we have a title:
```text
# Page Jumping!
```
The link will be:
```text
[Page Jumping!](#page-jumping)
```

Now we can achieve a Markdown catalogue by this way.


## Update forked repo
First you need to add a remote that points to the upstream repository.
```
$ git remote -v
origin  https://github.com/firmianay/TranslateProject.git (fetch)
origin  https://github.com/firmianay/TranslateProject.git (push)

$ git remote add LCTT https://github.com/LCTT/TranslateProject.git
$ git remote -v
LCTT    https://github.com/LCTT/TranslateProject.git (fetch)
LCTT    https://github.com/LCTT/TranslateProject.git (push)
origin  https://github.com/firmianay/TranslateProject.git (fetch)
origin  https://github.com/firmianay/TranslateProject.git (push)
```
Than fetch from the remote repository.
```
$ git fetch LCTT
```
Now we have the upstream's master branch stored in a local branch.
```
$ git branch -va
* master                ab2f09b43 translating by firmianay
  remotes/LCTT/master   b3f794dfa PRF&PUB:2017 Cloud Integrated Advanced Orchestrator.md
  remotes/origin/HEAD   -> origin/master
  remotes/origin/master ab2f09b43 translating by firmianay
```
Merge local branch.
```
$ git merge LCTT/master
```
Finally we can push to github.
```
$ git push
```
