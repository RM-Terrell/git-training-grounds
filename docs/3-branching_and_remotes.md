# Branching

## What is a branch?

In the last section we ended with making a branch and a commit to the branch. This will be a common workflow for anyone using `git` as one of `git`s main advantages over other VCS is how fast and efficient making and merging branches is. Older VCS could actually take significant periods of time to handle branch changes and with `git` it is near instant. [The "under the hood" reasons](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) for this performance has to do with how simple a branch really is. In short, _a branch is simply a pointer to a previous commit, continuing on in a chain as you add commits, with each commit storing its parent._ If that concept is a little confusing, it will begin to get clearer as we investigate branches and how to merge. Keep the above linked bookmark and refer back to it as we go.

In general when working with `git`, treat branches as ephemeral. Branch often, merge often even for very small changes, and delete your old branch once your changes are a part of the master copy. Avoid long lived branches that diverge significantly from the master copy, because as you will see later handling merge conflicts is tricky business and the less of it you need to do the happier you will be.

## main / master branches

There is nothing inherently special about the `master` or `main` branch. It is simply convention that the "final" copy of your code goes there, but that is by no means a hard rule or enforced by `git` itself. Some companies have a `production` branch that could be considered the true final copy of the code as it's the one sent to customers. The main difference is that the `master`/ `main`, and `production` branches will have special protections in GitHub or Bitbucket (or an equivalent tool) to prevent users from "pushing straight to master", meaning making commits on the branch and then pushing them up to the remote repo thus bypassing the pull request process and any of its verification steps.

## Common Branch Commands

### Switching branches

To switch branches via the console you use the `checkout` command. Changing branches will cause `git` to update all files in your repo to the state set by the newest commit on the branch you are switching to. To see this in action make sure you have the `/src/street_art.md` open, then to switch to `master` (which will not have your changes) run the command:

```bash
git checkout main
```

And then run another `git status` and you will see your branch has changed to `master`. Also, looking at `street_art.md` you should see your changes gone as they were part of your previous custom branch we made and the commit has not be made a part of `master`. If you want to skip ahead conceptually a bit and see this in action run the command `git log` on each branch and take a close look at the whats in the output. Do you see the difference? Hit `q` to exit when you're done and we'll come to `log` in the section "Exploring the Past".

Alternately, `git` now has a `switch` command that is a more user friendly way to switch branches, as `checkout` can be a bit of a confusing "swiss army knife" of a command, as discussed [here](https://stackoverflow.com/questions/57265785/whats-the-difference-between-git-switch-and-git-checkout-branch) on Stack Overflow. Docs on `switch` can be found [here](https://git-scm.com/docs/git-switch). Once again, `git` has a lot of ways to solve a particular problem as "software development" is a broad industry, so its worth exploring alternate workflows to see what fits your style and project best.

### When checkout fails

If you try to switch branches via `checkout` sometimes you will see an error like this:

```bash
error: Your local changes to the following files would be overwritten by checkout:
    src/street_art.md
Please commit your changes or stash them before you switch branches.
```

This happens when two conditions are met:

1. You have uncommitted changes in your working directory to a file.
2. The branch you are trying to switch to has changes in the same file you made changes to in condition 1.

This is `git` protecting you from losing your work in a file that has been changed in both branches and reflects the imperative nature of `git` as a tool. It won't automatically solve this situation for you as there is no objectively "correct" way to solve it. Thus it asks you to do it. You have a few options here:

1. **Commit your changes**: If you are happy with your changes, you can simply commit them to your current branch and then switch branches.
2. **Stash your changes**: Per the "Making Changes" doc, you can make use of `stash` to temporarily store your changes (thus cleaning the Working Tree and Staging Area) while changing branches, and then run a `git stash apply` to bring them back after switching and apply them on top of the incoming changes.
3. **Discard your changes**: If you don't care about the changes you made, you can run `git checkout -- file/path/here` to discard changes in a specific file, or `git reset --hard` to discard all changes in your working directory, as mentioned in the "Making Changes" doc. This WILL DELETE ALL YOUR WORK so make sure you want that before running it.

### Making new branches

To make a new branch, you use the `branch` command followed by a name for it. By default it will use whatever branch you are sitting on currently, with whatever the most recent commit was _in that copy you are on_ as the base for your new branch. It will NOT automatically get the newest code from Bitbucket, as per `git`s design choice of being a imperative tool, you must invoke every action yourself. Lets make a new branch that we are also going to delete in the next section.

Run the following command

```bash
git branch new-temp-branch
```

Or name it whatever you like. Run a `git status` and what do you see? Are you on your new branch? No. Again `git` is imperative and won't do things with you telling it to, things like switching branches. However, to verify your new branch was created successfully you can use the following command

```bash
git branch
```

With no other args passed in. This should generate an output like this, though likely with less branches

```bash
user@f15hf98499c5:/workspaces/project$ git branch
  ticket-48367_server-tests
  dev
  dev-abandoned-for-a-year-on-auto-delete-but-infra-critical
* cherry-pick-gitignore-updates
  kelsier_was_here
  dev-merge-me-on-a-friday
  main
  merge-conflict-of-damocles
```

The `*` is special here. It indicates what branch you are currently on. In your output you should see the name of your new branch. Now look back at the last section and figure out how to switch to it. Run another `git branch` to see the `*` move and then switch back to `master`.

As mentioned previously, by default the `branch` command will create a new branch based on the one you're currently on. This isn't always desirable. You can pass a different base branch name in as your second optional argument to the `branch` command like this

```bash
git branch new-branch alternate-feature-branch
```

To use `alternate-feature-branch` as your base. This will be especially helpful for when you're sitting on a branch you have active work being done on and you want to make a new feature branch off `master` without needing to switch to it first.

### Create a branch and switch automatically

It is a common enough workflow however to want to both make a new branch _and switch to it_ right after that there is a short cut to do just that. As an alternative to the previous example using `branch` followed by `checkout`, you can just run

`git checkout -b yet-another-new-branch`

Do that now, and then run a `git status` or a `git branch` to see that you are on the new branch.

### Deleting branches

Since branches are meant to be ephemeral though, we often delete them. We're going to do that now but purposefully fail at it in a few ways to learn to get used to `git`s error messages. First off, switch onto one of those new branches you made, then run the following command (sub in the name of your branch for `the-branch-youre-on`)

```bash
git branch -d the-branch-youre-on
```

In your console you'll see something like this:

```bash
➜  git-training-grounds git:(temp-test) git branch -d temp-test
error: Cannot delete branch 'temp-test' checked out at '/Users/rm_terrell/Documents/repos/git-training-grounds'
```

That makes sense given you have that branch currently "checked out", `git` won't let you delete a branch out from under yourself as what would it do after that? Auto switching to another branch is not the sort of thing `git` does. Switch to another branch and then run the same delete command again and you'll see a confirmation that you deleted the branch.

### Deleting branches with changes in them

One very helpful, if sneaky feature of `git` is that it can detect when branch has commits that hasn't been merged into its base branch. When you try to delete such a branch with a `-d` you will get an error like this:

```bash
➜  git-training-grounds git:(main) git branch -d temp-test     
error: The branch 'temp-test' is not fully merged.
If you are sure you want to delete it, run 'git branch -D temp-test'.
```

The error message even tells you how to delete it if you're really really sure using `-D`. When working with a remote repo on GitHub or BitBucket it is rather common to encounter this error and be confused as to why. Next time you merge a PR in a real `git` project you can recreate this error for yourself by locally trying to delete your feature branch with `-d`. But why? Didn't you merge your branch into `master` via a PR? Yes, but into _the master branch on the remote GitHub repo_.

This is jumping ahead a bit, but the code stored in tools like GitHub and Bitbucket is just another copy of your own code on a different server, and you'll need to pull down the fact that that branch was merged into `master` into your own local copy, and then the error will go away because the changes are now present locally. More on that to come shortly. In the meantime, know that this error is common and can be overridden with a capital D delete command or by synching down the changes to `master` to your local copy. One solution assumes you know what you're doing, the other follows the safe path. Your choice.

## Yet another brief historical tangent

Why does `git` use the word "checkout" for the command for switching to a different branch? The term actually doesn't originate with `git`, but from back in the early days of VCS. Early VCS often had locking mechanism where if you wanted to work on a file you "checked it out" like how you would checkout a book from a library, meaning that once you had the checkout on the file _only you could work on it_.

As you can imagine this doesn't scale. Imagine trying to work in a modern software company where only one person could work on one file at a time, and if they forgot to release the checkout lock when they went home for the day, the team on the other side of the planet couldn't edit it because that engineer was busy being asleep and not answering Slack pings. Although `git` doesn't have such limitations the term "checkout" was already so engrained in VCS lingo that it stuck around.

## Local code

This entire time you have been working with your own local copy of the code for this repository so this concept is well familiar to you. The files live on your system and can be explored with any file exploration tool, be it VS Code, VIM, IntelliJ, etc and they can be modified at will without automatically effecting other copies of the code on other systems. No surprises. You will need to invoke a `push` command to do that, which we will get to shortly. You do not have to have just one copy either on your system, clone it as many times as you want, but know that keeping them all in synch may prove tricky as you'll see shortly. There is however one workflow where having two copies of a repo on a single system makes sense...

### Multiple copies for multiple branches

The "right" way to view differences between two branches of code according to `git` is to use the `diff` command as it can take branches as arguments and will produce diff files for you to traverse or feed into AI tools for extra analysis. However, it is sometimes nice to have one copy of the code open on one branch in your editor, and one open on another branch side by side, possibly even running under debuggers so you can watch each version execute side by side. This is a common request for people who are new to `git` as it maps to how we're used to viewing versions of a file. Sadly though you can only have one branch of a repo checked out at a time for that given copy of the repo. If you open the same file twice in your editor, then switch branches, _both_ copies will update as they originate at the same file location controlled by `git`.

This can be worked around however by simply cloning the repo twice in two different spots on your machine, then opening each in a different editor window and running the needed `checkout` commands in each. It is generally advised not to do this for normal workflows as it becomes easy to make edits or run `git` commands in the "wrong" repo and lose track of your work, or hit confusions on which repo is the source of truth. For a quick branch comparison reference though feel free to do this if it helps you visualize changes or if, as mentioned above, you need to actually run two versions at once. Do try to get comfortable though with the `diff` tool between branches as that output is how you'll do code reviews anyway in Bitbucket / Github.

### Merging locally

It is unlikely you will do much merging of two branches on a local system (because remote repos are often used as the "source of truth" where comparison and merging happens), but it is worth seeing how it works because the same exact feature is used on remote repos like GitHub when you do a PR and also acts as a good way to solidify what commits are and how they integrate when merged.

Here's the exercise:

1. Make a new branch off `master` and call it whatever you want, I'm going to call it `alpha` for this use case. Then make another branch off `master` and call it something else clear, I'm going to call mine `beta`.
2. To verify these branches contain the same commits, run a `git log` against each branch (you'll need to run a checkout on each one first before running the `log`)
3. Change something in the repo on the branch `beta` and `commit` it. (Go nuts here with your changes, we're not going to integrate these changes into `master` anyway)
4. Switch onto the branch that you want to integrate the new changes into that you did on `beta`, in this case `alpha`. This bit is important as the `merge` command uses your current branch as the default target.
5. Run the command `git merge beta` and you should see an output like

```bash
    ➜  linter git:(alpha) git merge beta
Updating cc44gfe..3b374c3
Fast-forward
 linter/Cargo.toml | 1 +
 1 file changed, 1 insertion(+)
```

 along with your code changes now being present in `alpha`. To further verify, run a `git log` on `alpha`, and you will now see your recent commit from `beta` right there in your `alpha` branch, inserted at the top. Importantly, `beta` has not been deleted as part of this process. It's still there and can be switched to at will and now has the same exact commit history as `alpha`.

This quick local branch workflow might be useful to you if you want to test something out on a branch of your feature branch for a task in isolation, and you then want those changes in your feature branch. A branch is just a chain of commits so it makes no difference in the end when you go to pull request the final changes if those commits are the result of three different isolated testing branches all merged together, or were redeveloped on just one branch.

To finish, delete those `alpha` and `beta` branches before moving on. We're about to try something more serious and push / pull code from a remote.

## Remote code

 <!-- TODO YOURE HERE -->

Remote repos are really no different than what's on your computer right now. Somewhere on the internet there's a computer with `git` installed and a clone of the code sitting there waiting for you to merge changes to it, just with a fancy UI wrapped over the top of it. In our case, I'm using GitHub to manage this copy of the code and it and its `master` branch are treated as the source of truth. But under the hood both Bitbucket, GitHub, and Gitlab all just use `git`.

### Setting a remote

When you cloned this repo down onto your machine one automatic thing happened with regard to remotes, the `master` branch was automatically configured to have its GitHub location set as its "remote origin". What this does in practice is set a target for you to `push` to and you likely won't ever need to set the remote yourself (at least for the repo as a whole, individual branches are a slightly different matter). GitHub themselves have some great docs on the concept of a remote [here](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories). Any repo you clone down from GitHub will have the GitHub repo as the default remote.

To see what remote(s) your current clone of this project has, run:

```bash
git remote -v
```

To illustrate how this works and might change depending on the project I'll show two examples. Right now as of this commit, the current repo _has no remote_. I have been working on the text here while controlling the project with `git`, but without a remote repo in Bitbucket, GitHub, or elsewhere. This is a totally valid way to use `git` for pure version control purposes, but when running `git remote -v` I see the following:

```bash
➜  git-training-grounds git:(master) ✗ git remote -v
➜  git-training-grounds git:(master) ✗ 
```

Nothing. You however will see something different, more like this

```bash
  project_name git:(master) git remote -v

origin  git@github.org:example_project_name_here.git (fetch)
origin  git@github.org:example_project_name_here.git (push)
```

As you can see the origin of the `example_project_name_here` project is a GitHub repo for both `fetch` and `push`. This means that by default when running those two commands, `git` will reach out at that address to retrieve (`fetch`) or update (`push`) code. This can be reconfigured at any time via the options in the `remote` command. Most projects you work on will have a remote configuration like the above and you will never need to change it. This configuration will look very different if you ever encounter a "forked" repo and in that situation you may need to manually set your remote configurations to something other than the default. This falls under the scope of "forking" which is a concept not covered in this guide...yet.

## Synchronizing local and remote

Using VCS in a local directory all by yourself is useful, but connecting it to central remote repo and synchronizing the two (along with everyone elses changes) is where the real power of `git` starts to shine and make distributed software development possible. In this section we'll go over how to keep your local copy up to date with a remote copy, and how to send your changes to a remote.

### Updating your local repo from a remote (git fetch)

`fetch` is one of the two main ways to update local code _from_ a remote. What happens behind the scenes is a little sneaky though, as no locally checked out branches will be updated by running just a `fetch`. `fetch` will update the contents within a hidden folder in your directory called `.git` and synchronize what it finds there with the remote you fetched from. What that does for you, is make your local checked out branches aware that remote changes now exist (if there are any), but it's up to you as to how to integrate them. In practice, a `fetch` is followed with `pull` or `merge` command to actually do the updating of the desired branch.

Seeing this in action will require an element of timing in a "real" repo. Next time someone merges a PR into `master` in a remote repo on any project you're working on or have cloned down, follow these steps:

1. Locally `checkout` the `master` branch and run a `git status`. You probably won't see anything too interesting, a message about how your working tree is clean, or maybe something about changed files if you've changed something. Note that your local repo has no idea that someone has merged changes on the remote, as no output indicating that is present in the `status` output.
2. Now run a `git fetch`, followed by another `git status`. You will see something rather different now. No local files will have been changed but you will see an output like:

    ```bash
    DEV LOCAL rm-terrell@server:~/ui_code$ git status
    On branch master
    Your branch is behind 'origin/master' by 88 commits, and can be fast-forwarded.
    (use "git pull" to update your local branch)

    nothing to commit, working tree clean
    ```

3. Once again `git` has a useful output for us that tells us what to do. In this case simply running the command:

    ```bash
    git pull
    ```

    with no other arguments will update our `master` branch with the newest changes we got from the remote via the previous `fetch`. Do that in your real project repo, and take a look at the output from the `pull` command as we will be covering what a `pull` is shortly. Can you tell what it's doing?

### Seeing remote branches, locally

You may have noticed something funny in the last few sections if you've been using the `git branch` command to see local branches, namely after running a `fetch` command followed by a `branch` command you might wonder _where are all those remote branches I see in GitHub? Why aren't they listed locally after fetching the remote?_ They're there, but to avoid clutter in your console `git` hides remote repo branches behind the `-r` flag. There won't be many of these for this repo, so jump into another repo for another project you have cloned down (if you don't have one, the VS Code repo found [here](https://github.com/microsoft/vscode) would make for a good, realistic project to try this), and run:

```bash
git branch -r
```

And you will be presented with a list of all branches from the remote, and you can `checkout` any of these you wish. To see all branches that are in the remote _and_ locally, use `-a` instead of `-r`.

**You will frequently use this workflow when reviewing pull requests.** When a coworker opens a pull request, they did so on a remote branch. Nothing mysterious about that now. You can then look at their code locally by running a `fetch` (to synch their branch from the remote to your local copy) followed by a `branch -r` to gets its name again if you need it, followed by a `checkout` on that branch, and you will now have their code right in front of you and able to be ran just like on their machine. You can even run a `log` to see their commit history, and `diff` commands between their branch and `master` to see a summary of changes, all without using GitHub. And you can do it from anywhere, on any machine, even one without a UI like a pure Linux server.

### Updating your local branch from a remote (git pull)

You got a preview of this in the [fetch](#updating-your-local-repo-from-a-remote-git-fetch) section, but `fetch` does not update local checked out branches, it just synchronizes remote branch data to your local copy stored in the `.git` folder.

To actually update a local branch you will run a `pull` command. `pull` will do the work of taking all the commits from another branch, and integrating them into your current branches commit history. If a coworker made a commit 3 months ago that just got merged, that commit will show as 3 months ago on your end too if you run a `log` and find it. This keeps the entire history of the project perfectly in synch across copies of the repo regardless of when edits are made.

When you run a `pull` after a `fetch` you'll see something like this (example output from another project):

```bash
DEV LOCAL rm-terrell@server:~/ui-code$ git pull
Updating 55e4f3472f..fe83b169c9
Fast-forward
 .gitignore                                                                            |   5 +
 package.json                                                                          |   1 +
 app/playwright/pages/MainPage.ts                                                      |  81 +++++++++++++
 //more files updated but removed for brevity//
```

There's a few useful things happening here. First off, without any other args `pull` defaulted to pulling the updated remote copy of the branch you are on (it can target other branches too). The first line:

```bash
Updating 55e4f3472f..fe83b169c9
```

tells you the range of commits updated. If you fancy yourself a `git` wizard, when you do this for yourself you can take those commit hashes it output and run them through a `git diff` and you'll be able to see the exact changes that were made in the update you just pulled down. Very useful when catching up on the changes that have been made to a project you've been away from for a while. If many many files were changed you can also filter down the `diff` command to just a single file to see what changed by changing the diff with the commit hashes to:

```bash
git diff hash1..hash2 -- path/to/file/here
```

The rest of the output above tells you the method used to perform the update, which in this case was a "fast forward". `git` has a few methods of integrating changes, but "fast forward" is the usual default and you _probably_ won't ever have to know about the others. They make for thrilling late night reading though.

The rest of the output will be a list of files changed with a short summary on the right of what happened to them per a shorted version of a `git diff`.

**The usual workflow**: Alternatively, one can use just the `pull` command to skip the need for a `fetch` step if you just want to update your one branch (usually `main`/`master`). To do this, you'll need to specify the location of the remote to `pull` from (remember, `pull` on its own defaults to local branches). Most of the time this will be the default `origin` location. So following the previous example of updating `master` after a merge you'd do the following:

1. After someone merges a change into `master`, `checkout` master locally and then run

    ```bash
    git pull origin master
    ```

    and you're done. For many people this is the preferred workflow, as most dev tickets revolve around newest `master` and that branch only. Just know that your local repo copy will be unaware of other branches and their changes from the remote if you exclusively work this way, and you'll need to run a `fetch` before working with said branches.

It should now be clear why a "_pull_ request" is called a "pull request". You are requesting the system to run a `pull` on _your_ branch and integrate it into the primary branch of the repo.

### Sending your local branch to a remote (git push)

When you "push" a branch to a remote, you are telling `git` to send a copy of your current branch up to the remote of the repo. But how does `git` know where to put your new code / branch that you created locally?

It doesn't.

Remember how `git` is an imperative tool that does very little automatically? Well it also doesn't automatically set which branch you want to use in the remote to track your code, a concept called an "upstream". To see this in action, create a new branch locally in this repo, then run a `git push`. You will see something like this:

```bash
➜  project_name git:(new_branch_name) git push
fatal: The current branch new_branch_name has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin alpha

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.
```

This is an intentional design choice in `git`. `git` allows you to set the remote tracking branch (the upstream) to whatever you want and doesn't assume by default it will be the same as your local, although most of the time you will want just that. If you are truly confident you will never ever want to set an upstream branch anything other than what you name them locally, you can follow the instructions at the end of the output and change your `git config` to name them automatically. I would advise against this however, and instead when you create new branches locally to manually set the upstream via the output error message command. If the branch you're working on came from GitHub via a `fetch` instead of being created in your local copy, you will not need to configure this.

Once set you can now freely run a `git push`, and `git` will send your code to GitHub. Feel free to create new branches, set their upstreams, and push them to this repo as you want. Once pushed, you should be able to go into the GitHub page for this repo and click "Branches" to see all the branches your branches there. If they truly are temporary branches, please clean up after yourself and delete them once you're done with them, just to cement that as a habit. Repos cluttered with hundreds of old branches aren't objectively bad, but they can be confusing to new team members and make it harder to find the branch you're looking for.

You are also not limited to pushing just the branch you are currently on. If you run a `git push origin another-branch-name` on a different branch in the repo that you don't currently have checked out, `git` will push the specified branch (`another-branch`) to the remote repository (`origin`). It will NOT push that other branch onto the one you currently have checked out thus mixing the two together (a common concern). You are also free to use the fully specified command for the branch you are currently on, for example `git push origin current-checked-out-feature-branch-name`.

## The whole workflow loop

Putting all this together, the typical workflow loop for `git` will look like this.

```plaintext
1. Create a Branch Locally
---------------------------
Local Repository:
  main
   |
   |-- (create branch) --> feature-branch

Commands:
  git checkout -b feature-branch

2. Commit to the Branch
------------------------
Local Repository:
  feature-branch
   |
   |-- (make changes) --> commit

Commands:
  git add .
  git commit -m "Your commit message"

3. Push the Branch to Remote
-----------------------------
Local Repository:            Remote Repository:
  feature-branch             origin/feature-branch
   |                          |
   |-- (push) -->             |-- (new branch created)

Commands:
  git push origin feature-branch

4. Merge Branch to Main in Remote Via a Pull Request in GitHub
--------------------------------------------------------------
Remote Repository:
  origin/feature-branch
   |                          origin/main
   |-- (create PR) -->        |
   |                          |-- (merge PR) --> updated origin/main

Commands:
  (Create a pull request in GitHub and merge it upon approval)

5. Pull Updated Main Branch Locally
-----------------------------------
Local Repository:            Remote Repository:
  main                       origin/main
   |                          |
   |-- (pull) -->             |-- (updated)

Commands:
  git checkout main
  git pull origin main

6. Delete the Feature Branch in Remote
---------------------------------------
Remote Repository:
  origin/feature-branch
   |                          origin/main
   |-- (delete) -->           |-- (unchanged)

Commands:
  Deleting the feature branch is often handled via the merge command in GitHub, or via the UI

7. Delete the Feature Branch Locally
-------------------------------------
Local Repository:
  feature-branch
   |                          main
   |-- (delete) -->           |-- (unchanged)

Commands:
  git branch -D feature-branch
```

## PR Exercise

Remember your training branch `your-name-here-training-branch` that you added a commit to the `street_art.md` file? We're going to integrate it into `master` now with the above info as your guide.

1. Switch to the `your-name-here-training-branch` branch via `checkout`
2. Push the changes up to the remote repo via `git push`. You will need to set the upstream branch as the error message will tell you, or run `git push origin your-name-here-training-branch` to push it up without setting the upstream explicitly.
3. The final output will provide a link to open a PR on that branch. Click it to load the GitHub UI and create a PR. Alternatively, you can navigate to the GitHub repo for this project and click the "Create Pull Request" button and select your branch.
4. Feel free to open the PR and tag me as a reviewer, and as long as it meets the criteria of adding something to the `street_art.md` file (that isn't too objectionable), I'll approve it and merge it. I do have a life and full time job but I'll get to it soon considering how simple such a change is.
5. Now that `master` is updated with your changes, locally `checkout` `master` and run a `git pull`, you will now see your changes in the `street_art.md` file.
6. Delete your training branch locally and remotely if it wasn't deleted as part of the PR process.
7. Welcome to open source!

## Onwards

Unfortunately merging branches together sometimes doesn't go so well and results in something called a "merge conflict" that requires manual editing of files on a case by case basis. Open up `4-merge_conflicts.md` to read about how to deal with them.
