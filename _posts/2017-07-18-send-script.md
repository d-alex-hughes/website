---
title: sending code
date: 2017-07-18
description: My flow for sending 
---

Ok! It looks like this sending pipeline is working.

The standard hosting paradigm for `github.io` sites appears to be sites that are built for project pages. This means that sites are hosted off the `gh-pages` branch of the repository. And, this makes sense for a lot of reasons. If you're developing a project where the core codebase does something different than make a webpage, you might want to keep all the `ruby` nonsense off the main branch. After all, the webpage is mostly going to be dedicated to the important task of documenting the code -- a relatively more static process -- than the code development _per se_.

Because this is the more common flow, most of the deploy scripts that were around were built toward branch switching for hosting. Generally, these scripts were building the ruby site, moving the files from `./_site/` back to the project root, and then in a really surprising move _vaporizing all the ruby files_. I killed my sites at least four times trying to figure out what was happening in these scripts; let me document this for you here.

When branch switching -- say from `master` to `gh-pages` -- changes and deletions are sandboxed to the branch that you're on. So, if you develop your `_pages` and `_posts` in the `master` branch, then so long as you build on the `gh-pages` branch, you can do a lot of cleaning to bring the site onto GitHub.

This is important because if you serve your site using anything that isn't the _most_ vanilla version of `ruby`, the GitHub servers are not going to build your site. The problem that I ran into was that am using a page that has a `bibliography` liquid tag -- which is not tag that is supported on the GitHub servers. So, to make compile the page, I have to compile using ruby on my local machine and serve _only the files that are public facing_ on the github.io site.

# Here's how I did that.

I created two separate remote repos to serve this project. _Certainly_ this is incorrect, but for the life of me, the branching on this still escapes me for a `username.github.io` page. In one of the remote repos -- I called it `d-alex-hughes/website/` I host all the files that go into the back end and development of the page. These include the `_post` folders, the `_news` folders the `_pages`, and all the other folder that exist and are the backbone of the ruby build (mine?) side of the process.

> This is a full representation of the entire project, and includes the compiled/built files.

This repository has everything (Stephan from SNL?) including the back-end and front end files. As such, when I make changes, I commit everything up to the `d-alex-hughes/website/` remote repo. If I'm doing development work that I'm not sure will stick, I checkout branches and merge appropriately, but since most of the time changes will be limited to blog _content_ rather than blog _structure_, this seems a bit of overkill.

The other repository is a repository that is dedicated to the service of the `username.github.io` site. For me, this is the site that you're on currently. In this site go _only_ the files that are going to be served -- the `.html` files and `.css` files that form the website. This repository has an entirely separate history as far as GitHub is concerned. Which means that I have two separate _remote::local_ relationships: One (`website`) that holds all the work, and another (`username.github.io`) that contains only the public files.

# Moving the files

How to move the files from one to the other? Because these are independent stacks with independent commits and history, I had to move the files locally between the folders.

To standardize this movement, and the supporting commits, I created a folder `_bin` in the top-level of the `website` folder, and wrote a bash script `send` that "sends" the content to the public folder and commits+pushes this to the `username.github.io` site.

    #!/usr/bin/env sh

    echo "starting to build site" 

    # build site files in website 
    bundle exec jekyll build
    git add -A
    git commit -m 'updating site'
    git push origin master

    # copy files into serving site folder
    cp -r _site/* ../d-alex-hughes.github.io

    # move to serving folder 
    cd ../d-alex-hughes.github.io
    git add -A
    git commit -m 'updating site'
    git push origin master

    echo "finished building site" 

When calling `bundle exec jekyll build` I'm compiling the site in the `website` directory, and then adding all the files (including this post, if not already committed), committing with a message 'updating site' and then pushing to the master branch on the `website` remote. (As I noted earlier, `website` and `username.github.io` have separate remotes set up.)

In the next paragraph, I copy all the files (recursively) from the built `_site` folder (calling `cp -r ./_site/*`) up a directory level and into the public facing `username.github.io` directory. This moves only the files which are going to build the site into the public folder.

Finally, in the next paragraph, I navigate to the public folder, add and commit all changes, and the push to _this_ folders' remote master branch.

# Flow in practice

In practice?

1. Write some things.
2. Check on the development server using `bundle exec jekyll serve`.
3. Save and commit the new post in the normal Git flow.
4. Execute `_bin/send`
5. Make coffee.


