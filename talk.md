# Git feature branches

How to use Git to manage new features in your app – branching, merging, rebasing.

## Preliminary

At [Grrr](http://github.com/grrr-amsterdam) we use the Git Flow model as described in [A Successful Git Branching Model](http://nvie.com/posts/a-successful-git-branching-model/) by [Vincent Driessen](https://github.com/nvie).  
Read the article if you're not familiar with it. 

In summary:

- `master` is always pristine and should be ready for deployment at any given time. Nobody works directly in `master`.
- `develop` receives your updates through feature branches. 
- Feature branches merge into `develop`.
- Release branches package the work in `develop` for release into `master`. They merge into `master` and back into `develop`.
- Hotfix branches branch from `master` and also merge into `master` and into `develop`. Use these for bugfixes.

We recommend this fork of `git flow`: [petervanderdoes/gitflow-avh](https://github.com/petervanderdoes/gitflow-avh).

## Feature branches

But what is a good way to manage feature branches? And in a broader sense, what do you want your Git history to look like?  
Let's answer the second question first to get an idea for the first.

### Git commit messages

90% of the time, when I look at the Git history of any repository, I want to quickly see which features are enabled by the given (groups of) commits.  
I want to answer the question

> What was intended here?

And if the commit message answers that question I can checkout the commit hash and look at how the intented behavior was achieved.  

Ideally I would be able to switch features on and off by going back and forth in the commit history.  

A bad example would be this:

```
- start email feature
- WIP: added sending of notifications
- Fixed missing subject line
- Changed variable names to camelcase
- Fixed undefined var from previous commit
- Allow multiple recipients
```

Where a good example would just be:

```
- Send email notifications
```

If I ever want to go back to before the app sent email notifications, I could just go a step back into history and checkout the situation.  

But we're all humans. Sometimes we make mistakes. Sometimes we want to switch snake_case to camelCase. How do we prevent those commits from muddying the commit history?

### Lifetime of a feature branch

Ideally, when working in a team, your process looks like this:

- You branch off `develop` and create a new feature branch: `feature/email-notification`.
- You create commits that form your new feature.
- You rebase your commits into a package that adds value to your commit history. Honestly, most of the time this should probably be just a single commit describing the feature it adds.
- You publish your feature branch and receive a code review.
- You process the feedback given in the review, rebase the new commits to not muddy your neat package and push again.
- Your branch gets accepted and merged into `develop` (with or without fast-forward, whatever your preference)

This will result in a clean git history that describes only what features were added when and by whom and not so much _how_ you did it in ways that are inappropriate for a git commit message.

### Starting a branch

If you use the git flow model, it's as easy as 

```bash
git flow feature start email-notifications
```

At Grrr, we have added some shortcuts to [Garp](https://github.com/grrr-amsterdam/garp3) that prefix your branch with the name of the author, to quickly glance who's responsible for a given branch.

### Committing

You create commits, however they make sense to you.  
A feature branch is your personal workspace, it doesn't matter what you do as long as it's contained to your own branch.

If you wish to create a hundred small commits so you can skip back and forth in your process, do it. Whatever makes you productive.  
It doesn't matter at this point, because in the next step we're going to prepare our glorious mess into a pretty package.

Having said that, it's probably a good idea to get familiar with the `amend` flag:

```bash
git commit --amend
```

This allows you to merge your current and previous commits, optionally tweaking the commit message. It's the easiest way to merge two commits and super helpful whenever you forgot something in the previous commit, or made a typo or whatever.

### Rebasing

Your feature is done! Let's have a look at the commit history:

```bash
0f75d8c Added HTML email template
4b7a292 Fixed typo in docs
b61b840 Added documentation 
692e21e Allow admin to modify email subject
e006f04 Add email notifications 
```

Alright, that's five separate commits. We can do better and make this a bit more presentable for the reviewer.  
Let's use interactive rebase to do just that:

```bash
git rebase HEAD~5 -i
```
In this command, `-i` means interactive, `HEAD~5` means to rebase the last 5 commits.

Your `$EDITOR` will be opened and you're presented with the following document:

```bash
pick e006f04 Add email notifications
pick 692e21e Allow admin to modify email subject
pick b61b840 Added documentation
pick 4b7a292 Fixed typo in docs
pick 0f75d8c Added HTML email template 
```

In the commented section of the document is an explanation of all the rebase options:

- p, pick = use commit
- r, reword = use commit, but edit the commit message
- e, edit = use commit, but stop for amending
- s, squash = use commit, but meld into previous commit
- f, fixup = like "squash", but discard this commit's log message
- x, exec = run command (the rest of the line) using shell
- d, drop = remove commit

You can now change the list of commits using the various rebase options. For instance, if you would save and quit with the following list:

```bash
pick e006f04 Add email notifications
squash 692e21e Allow admin to modify email subject
squash b61b840 Added documentation
squash 4b7a292 Fixed typo in docs
pick 0f75d8c Added HTML email template
```

You will be left with a new history looking like this:

```bash
0f75d8c Added HTML email template
e006f04 Add email notifications 
```

The first three commits have become one, leaving only the first and last commits. Great! Much better for code review, and much easier to parse next year when you just want an overview of added features.

I don't like the way those verbs are used though. One commit says `Added`, another says `Add`. Sloppy! Let's correct that.

```bash
git rebase HEAD~2 -i
```

If we save the file like this:

```bash
pick e006f04 Add email notifications
reword 0f75d8c Added HTML email template 
```

We will be dropped into an editor to correct the `Added HTML email template`. If we save this as `Add HTML email template`, the history looks consistent again:

```bash
0f75d8c Add HTML email template
e006f04 Add email notifications
```

### Code review

Great! Push your work (`git flow feature publish` can do this for you) to the remote, log into Github and create a new Pull Request.  
The reviewer checks your code and alas, finds a typo. 

### Pushing 

Now there's a problem: your code is sent to the remote, meaning you can no longer safely amend or rebase. This would change the history of your repository and changing shared history is a cardinal sin.

So you're left with a history looking like this:

```bash
1fed98c Fix typos
0f75d8c Add HTML email template
e006f04 Add email notifications
```

Damn. You will notice when you rebase this, or add `--amend` to the commit, git tells you you have diverged and should both pull and push. It's annoying and can be quite confusing.

The answer is two-pronged: if you're the only developer working on the branch, force your push. Change history!

```
git push origin my-feature -f
```

But if you're collaborating with others on the same branch, refrain from amending or rebasing pushed commits. Wait until you're **merging**, and right before merging into the parent branch, do a quick rebase to tidy up.

### Merging

All done? Great. Either do a last rebase, as described in the previous session to tidy up, or go ahead and merge the commits as they are.

Note that you can (and maybe should) `git rebase master` to grab any commits that were added to the `master` branch since you branched off on your feature branch.


