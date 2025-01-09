# Merge Conflicts AKA Code Thunderdome

Merge conflicts are probably the most consistently mystifying and stress inducing part of using `git` for developers, the author included. Like previous sections we will perform this merge conflict resolution entirely via the command line to see how the mechanics work, but this is one of the situations in `git` where full UI tools begin to shine in real world workflows. As such, work through this section as a way to give context and clarity to the UI merge conflict tools you will use in programs like VS Code, Source Tree, GitHub Desktop, and others. They all build off the same text based output that `git` creates, and handling a conflict via CLI will make the others a breeze. Your UI focused coworkers will probably think you're a wizard also.

## What is a merge conflict and is it the end of the world?

A merge conflict occurs when two branches have made different changes to the same line of a file from the same base commit, and `git` cannot automatically determine which change to keep. Note for a moment how interesting this is. `git` does NOT just default to whatever change was the newest and keep that one (like many other softwares that track changes would do), thus potentially losing the older change that might actually be the "correct" one, or at least contained changes that needed to be integrated somehow. Instead, a "merge conflict" is just a special state in `git` where two sets of changes are compared and a human decides which one to keep and a new commit is generated to store that final choice. It is less an "error" and more a "decision point".

## Creating and solving a merge conflict

Lets get ourselves in trouble on purpose using two local branches to see how to untangle a conflict. The workflow will be the same when pulling and merging from a remote repository but we'll do it all local to simplify things.

1. Find and open the file `modify_me.py` on the `master` branch. In there you'll see a `print()` function. We're going to create a rather realistic merge conflict there.

2. Create a new branch called `feature-branch` (`branch` command) based off `master`. We're going to create our merge conflict trying to merge into this new branch, and that way if this all goes horribly wrong just delete all the new branches and start over and avoid messing with `master`.

3. Create two new branches based off the new feature branch (remember that `branch` won't switch to the new branch automatically). The first one called `alpha-branch`, and the second one called `beta-branch` also based off of `feature-branch`. Make sure you don't accidentally base beta off of alpha (running the `branch` command from the alpha branch without specifying a different base). If you are unsure of the command for basing a new branch off one you don't currently have checked out and aren't able to figure it out, click the spoilers below.
    >! `git branch alpha-branch feature-branch`
    >! `git branch beta-branch feature-branch`
    In this case there are no new commits yet so using a visualization command like `git log --oneline --graph --decorate --all` won't show much. Remember, a branch is just a pointer to a commit, and until we make some new commits the branches will all point to the same commit. This will soon change.

4. `checkout` the alpha branch and edit the `print()` function to output:

    ```python
    print("Alpha feature. Current day of the week:", time.strftime("%A"))
    ```

    And then `commit` your changes to the branch.

5. `checkout` the beta branch and edit the same `print()` function to output:

    ```python
    print("Beta feature. Current week of the year:", time.strftime("%U"))
    ```

    And then `commit` your changes to the branch.

6. You now have a situation where two branches started from the same commit, then diverged and made changes to the same line of the same file. Now we can merge one branch into the other and create a merge conflict.

7. `checkout` the `feature-branch` and run the command:

    ```bash
    git merge alpha-branch
    ```

    This should complete without issue since `feature-branch` has not changed since we created it.

8. Now while still on the `feature-branch` lets merge in the changes from `beta-branch`. Run the command:

    ```bash
    git merge beta-branch
    ```

    You should see an output like this:

    ```bash
    ➜  git-training-grounds git:(feature-branch) ✗ git merge feature-branch beta-branch
    Auto-merging src/modify_me.py
    CONFLICT (content): Merge conflict in src/modify_me.py
    Automatic merge failed; fix conflicts and then commit the result.
    ```

9. Breath. This is fine, and as usual with `git`, if you run a `status` you will probably get some useful guidance on what to do next.

    ```bash
    ➜  git-training-grounds git:(feature-branch) ✗ git status
    On branch feature-branch
    You have unmerged paths.
    (fix conflicts and run "git commit")
    (use "git merge --abort" to abort the merge)

    Unmerged paths:
    (use "git add <file>..." to mark resolution)
            both modified:   src/modify_me.py

    no changes added to commit (use "git add" and/or "git commit -a")
    ```

    As you can see, running a `git merge --abort` will allow you to back out of the merge if this merge was an accident. We're going to follow `git`'s other piece of advice and `(fix conflicts and run "git commit")`.

    Open the file `modify_me.py` in your text editor and you will see that `git` has actually modified your file with some special markers to show you where the conflicts are. (**Note**: many editors and IDEs will automatically add a lot of UI overlay on merge conflicts to help you. This is good, but if the output is confusing, try opening the file in a basic text editor like VIM, or just use `cat` to see the file contents in their raw form.)

    You'll see the following:

    ```python
        def main():
    <<<<<<< HEAD
        print("Alpha feature. Current day of the week:", time.strftime("%A"))
    =======
        print("Beta feature. Current week of the year:", time.strftime("%U"))
    >>>>>>> beta-branch
    ```

    There are a few things going on here:

    1. Remember that `HEAD` refers to the current branch you are on, which in this case is `feature-branch`, so changes between the `<<< HEAD` and `===` markers are changes already on `feature-branch` from before you started the merge, from the first commit you merged that originally came from the `alpha-branch`.

    2. The changes between the `===` and `>>> beta-branch` markers are the changes from the `beta-branch` that you are trying to merge in but conflicted. These are also known as "incoming changes".

    These sets of special characters are how `git` indicates a merge conflict. To fix the merge conflict you simply need to remove them, and change the code to be in the state you desire. The special characters are just there to show you the two different versions that `git` can't automatically resolve in a clear and separated way.

10. In this case, lets assume the final solution is _not_ purely the code from either branch but a blend of the two. This is a common scenario where two developers are working on parallel features that are both needed but crossed over the same lines.

    To keep track of which changes you want as your final set modify the code between the `HEAD` and `===` markers to look like this:

    ```python
        def main():
        <<<<<<< HEAD
            print("Current day of the week:", time.strftime("%A"))
            print("Current week of the year:", time.strftime("%U"))
        =======
    ```

    Then delete the code between the `===` and `>>>` markers, along with the markers themselves. Finally remove the `<<<<<<< HEAD` line as well. Your final code should look like this:

    ```python
    import time


        def main():
            print("Current day of the week:", time.strftime("%A"))
            print("Current week of the year:", time.strftime("%U"))


    if __name__ == "__main__":
        main()
    ```

    **Note**: the workflow of adding your final code between the 'HEAD' and '===' markers, and then deleting everything else is just one way to resolve a conflict. The author has found that to be a good way to keep track of the desired end result of the code, but you can also just delete everything and write your own code from scratch if you prefer. Or put it all in the incoming block. It's really up to you and what you find most comfortable for keeping track of changes, as merge conflicts often involve many sets of conflicts across many lines within a file so its best to work in a way that is consistent and repeatable to mentally keep track of what you're doing.

11. The merge conflict is now resolved. At this point you can `add` your files via a `git add` and you should see this in the terminal via a `status`:

```bash
➜  git-training-grounds git:(feature-branch) ✗ git status
On branch feature-branch
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   src/modify_me.py
```

If your merge had multiple files conflicting the `status` would list them out in the `status` command for you to work through one by one. Modern editors will often highlight files with `git` conflicts too in some way.

You can now `commit` your changes to finish the merge. Note that the process of resolving a merge conflict (and merging generally) creates a new commit. Again, `git` rarely loses information, it simply adds new information.

To visualize what you've just done, run the following command as a preview of the next section:

```bash
git log --oneline --graph --decorate --all
```

And you'll see a text based graph of your branches and commits that you just created and merged. Hit `q` when you're done to exit.

## Practice

If you want a harder challenge, delete those branches and do it all again, but this time add new functions (make them different) to the python file on the same line in each function, on top of changing the original `print()` statement to print different things. This will create a multi-line merge conflict that you will need to resolve. Multi-line merge conflicts tend to become very visually confusing but are really just the same concept as above repeated.

Another good exercise would be to do the same merge conflict generating workflow, but then try to solve it with UI tools like VS Code.

Go forth and fear no merge conflict.

Open up the next section, `5-exploring_the_past.md` to see how the various tools for exploring `git` history.
