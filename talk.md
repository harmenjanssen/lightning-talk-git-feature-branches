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

### Rebasing

### Code review

### Pushing 

Force

### Merging
