# Getting yourself in trouble

## Creating weird commit histories and diffs across branches

Feature branches require care when merging changes from `master` back into them. A common situation involves a feature branch for a big feature build out, a sub feature branch off it for an individual devs work, and then some new work comes into `master` that is needed in both. Visually like this:

```plaintext
master --> bug-fix-commit
  |
  |--- feature-branch
          |
          |--- sub-feature-branch
```

The temptation for the dev working on `sub-feature-branch` is to pull `master` straight into `sub-feature-branch`, and then someone else pulls `master` into `feature-branch`. However, with what you know from previous `How git tracks changes` section you know that a merge creates a new commit. So, if you merge `master` into the sub feature branch and then also into the feature branch, you'll have two merge commits that are identical in content (both containing the bug fix) but with different hashes. This is because each merge commit is created on branches with their own different commit history since the parent commit is different in the two branches. Thus when you go to open a pull request between the sub feature and feature branch in a web source control tool like GitHub or Bitbucket you will see extra commits between the branches, and changes visualized in the's diff UI that "shouldn't" be present because the changes already exist in the destination branch. This of course will be the intuitive expectation if you think of `git` as tracking changes, which it actually doesn't do directly, per the discussion in "How git tracks changes" in the first section of this guide.

To avoid this problem, always merge master into the base feature branch first, and then merge the feature branch into the sub-feature branch. This ensures that the commit history remains clean and avoids duplicate merge commits. However, if you find yourself in this situation, you can use the `patch` command to move changes from your incorrectly merged branch into a fresh branch off `feature-branch`.

Let's create this situation on purpose to see it in a safe environment and then untangle the mess.

1. Open your terminal and create a new directory somewhere on your machine (I use `/Documents/repo/`), then run a `git init` command in it and make a file. Heres the commands to do it fast:

    ```bash
    cd Documents
    mkdir weird-diff-example
    cd weird-diff-example
    git init
    touch functions.py
    ```

2. OPen the new directory in your code editor and add this content to `functions.py` and save it. Then add it to the staging area and commit it. For all of these steps, add verbose commit message mentioning the branch you're on to make it easier to understand in `log` history later.

    ```python
    def first_function():
        return "First function"

    def second_function():
        return "Second function"
    ```

3. Now create a new feature branch and switch to it, then add this content to `functions.py` and commit it.

    ```python
    def first_function():
        return "First function. feature work"

    def second_function():
        return "Second function"
    ```

4. Now create a branch `sub-feature-branch` off of `feature-branch` and switch to it. Then change the contents to the following and commit it.

    ```python
    def first_function():
        return "First function. feature work. sub-feature work"

    def second_function():
        return "Second function"
    ```

    What we've done here is simulated a feature being developed on a main `feature-branch` and on a `sub-feature-branch` like an individual engineer building upon it.

5. Now switch back to `master`. We're going to simulate a bug fix being merged into `master` that needs to be integrated into our feature and sub-feature branches. Add the following content to `functions.py` and commit it.

    ```python
    def first_function():
        return "First function. feature work. sub-feature work"

    def second_function():
        return "Second function. critical bug fix"
    ```

6. Get ready to make a mistake. Switch to `sub-feature-branch` and run

    ```bash
    git merge master
    ```

    **For those who have never run a merge locally**: you'll be presented with a text editor to write a merge commit message. Save and close it without any edits (`!wq` if you're in VIM, or just close the editor tab if it opens in another editor) and you'll see that the merge was successful.

7. Now switch to `feature-branch` and run the same merge command.

    ```bash
    git merge master
    ```

    Again, save and close the merge message editor without any edits. At this point if you switch between the branches and look at `functions.py` you'll see that the bug fix is in both branches, which seems to imply that nothing is amiss. However, `git log` will show the truth.

8. With `sub-feature-branch` checked out, run `git log --oneline --graph --all` and you'll see an output like the following:

    ```bash
    *   c4c112e (feature-branch) Merge branch 'master' into feature-branch
    |\  
    | | *   e73f43a (HEAD -> sub-feature-branch) Merge branch 'master' into sub-feature-branch
    | | |\  
    | | |/  
    | |/|   
    | * | 6887eef (master) bug fix
    | | * 75b17b5 sub feature work
    | |/  
    |/|   
    * | 55c1bbb feature branch
    |/  
    * 2252818 initial commit
    ```

    Meanwhile if you had pulled `master` into `feature-branch` first and then pulled `feature-branch` into `sub-feature-branch` you would see the following:

    ```bash
    *   bbf53db (HEAD -> sub-feature-branch) Merge branch 'feature-branch' into sub-feature-branch
    |\  
    | *   db25512 (feature-branch) Merge branch 'master' into feature-branch
    | |\  
    | | * c19288d (master) bug fix
    * | | d77437f sub feature work
    |/ /  
    * / 3de9ba8 feature branch work
    |/  
    * 1f9e317 initial commit
    ```

    Take some time to look at these two graphs and see the differences. The result of this difference will be that when you open a pull request between `sub-feature-branch` and `feature-branch` in Github you'll see the bug fix changes in the diff, even though they are already present in `sub-feature-branch`. GitHubs's diff algorithm is not smart enough to see that the changes are already present in the destination branch because it bases the changes off of commit hashes which in this case are different. If you want to try this in the wild, make this same branch structure in one our product repos that's active, wait until changes come in to `master`, and then run this incorrect merge structure. Then push up both branches to GitHubs and open a PR between them (just the preview stage, no need to full open it) and you'll see the changes that came into `master` in the PR diff.

9. Now to fix this mess. One option is to rewrite history using the `rebase` command in `git`, but we're going to go for the slightly more blunt but easy to understand approach of making a new branch off of `feature-branch` like we were starting over, and then grab the raw changes via a generated `.patch` file. First, create a new branch off of `feature-branch` and switch to it.

    ```bash
    git checkout -b sub-feature-branch-v2 feature-branch
    ```

10. Now we're going to create a `.patch` file of the raw changes between `sub-feature-branch` and `feature-branch` which will contain all your work.

    ```bash
    git diff feature-branch..sub-feature-branch > change-summary.patch
    ```

    This will create a `.patch` file in your directory with the changes between the two branches. If you open it you'll see a standard `diff` summary of all changes between your two branches, even across multiple files. Now we're going to apply this patch to the `sub-feature-branch-v2` branch. First switch to your new branch and then run the following:

    ```bash
    git apply change-summary.patch 
    ```

    If you now run a `git status` you'll see that `functions.py` has been updated with the changes from the patch file. This will scale regardless of how many changes you might have between two branches and the process will be the same. You can now `commit` those changes as usual. Use this technique any time you need to move changes between branches programmatically.

    **NOTE**: This technique of generating patch files with `diff` is useful in any situation you need to move a set of changes from one branch to another regardless of the underlying branch or commit structure. It's a powerful tool to have in your `git` toolbox.

    An alternate way to do this is to [use `git format-patch`](https://git-scm.com/docs/git-format-patch). However, `format-patch` creates patches based on the differences in commits between the two branches, not the raw changes. Because of that, in this specific situation the resulting `.patch` file will contain the incorrect state for `second_function` due to the merge commit that was created in `sub-feature-branch` and applying the patch will fail. Using `diff` instead has the advantage of working in a more intuitive manner based on raw changes. Another alternate way to do this is via [the `cherry-pick` command](https://git-scm.com/docs/git-cherry-pick), however it will require you to know the commit range to be cherry picked (can be found with `log`) and will immediately create a new commit in the branch you're on. The `patch` method is more flexible and allows you to review the changes before committing them.

## Committing sensitive data

Once you commit something in `git` it is stored in history "forever". This is because as you've seen `git` almost never actually deletes data, it just adds new data that reverts previous data. Even if you delete something sensitive and make a new commit, the code state before you deleted it can always be reloaded via commands you already know (`git reset commit_hash_here`, targeted uses of `git blame`, `git log`, and `git diff`). This is how people steal server credentials from open source projects, by creating scripts that comb commit histories of projects whose maintainers think they deleted something sensitive when all they did was override it with a new commit, leaving the rest in history. If you commit a credential and then push it out to a remote `git` repo for example you have a very difficult situation on your hands as you now need to rewrite history to remove it.

Your first step is to contact the relevant IT Security folk and update / change that credential so its not valid anymore. Your next step is to find someone who knows how to use `rebase` / `reflog` to erase that commit out of commit history. If you're feeling brave, grab the `git` documentation and your must trusted AI buddy and figure it out yourself. The steps in doing this can range from "tricky" if that branch was never pulled by anyone else, to "incredibly tricky" if that branch was pulled and used to base other work off of. Such rewriting of history is beyond this guide but know that `rebase` / `reflog` are the features you're looking for and that is _is_ possible to remove it. But it will be painful and tedious.

Preferably, think twice before committing sensitive info and remember that you can use `.gitignore` to prevent files from being tracked with `git`, or investigate other tools to inject that data into your application outside of source control.

## Errant commits to master

You made a commit straight onto `master`. Woops. Even worse it has a lot of changes that you need and are a pain to copy paste. This is not too bad to fix and involves two steps, first using `git cherry-pick` to move the commit(s) to a new branch, and then using the `reset` command to remove the commit(s) from `master`. Lets create this situation in this repo and then fix it. (Know that `master` is locked down so that you can't push commits to it so theres no way to mess something up here).

1. Make a change to something in this repo, say `src/modify_me.py` and commit it to `master`. Then make another change somewhere and commit it to `master`. You now have two commits on `master` that you want to move to a new branch.

2. Create a new branch off of `master`.

    ```bash
    git branch new-branch
    ```

3. Now we're going to use `cherry-pick` to move the commits from `master` to `new-branch`. First, find the commit hashes of the commits you want to move via `git log`. Once you have the two commit hashes run the following:

    ```bash
    git checkout new-branch
    git cherry-pick commit_hash_1 commit_hash_2
    ```

    This command is a little wild to use if you've never seen it before, as when ran successfully it instantly creates a new commit in the branch you're on with the changes from the commit you specified. If you run `git log` you'll see the new commit at the top of the log along with the changes now being present in the file(s) you modified.

4. Now that we have the errant commits saved to a new branch we can remove them from the `master` branch. This will prevent issues if you go to `pull` `master` and `git` then struggles to integrate on top of your errant commits. There's two ways to do this. One would be passing the commit hashes into the `reset` command, but in this case since we know that the last two commits are the ones we want gone, so we can just run:

    ```bash
    git checkout master
    git reset --hard HEAD~2
    ```

    This will remove the last 2 commits. If your commits are not consecutive it will be best to use the commit hashes.

5. Problem solved. Switch back to your `new-branch` and carry on. Do note that this final reset command is very danger as it is one of the few situation in `git` where you can truly lose work.

**WARNING**: When your work has stayed local, throwing the `reset` hammer is fine. But if you have pushed a branch to a remote repo, and other people have built work off of that branch, and you then `reset` commits out of the branch and push it again, you have effectively rewritten history and will cause a huge headache for everyone using that branch as they will now start encountering errors on merging their branches due to these differing histories. If this situation occurs you will need to contact the people who have built off of that branch and let them know that they need to `rebase` their work off of the new branch head.
