## Git helpers - custom commands
Git is really extensible. You can even create your own commands by simply making a file named `git-mycommand` in your `PATH`. Now you can reference the command as `git mycommand` from anywhere.

This repository contains a collection of my personal git helpers which I use in my workflow. The reason why I made those helpers as custom commands, and not as `git aliases`, is simply because they are a lot more customizable. I can pass parameters to the command, provide default values, custom messages, keyboard input, etc.

### Installation
Clone this repository and move the files in your `PATH`. You can move them in `~/.local/bin` for the current user, or `/usr/local/bin` for all users on the machine.

After that you have to mark them as executable. Something like `chmod +x git-*` should do the work.

### Clear merged branches
```
git cmb [brachname]
```
This command will remove all merged branches, compared to the currently active branch or the one provided as a first parameter.

**Example usage:**
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git cmb
  feature/my-awesome-feature
  fix/quick-patch
Those branches are merged into "master" and are no longer used.
Are you sure you want to delete them? [y]
y
Deleted branch feature/my-awesome-feature (was f5ecc7e).
Deleted branch fix/quick-patch (was 23f0ace).
```
This command is useful when you have incorporated a change in master and you no longer need the feature branch.

**Bonus:**
You can specify which branches should be skipped when running the command. You only need to modify the `BRANCHES_TO_SKIP` array in the script.

### Stash changes in the current branch
I make a lot of commits all the time. I use them for everything, even as a replacement for stashes.

When I jump between branches to work on different tasks, I often have to save my current changes somewhere. The first thing that comes to mind is `git stash`. But if you use it too much, it will eventually get bloated with all sorts of stashes.

That's where `git save` comes to the rescue!
You no longer have to remember where to apply a given stash, because `git save` is going to make a stash commit with datetime in your current branch.

**Example usage:**
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git status
 M README.md
 M adminer.css
 M src/adminer.scss
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git save
[feature/awesome 7e55134] Stash 2018-04-24 23:14:38
 3 files changed, 36 insertions(+), 17 deletions(-)
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git log
commit 7e5513481958cf2f1c6380ff1d9b240ee50a9df8
Date:   Tue Apr 24 23:14:38 2018 +0300

    Stash 2018-04-24 23:14:38
```

### Undo commit but keep the changes
So far so good. We've managed to stash our changes into our active branch, but how do we unstash them? It can get really tedious to type `git reset HEAD~1` all the time, just to unstash from `git save`.

That's how `git undo` saves the day!
You can easily undo commits, but keep the changes introduced into them.

**Example usage:**
Uncommit the last commit (press Enter once again for confirmation):
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git undo
Files that are about to be uncommitted:
aea42b1 removed footer border
M       adminer.css
M       src/adminer.scss
Are you sure you want to uncommit the changes back to "23f0ace54ed6fa577e2d51e9c3307fa008718f83"?
Press [q] to cancel.

Unstaged changes after reset:
M	adminer.css
M	src/adminer.scss
```
Uncommit several commits back, or provide commit hash (press Enter once again for confirmation):
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git undo HEAD~3
papernick@PaperNick-S1:~/Adminer-Material-Theme (feature/awesome)$ git undo 1a8fbe1
```

### Show changes in a commit with a GUI difftool
The default behavior of `git show` is to display changes introduced in a commit. The only problem is that it shows the diff in the console. If you're dealing with a larger commit, the chances are that you're going to get lost very quickly.

This where `git showtool` comes into play!
By default `git showtool` is going to show the changes from the latest commit in your branch. The first parameter gives you the opportunity to specify a commit hash.

**Example usage:**
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git showtool HEAD~2
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git showtool f5ecc7e
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git showtool
```

**Note:** You need to set a difftool first. I use `meld`:
```
git config --global diff.tool meld
```

### Reset changes safely
When I start working on a feature, I usually implement it in the most straightforward way, just to make it work.
After that, I iterate over the initial solution until it's polished and it looks clean.

This usually works fine, but there are times that things go completely wrong, and only a fresh start with `git reset --hard HEAD` can clear things up.
This command basically resets all your uncommitted changes, as if they never existed. But after you've executed this command, you remember that there was a hidden gem among the waste. Well, too bad, it's gone forever.

With `git wipe`, you can safely clear you current changes without having any second thoughts. Before deleting the changes, `git wipe` actually commits them, prints the commit hash into the console, and then removes them. The cool thing about this, is that if you want to restore them, you only need to revive the commit by its hash that was printed earlier. A simple `git cherry-pick hash-of-deleted-changes` should do the work.

**Example usage:**
```
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git status
 M Gruntfile.js
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git wipe
Wiped content commit hash was: 7f5aa79
HEAD is now at b15c27c Updated README.md by listing the differences from the original theme
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ echo 'Oh no! My precious grunt task!'
Oh no! My precious grunt task!
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git cherry-pick 7f5aa79
[master dec7ca2] Wipe checkpoint
 Date: Mon Oct 1 23:17:44 2018 +0300
 1 file changed, 1 insertion(+), 1 deletion(-)
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ git show HEAD~1..HEAD --stat
commit 7f5aa790e8c9894d23adc1fcc1e662d8c2308f1c
Date:   Mon Oct 1 23:17:44 2018 +0300

    Wipe checkpoint

 Gruntfile.js | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
papernick@PaperNick-S1:~/Adminer-Material-Theme (master)$ echo 'Whew, everything is here'
Whew, everything is here
```
