[[_TOC_]]

---
# What this training is about

- An attempt to make you more comfortable with the `terminal` git commands instead of being reliant on GUIs
- An attempt to teach you some tips and tricks with git to make your day-to-day workflow more streamlined
- Get a little deeper into git to really understand what's going on under the hood with common  issues you might face 


# See what's happening with `git log`

The docs page for the `git log` is thousands of lines long. Here are some useful options:

The default `git log` output looks like this - it's fairly self-explanatory:

```
commit c79a8d5cbd209be43ce491f6d7457b9d2b9a9f51
Author: Jazz Singh <jazz.singh@skymill.io>
Date:   Thu Jan 12 23:08:00 2023 +0100

    Add environment variables for OKTA SSO

commit ccbb7678df9db2a833d4ad9c67f25f644aae933c
Author: Jazz Singh <jazz.singh@skymill.io>
Date:   Tue Jan 10 17:00:17 2023 +0100

    Add environment variables for AWS
```


For more info you can use the `--pretty=<format>` flag. There are various options here and the default is `medium` (shown above), so use whichever suits your needs at the time. For example, for even more info you can use `git log --pretty=fuller`, which splits the `Date` property from into `AuthorDate` and `CommitDate` [^1]


```
commit 0f38d2707f8a489bb0619b619364248be754f9c1
Author:     Jazz Singh <jazz.singh@skymill.io>
AuthorDate: Sun Jan 15 10:06:55 2023 +0100
Commit:     Jazz Singh <jazz.singh@skymill.io>
CommitDate: Sun Jan 15 10:06:55 2023 +0100

    Update README with release instructions

commit c79a8d5cbd209be43ce491f6d7457b9d2b9a9f51
Author:     Jazz Singh <jazz.singh@skymill.io>
AuthorDate: Thu Jan 12 23:08:00 2023 +0100
Commit:     Jazz Singh <jazz.singh@skymill.io>
CommitDate: Thu Jan 12 23:08:00 2023 +0100

    Add environment variables for OKTA SSO
```


Perhaps a more commonly-used command for `git log` is `git log --pretty=oneline --abbrev-commit`, or its alias `git log --oneline` which produces output similar to the below [^2]

```
af31224 Remove redundant env var
621a546 Add environment variables for Google SSO Authentication
20d4b3e Update README.md
ef26ee4 Fix ordering of .env variables
1792784 Update README, and add env vars for user-service
4959618 Update README
2ab4852 Update env.example
494a482 Add JWT_KEY environment variable for backend service
0d280a1 Merge branch 'finalise-user-service-local-env' into 'master'
94f8d5e Fix environment variables for user-service .tar image
2175fd2 Merge branch 'add-user-service-image-tar' into 'master'
915fb26 Add user-service.tar for local dev
f883eef Merge branch 'add-user-service-local-infra' into 'master'
e2ec874 Add SONY_FORUM_USER_SERVICE__URL env var
```


**Protip**: you can add `-<number-of-rows>` to only see `n` rows, for example: `git log --oneline -4` would produce the following output:

```
0f38d27 Update README with release instructions
c79a8d5 Add environment variables for OKTA SSO
ccbb767 Add environment variables for AWS
e02e95e Ad notifications db to compose file, update env.example
```

## See the contents of a single commit
It's very likely that you may one day need to see what actually went into a commit. You can use the simple command `git show <commit>` for this to see a *diff* of only what changed in that commit, compared to the commit before. Info on reading *diffs* is below.

# Fix mistakes with `git commit --amend`

This is a very useful command to use with `git commit` in (amongst others) these common situations:
1. You make a commit and forgot to include a file
2. You had a typ0 in the commit message

In a nutshell, this could be a simple scenario:

```
git add file1.txt
git commit -m "Add new files"
```

Oops! I forgot to add `file2.txt`! I *could* do this:

```
git add file2.txt
git commit -m "Forgot to add file2.txt"
```

*Or*, a cleaner way to do it would be using `git commit --amend` like so:

```
git add file1.txt
git commit -m "Add new files"
git add file2.txt
git commit --amend
```

You'll be presented with your configured text editor (default: vim) - something like this:

```
Add new files

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sun Feb 26 15:33:50 2023 +0100
#
# On branch master
# Changes to be committed:
#       new file:   file2
```

At this point you can just code the editor if all you wanted to do was add changes to the last commit, or you could also fix a typ0 in the commit by changing the commit message, saving, and then closing the editor.

**Note**: The same applies for ANY change, not just adding new files.

# See what's changed with `git diff`

## What does a "diff" look like?

Here's an example of the output of `git diff`, with some labels to explain the different parts:

![[Pasted image 20230226180847.png]]

The first line simply shows us an expanded version of the command.

- Blue: Metadata about the files being compared - usually not too useful, basically a before and after hash of the file
- Pink: This shows that the latest commit version of the `index.html` file is given the token `-` and the current, modified version is given the token `+`
- Yellow: Everything from the `@@` to the line before the next `@@` (or the end of the output) is called a **diff chunk**, and the first line is always the **chunk header**

A chunk header can look a little cryptic but is just two sets of numbers, one from the first file (`-`) and one from the current change (`+`). These numbers simply show:
1. Which line the chunk starts on
2. How many lines of the file in the chunk showing

So for example, this chunk header:
```
@@ -16,10 +16,11 @@
```

Is telling us that for file `-` this chunk starts at lines 16 and is 10 lines long, and for file `+` the chunk starts on the same line, but is 11 lines long, (in this case because we added a line).

**Note**: Although git *usually* picks file `a` as the current commit and gives it the token `-` and picks file `b` as your new, working changes file and gives it the token `+`, this is **not** always the case. Most of the time, though, we  get away with thinking of the `-` file as the "old" version 
and the `+` file as the "new" version

## See unstaged changes

Without additional options, `git diff` compares all the changes in the working directory that are **not** staged for the next commit. In other words, running `git diff` on its own will only show the difference between what git "knows about", and what git doesn't "know about".

```
Demo:

mkdir ex1
cd ex1
git init
touch employees.txt

[add a name to the file]

git add .
git commit -m "add first name"
git diff
```


You should see... nothing! There is nothing git doesn't know about. You added a file, modified it, but then you committed it!

6. Now add another name to the file, but don't add or commit it, and run `git diff`

This time you'll see some output, because git doesn't know about the change yet!

**Caveat**: This doesn't apply to new files. New files are "untracked" and therefor you might say "git doesn't know about it", but this is an annoying exception to the rule

## See staged changes

Similarly to above, you might want to see the changes that are going to make up your next commit. If you followed the above example and added another name to the file, if you now run `git add .` you'll see that again, even though it's **not committed** you still can't see the diff by running `git diff`.

To see a diff of what's in the staging area, run `git diff --staged` [^4]

## See all changes since the last commit

To see everything that's changed since your last commit, r**regardless of whether it's staged or unstaged**, run `git diff HEAD`. 

Note that `HEAD` must be capitalised.

## Comparing files

With the above two commands, you have everything you need to see the changes to specific files or directories since the last commit:

`git diff HEAD file1.txt`
`git diff HEAD src/components`

As well as changes that are staged for commit to a specific file/directory

`git diff --staged file.txt`

## Comparing branches

In order to compare two branch, you can use the below command:

`git diff branch1..branch2` or `git diff branch1 branch2`

At this point the diff can get more confusing, since you can change the order of the branches (`git diff branch2..branch1`), so it's even more important to remember that "-" doesn't necessarily mean "removed" and "plus" doesn't necessarily mean "added". They are simple symbols to differentiate one context from another. A "context" in this a branch, but could be a file or a commit etc.

So, in the above example "a" in the diff will be `branch1` and "b" will be `branch2`

```
diff --git a/employees.txt b/employees.txt
index bf3dd7e..e427a65 100644
--- a/emp.txt
+++ b/emp.txt
@@ -5,3 +5,5 @@ staffan
 jazz
 james
 jean
+maria
+bob
```

In the above diff we ran `git diff branch1..branch2` so we are effectively seeing that `branch2` has "maria" and "bob" , and branch 1 does not.

If switched the order of the `diff` command though and ran `git diff branch2..branch1`, we would get this:

```
diff --git a/employees.txt b/employees.txt
index e427a65..bf3dd7e 100644
--- a/emp.txt
+++ b/emp.txt
@@ -5,5 +5,3 @@ staffan
 jazz
 james
 jean
-maria
-bob
```


Now `branch2` is given the "-" symbol and is context "a", but we are saying the **exact same thing**, "maria" and "bob" are coming from `branch2`

## Comparing commits

Comparing commits works exactly the same way as comparing branch, but you simple use commit hashes instead of branch names:

`git diff commit1..commit2`

e.g `git diff c2c1c13..c8ee40f` - note, you can use the abbreviated commit hash


# Undoing Changes and Time Travelling

## Checking out old commits
`git checkout` does a lot of things, which is why new commands like `git switch` and `git restore` were introduced with newer versions of git.
We can use `git checkout` to not only checkout a branch, but also an individual commit.

e.g `git checkout d2349e3` [^5] 

If you do the above you'll check a "scary" message about your HEAD being detached!

```
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

### Detached HEAD state
This is a temporary state, you can take a look around at your files to see what the state of the project was at the given commit. You can easily exit the detached HEAD state by running `git switch -` (see tips & tricks for more detail on this syntax).

HEAD *usually* refers to a branch, but not necessarily, as you can see in this case. In this case, HEAD points to a commit of your choosing.

### Referencing commits relative to HEAD
You can also refer to the commits relative to HEAD, e.g. `git checkout HEAD~1`. In this example we are checking out *the commit before* HEAD. You can think of tilde (`~`) as "minus". An alternative for referencing one commit prior is `git checkout HEAD^`.

Of course you can go any number of commits back: `git checkout HEAD~5`.

## Discarding changes
**Important note**: Most of the following commands result in a permanent change!

Sometimes we may want to undo the changes we have made to one or more files since our last commit, and "reset" a file to the way it was. You can, of course, simply undo the changes manually, though it is often quicker and less error prone to use a git command. 

As always with git, there's more than one way to accomplish the same thing: 

1. Using `git checkout`:
	+ `git checkout HEAD <file>`
	+ `git checkout -- <file>`
2. Using `git restore`:
	+ `git restore <file>`

## Reverting a file back to the way it was at commit `{x}`
**Important note**: Most of the following commands result in a permanent change!

Sometimes you might want to revert a file back to the way it was at a particular commit (or n commits ago). The syntax is the following, note the `--source` flag - just be careful since it's a permanent change!

`git restore --source <commit>`

Or of course the standard HEAD syntax also applies:

`git restore --source HEAD~1`

## Un-staging changes
If you even accidentally stage a file that shouldn't be staged, you can easily unstage it:
`git restore --staged <file>`

`git status` gives you a pretty big hint!
```
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
```

## Undoing commits
There are at least two way of "undoing" commits with git, which both accomplish the same thing in different ways

### Resetting with  `git reset`
You might end up in a scenario where you made some commits on a branch you didn't mean to, for example directly on "master" instead of a feature branch. You can use `git reset` to fix that - `git reset` moves the HEAD pointer backwards, to the commit you choose, **eliminating commits**

`git reset <commit>` will reset the repository back to that commit, anything before that commit is **gone** - but **not the actual changes**. The changes are still in your working directory.

```
Demo:

[make a change]

git add .
git commit -m "mistake commit"
git log --oneline

[note mistake commit is present, and get the commit hash]

git reset <commit> or git reset HEAD
git status

[note that the commit is gone, and the changes remain in the working directory as unstaged]
```

You can then "take these changes with you" to another branch and make a new commit.

#### Hard resetting
If you **do** want to remove the commit **and** permanently remove the changes in the files, you can add the `--hard` flag, however read about `git revert` below before you go this route to ensure it's the best option for you and your team.

### Reverting with `git revert`
`git revert` is similar to `git reset` in that it will undo a commit, however the difference is that `git revert` will create a new commit that will contain the "undoing" of the commit you chose.

The main reason you would choose `revert` over `reset` is if **other people already have commit you are reverting on their machines**. Resetting can cause issues with merge conflicts that reverting doesn't, since reverting is not changing the git timeline, simply appending to it.

```
Demo:

[make a change]

git add .
git commit -m "mistake commit"
git log --oneline

[note mistake commit is present, and get the commit hash]

git reset <commit> or git reset HEAD <--- note, not HEAD~1!

[editor will open, accept the default message and close it]

git log --oneline

[note that the commit is still there, but now there is a new commit which reverses the changes]
```

# Fixing merge conflicts
You're probably used to fixing merge conflicts in files with the help of a conflict GUI, usually one that's built directly into your IDE. But without all the fancy colours and highlighting, what do they actually look like, and how can we fix them manually?

Creating a conflict

```
Demo:

git switch master
[make a change]

git add .
git commit -m "some change"
git switch -c new-branch

[make a change on the same line]
git add .
git commit -m "same line change"

git switch master
git merge new-branch
```

You'll see some **conflict markers** that look similar to this:

```
<<<<<<< HEAD
james
=======
jane
>>>>>>> new
```

All we need to do to fix this is to delete the conflict markers and "make the file the way we want it to look". **You cannot perform any git actions if there are conflict markers present in any file**. The conflict markers in the above example are telling us that our current branch and HEAD position has the change "james", and the branch `new-branch` has the change "jane".

In this case we can simply remove these three lines:

```
<<<<<<< HEAD
=======
>>>>>>> new
```

and then choose if we want to keep "james", "jane", or both. After that we just need to run `git add .` to stage the changes, and then commit the new changes. 

# Rebasing
The scariest/most confusing git command? Not really. There are just a few things to know about that can clear things up.

## Rebasing vs Merging

We won't go into detail with "normal" rebasing right now, but we'll mention the difference between merging and rebasing. 

`git merge` combines different branches' changes and creates a new commit that merges them together. The result is a history that shows multiple parallel lines of development.

`git rebase` moves changes from one branch to another by reapplying each of the changes onto the destination branch. This creates a linear history that may be easier to understand but can result in conflicts if multiple developers are working on the same code.

The part that most people are scared of (even though they might not be able to describe it) is that rebasing can become hell if team member “a” rebases commits that team-member “b” does not have.

# Interactive Rebasing - cleaning up history
Interactive rebasing in Git allows you to modify, reorder, or split commits in a branch before incorporating them into another branch, by using the command `git rebase -i`. This can help keep a clean commit history, but should be used with caution to avoid potential issues for other contributors, as discussed above.

```
Demo:

Clone this repo to get a demo repo with a bunch of silly commits:

git clone https://gitlab.com/microtraining-git/interactive-rebase-demo

```

When you run `git rebase -i` you need to provide a commit to go back to. You can do this by  either giving an explicit commit hash, or using the HEAD syntax discussed previously. For example to include the last 4 commits in your rebase you could use:

`git rebase -i HEAD~4`

If you only have a few commits and what to go back to the very first commit the HEAD syntax won't work if you also want to include the "initial commit". For this situation you need to use the `--root` flag instead of a commit reference. Usually you will likely have hundreds if not thousands of commits, so it wouldn't be practical to go all the way back to initial project creation, but in this case we only have about 11 commits in total so we'll keep it simple:

`git rebase -i --root`

When you start a rebase there are a few options available to you - let's talk about just a few of them right now, by looking at the self-explanatory messages for each - notice that one can use just the first letter of the command:

```
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
```

When you run the command, your chosen editor will open and you'll see all the commits from the commit you chose all the way up to HEAD. You'll notice that the order of the commits has changed. Normally the order goes from (top to bottom) newest commit to oldest. In the interactive rebase it's reversed (example shows the result of `git rebase -i --root):

```
pick 0e19c7a initial commit
pick cbee26b I added project files
pick 519aab6 add basic HTML boilerplate
pick 240827f add bootstrap
pick 6e39a76 whoops forgot to add bootstrap js script
pick 4ff2290 add top navbar
pick 2a45e71 fix navbar typos
pick 655204d fix another navbar typo
pick 2029e20 my cat made this commit
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

`pick` just means "keep this commit" and is the default command for each commit

## Rewording
If you want to reword a commit, you can simply change "pick" to "reword" (or "r"). In this example I want to reword commit `cbee26b I added project files` to read "Added project files" instead. So let's change `pick` to `reword`:

```
pick 0e19c7a initial commit
reword cbee26b I added project files        <---
pick 519aab6 add basic HTML boilerplate
pick 240827f add bootstrap
pick 6e39a76 whoops forgot to add bootstrap js script
pick 4ff2290 add top navbar
pick 2a45e71 fix navbar typos
pick 655204d fix another navbar typo
pick 2029e20 my cat made this commit
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

Note that **I don't actually change the commit message here**. I'm just telling git what I want to do, to each commit, and I can queue up several commands. In this case when I save and close my editor, another instance of my editor will immediately open:

```
I added project files
  

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
// ...
````

Here is where you can actually change the commit message to whatever you want. Running `git log --oneline` now shows the newly edited commit message:

```
4d03f73 (HEAD -> main) Add credit to Colt Steele in README
d954e32 Create README.md
fb3f9b8 my cat made this commit
2289422 fix another navbar typo
60da46c fix navbar typos
99fd100 add top navbar
fccbae1 whoops forgot to add bootstrap js script
d81b7af add bootstrap
8eaf278 add basic HTML boilerplate
804b7a1 Added project files                   <---- 
0e19c7a initial commit
```

You'll notice now that every commit starting with the commit whose message we changed now has a new hash (`0e19c7a initial commit` is unchanged).

## Fixing up / squashing

`fixup` and `squash` accomplish the same thing, but the difference is just which log messages to keep. The idea behind these commands is to merge together two or more commits into one. Let's reset back to the remote and get our repo back to how it was without any rebasing:

`git reset --hard origin/main`  <-- *(this command will be explained later)*

`git rebase -i --root` should now show us this output again:

```
pick 0e19c7a initial commit
pick cbee26b I added project files
pick 519aab6 add basic HTML boilerplate
pick 240827f add bootstrap
pick 6e39a76 whoops forgot to add bootstrap js script
pick 4ff2290 add top navbar
pick 2a45e71 fix navbar typos
pick 655204d fix another navbar typo
pick 2029e20 my cat made this commit
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

If I want to merge together two commits, for example:
+ `6e39a76 whoops forgot to add bootstrap js script` 
	and
+ `240827f add bootstrap`

It makes sense to just have one commit that adds Bootstrap to our project, `6e39a76` is really just a mistake since the JS script is also necessary for Bootstrap - so it would be tidier to just have a single commit with the message "add bootstrap" that included all the changes from these two commits. Here's how we would do that:

```
pick 0e19c7a initial commit
pick cbee26b I added project files
pick 519aab6 add basic HTML boilerplate
pick 240827f add bootstrap
fixup 6e39a76 whoops forgot to add bootstrap js script <--- [added 'fixup']
pick 4ff2290 add top navbar
pick 2a45e71 fix navbar typos
pick 655204d fix another navbar typo
pick 2029e20 my cat made this commit
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

On closing the editor we just get a `Successfully rebased and updated refs/heads/main.` message in the terminal - and here's the output of `git log`:

```
98aeb81 (HEAD -> main) Add credit to Colt Steele in README
efb4bba Create README.md
0a29a57 my cat made this commit
1de4c87 fix another navbar typo
71e2ecc fix navbar typos
5328d1a add top navbar
4a53935 add bootstrap
519aab6 add basic HTML boilerplate
cbee26b I added project files
0e19c7a initial commit
```

Notice we no longer have the `whoops...` commit - it has become part of the `add bootstrap` commit and the hashes have changed from then onwards. 

For reference, here's what the output would be if we used `squash` instead:

```
33c3c32 (HEAD -> main) Add credit to Colt Steele in README
2cb213a Create README.md
3b426af my cat made this commit
27ae71e fix another navbar typo
52c3fd7 fix navbar typos
293c41f add top navbar
cdae1a2 add bootstrap
519aab6 add basic HTML boilerplate
cbee26b I added project files
0e19c7a initial commit
```

Looks the same right? But let's try `git log` instead of `git log --oneline`:

```

commit 293c41f10baeb82781f9bcca5f87bd1222f3210a
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 14:07:11 2021 -0800

    add top navbar

commit cdae1a2b54b1457497caccc24f09e68ae73b850b
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 14:04:05 2021 -0800

    add bootstrap                                         <--- 

    whoops forgot to add bootstrap js script              <---

commit 519aab65cbe432f3c982e0a11f7cb102e96ec402
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 14:02:45 2021 -0800

    add basic HTML boilerplate

commit cbee26b0cfb3a3fe518a69aaec3996dfbef213c2
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 13:58:21 2021 -0800

    I added project files

commit 0e19c7ad4130d9edb45bf5e4d9f4cbdd233dcffc
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 13:57:10 2021 -0800

    initial commit
```

We can see that the first line is the commit we want, but we have also kept the commit message from the squashed commit. Most of the time you probably want to just use `fixup` but there may be times for documentation purposes that you will want to retain information of the squashed commit. 

Note that when using `squash` after closing the editor, another instance of your editor will open (unlike with `fixup`) where you can further edit the commit message of the squash. The default will be as you can see above - where each commit message is separated with a new line.

## Dropping commits
You may well have a commit that you want to completely get rid of, perhaps it's no longer relevant, or just a mistake. In our example `2029e20 my cat made this commit` probably fits the bill. Let's inspect it with `git show 2029e20`:

```
commit 2029e202554152bc11dc82803c2980d2f260cbe5
Author: Colt Steele <5498438+Colt@users.noreply.github.com>
Date:   Tue Feb 9 14:16:17 2021 -0800

    my cat made this commit

diff --git a/app.js b/app.js
index e69de29..4da4f9b 100644
--- a/app.js
+++ b/app.js
@@ -0,0 +1,4 @@
+alskj
+askdj;) (; a
+sdkljasdkljas
+aslkdja)
\ No newline at end of file
```

Yes - it doesn't seem to be very helpful and obviously a mistake! Let's get rid of it. This time we only need to go back 3 commits so let's use `git rebase -i HEAD~3`:

```
pick 2029e20 my cat made this commit
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

Let's use `drop` this time:

```
drop 2029e20 my cat made this commit          <---
pick 3b23c17 Create README.md
pick 5817334 Add credit to Colt Steele in README
```

And just like that, the commit is erased from the history:

```
e1d9a57 (HEAD -> main) Add credit to Colt Steele in README
3b104a3 Create README.md
655204d fix another navbar typo
2a45e71 fix navbar typos
4ff2290 add top navbar
6e39a76 whoops forgot to add bootstrap js script
240827f add bootstrap
519aab6 add basic HTML boilerplate
cbee26b I added project files
0e19c7a initial commit
```

## Pushing your rebased changes to the remote
This is where things can get a little messy. Ideally you would be 100% sure that nobody else has worked on your branch. if it's a feature branch this is likely the case. Your local git commits are now completely different to what's in the remote, even if you only made one change, everything after that commit will effectively be a new commit! `git status` will confirm this:

```
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 3 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

nothing to commit, working tree clean
```

Contrary to the message above, you don't want to pull the remote change into your local changes - you want to **forcibly update** the remote branch with **your** changes. You normally have to real options at this point:

`git push --force`
	or
`git push --force-with-lease`


`git push --force` simply forces the push, overwriting the remote branch without checking for changes. This can be dangerous if other people are also working on the same branch, as it can overwrite their changes and cause conflicts.

`git push --force-with-lease`, on the other hand, performs an additional check before overwriting the remote branch. It checks whether the remote branch has changed since the last time you pulled or pushed to it. If the remote branch has changed, **the push is aborted to prevent conflicts**. If the remote branch has not changed, the push proceeds as normal.

Basically, if you're 100% certain that you are the only person who has committed to your branch, use `--force` , otherwise use `force-with-lease` for safety.

***
---
***
---
***
---


# Tips & Tricks
This section will grow, but here are just a few quick tips that you might find useful!


### Switching back and forth between branches quickly
`git switch -` will switch you back to the last branch you were at. Running it again will revert you to the branch you were on before you originally ran the command. 

### Changing the default editor for git (if you hate vim)
Most of the time we don't need to use a text editor with git, but for some commands using an editor is unavoidable and, depending on your setup, the default editor is probably `vim`. Vim is a great tool, though has somewhat of a learning curve and probably requires its own microtraining!

Let's assume we want to use `vscode` instead - here's how we can make it so `vscode` opens instead of `vim` any time we need to use the editor with git [^3]

```
git config --global core.editor "code --wait"
```

If you want to use an editor other case `vscode` the official docs have great page that shows the command for [various other editors](https://git-scm.com/book/en/v2/Appendix-C%3A-Git-Commands-Setup-and-Config)

## Dealing with branches
### Creating branches the easy way

- Create a branch:
>`git branch <branch-name>`

- Checkout a branch
>`git checkout <branch-name>`

- Create a new branch and switch to it:
>`git checkout -b <branch-name>`
>
  **or**
>
>`git switch -c <branch-name>`

**Recommendation**: use `git switch` for simple tasks around switching and creating branches. `git checkout` has many more functions.


###  Deleting branches

- Switch to any other branch
> `git switch <other-branch>`

- Now you can delete the branch
> `git branch -d <branch-to-delete>`


#### Force delete
If the branch you want to delete has uncommitted changes, you might get this message:
```
The branch <branch-to-delete> is not fully merged
```

To force the delete, simply use a capital "D" for the delete flag in the previous command:
>`git branch -D <branch-to-delete`


### Renaming branches

- Switch to the branch you want to rename - **note**: unlike the delete command, with which you need to be on **any other branch**, you must be on the branch you want to rename
>`git branch -m <old-name> <new-name>` <-- "-m" stands for "move"


# Coming in future parts(?):
+ Stashing
+ Cherry-picking
+ Git Tags
+ Administering Remotes (dealing with force pushing etc)
+ Merge strategies
+ Git submodules
+ Git aliases
+ Inside the Git folder
+ Reflogs
+ More tips & tricks

Footnotes

---

[^1]:  you might be wondering about the difference between the two types of date. The **author date** notes when this commit was originally made (i.e. when you finished the `git commit`). According to the docs of [`git commit`](http://www.kernel.org/pub/software/scm/git/docs/git-commit.html), the author date could be overridden using the `--date` switch. The **commit date** gets changed every time the commit is being modified, for example when rebasing the branch where the commit is in on another branch ([more](https://gist.github.com/x-yuri/ec13ea740f766e72430ed8b9a9a72884)).

[^2]: Note that the `--oneline` only shows the first line of the commit message.

[^3]: If you used a graphical installer when you first installed git, you could also have chosen the default editor at that point. If so, you can skip this part :)

[^4]:  You can also use`git diff --cached` is an alternative alias for `git diff --staged`
[^5]: You can also use `git switch {hash} --detach` to accomplish the same thing


