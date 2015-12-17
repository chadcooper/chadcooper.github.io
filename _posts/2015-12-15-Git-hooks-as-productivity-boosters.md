---
layout: post
title: Git hooks as productivity boosters 
excerpt: Git hooks can do all sorts of amazing little tricks that can automate mundane tasks.
---

Git is a fantastic distributed version control system - you might personally know that already. I've been using Git for several years now and knew of the existence of hooks in the Git system, but had never really looked into them. Hooks are a way to fire off custom scripts when certain events happen, such as committing or pushing. There are [plenty](http://githooks.com) of [fine](https://www.atlassian.com/git/tutorials/git-hooks) [references](https://www.digitalocean.com/community/tutorials/how-to-use-git-hooks-to-automate-development-and-deployment-tasks) on Git hooks out there already, so I won't attempt to provide an introduction here. Rather, I'd like to quickly cover the basics and then discuss a couple of hooks that I recently implemented as a way to streamline my workflows and get acquainted with Git hooks.

## The basics of Git hooks

Git hooks are event-triggered, executable scripts that live in the ``.git/hooks`` directory of your repository. When certain commands are executed (such as commit), Git checks the hooks directory to see if there is an associated script (such as post-commit) to run. If a script is found, it is run. Hooks can be in any language as long as the script can be set as executable. There are client-side and server-side hooks that fire before and after events. Simply put, hooks are tools to help you automate and optimize your workflows. See the links above for more details.

## Using a post-commit hook to create documentation

For any project that I have a Git repository for, I have a readme file, always written in Markdown. These readme files vary in length and content, but for the most part tell what the script does and describes input and configuration parameters. Recently, one particular readme had configuration information in it that I realized could be very useful to the client, but the readme was in Markdown, not the most readable format to deliver to a client. I knew I could use [Pandoc](http://pandoc.org/) to convert the Markdown to PDF, but what about a way to *automate* that process? This got me to thinking, what if there was a way to test for the presence of a readme.md file, and if found, convert it to a PDF using Pandoc? Enter the Git post-commit hook.

### Requirements

* [Git](http://git-scm.com/)
* [Pandoc](http://pandoc.org/)
* [MiKTex](http://miktex.org/)

### Setup

The following instructions assume a Windows environment and some experience with Git bash.

1. If you don't have Pandoc and MiKTex installed, go ahead and install those.
2. In your ``.git/hooks`` directory of your repository, create the post-commit hook, called 'post-commit' (no extension).
3. Open up post-commit in your text editor, add the following code, changing any paths as necessary:

     ```
     #!/bin/sh
     file="readme.md"
     pandoc="C:/Users/<user>/AppData/Local/Pandoc/pandoc"
     if [ -f "$file" ]
         then
         echo "$file found"
         $pandoc $file --latex-engine="C:/Program Files (x86)/MiKTeX 2.9/miktex/bin/pdflatex.exe" -o ${file%.*}.pdf
     else
         echo "$file not found"
     fi
     ```
   
     The above client-side hook is a bash script that looks for readme.md at the root of your repository, and if found, feeds it into a Pandoc command to convert it to PDF. 

4. Set the hook as executable, from Git bash:

     ```
     $ chmod +x post-receive
     ```

### Testing

To test, make a readme.md file at the root of your repository, add it to your repository, then commit it. If all is properly set up, you will get your readme.pdf at the root of your repository, right next to readme.md. It's important to note here that once you make *any* change to *anything* under source control in your repository, committing those changes will fire off the post-commit hook. Our hook only cares about the event, not which file(s) triggered it. Here's a tip: If you are using SourceTree (or some other Git GUI), set it to "Always show full console output", that way if you hook fails, you will see the output which can help you debug.

With Pandoc and MiKTex installed, this post-commit hook translates our Markdown file into a beautiful PDF that we can proudly present to our client.

### Extra credit

For whatever reason, the left and right page margins on the MiKTex default template are wide, as in several inches each. To fix this, do the following:

1. Create a directory at C:/Users/<user>/AppData/Roaming/pandoc/templates to house your new default template. MiKTex looks for an override default template in this directory.
2. Get default.latex from https://github.com/jgm/pandoc-templates/blob/master/default.latex, put it in the templates directory you just created.
3. Modify default.latex by putting the following on line 2. This makes the page margins one inch:

     ```
     \usepackage[margin=1.0in]{geometry}
     ```

4. Edit your readme.md (or anything under source control in the repository) and commit the change. Check your PDF, your page margins should now be one inch.

## Using a post-receive hook to push a site to production

For another project, I was writing documentation in [Sphinx](http://sphinx-doc.org/index.html) and had a [Jenkins](https://jenkins-ci.org/) job polling my remote Git repository for changes every few hours. If Jenkins saw changes in the repository, it would copy the Sphinx HTML output over to an internal web server, effectively deploying the docs. What I *really* wanted to do was push the code manually at will, and I knew there had to be an easy way to do this, probably with Git.

This server-side hook isn't anything fancy, and plenty of variations of it can be found around the web. The following version is simple, pushing from a local development repository up to a bare remote repository on an internal server where a hook then deploys it.

### Requirements

[Git](http://git-scm.com/) installed locally and on your target server.

### Setup

The following instructions assume a Windows environment and some experience with Git bash.

1. Setup a bare repository on the target server. The sole purpose of this repository is to accept pushes and house our post-receive hook that will deploy the code.

     ```
     $ pwd 
     $ /e/test
     $ mkdir push-test
     $ cd push-test
     $ git init --bare
     ```

2. If you don't have a local development repository setup, then do so. Once you have one, register the remote. Note here that even though I am using a UNC path to my remote on the E drive of server1 (I found the use of forward slashes necessary), you could just as easily use a share path. I'm naming my remote "production", but you can name it whatever you want:

     ```
     $ git remote add production //server1/e$/test/push-test
     ```
     
3. Setup your post-receive hook in the bare remote repository so Git knows what to do when you push changes. Go into the .git/hooks directory of your remote repository and create a file named post-receive (no file extension). Open it up in your text editor of choice and add the following:

     ```
     #!/bin/sh
     GIT_WORK_TREE=//server1/c$/inetpub/wwwroot/git-push-test git checkout -f
     ```
     
    Note again that with UNC paths you have to use forward slashes. This simple bash script checks the code out to the directory specified by ``GIT_WORK_TREE``, effectively deploying our site. Also, even though the hook is on the server, you must use the UNC path and not the actual drive letter path.
    
4. Set the hook as executable:
 
     ```
     $ chmod +x post-receive
     ```
     
### Testing

Testing is simple, just push to your remote from development:

```
$ git push production master
Counting objects: 10, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 485 bytes | 0 bytes/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To //server1/e$/test/push-test
5e1b61d..1046196  master -> master
```
     
If all went as planned, once the remote received the push, the post-commit hook fired and your code was deployed to the directory you specified in ``GIT_WORK_TREE``.

## Conclusion

Git hooks are an easy way to enhance your productivity and help you complete tasks from the mundane to the extraordinary. A Git hook is merely an executable script that does work for you, whether that script is bash, Python, Ruby, or some other language is up to you. Although we barely skimmed the surface of what Git hooks are truly capable of, I hope these examples might help you think of ways to incorporate Git hooks into your workflows.