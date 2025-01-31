# Exploring the past

Some previous sections have mentioned the `log` command and how useful it is for seeing back into the history of project. This section explores it more thoroughly.

## Seeing the commit history of a file (git log)

You can see a history of all commits in the repo for a file via the `git log` command. This will come in handy when you're working on large project with a lot of work that might be changing in real time and you want to make sure your branch has certain commits in it.

Heres an anonymized but real example from a project where the I was trying to hunt down when a certain file changed in order to understand _why_ it changed

```bash
user@900c945cee32:/workspaces/app$ git log app_directory/src/main/java/com/File.java
commit 2e4128s6153b49862f8jh4c37f2bfb68c91e59565 (origin/other_user/fix_bug)
Author: other user <other.user@email_address.com>
Date:   Sun Sep 1 19:54:41 2024 -0700

    Fix for app_directory

commit 7ac2c24e9ca056030e27x0358b41a799g2d8e8e5
Merge: 526b9b5 9dc0714
Author: another user <another.user@email_address.com>
Date:   Thu Aug 29 10:32:20 2024 -0500

    Merge branch 'upgrade' of bitbucket.org:company/app into other-upgrade

commit 444db8g70838af2a0493a81476fg13c4e0db4737 (tag: release45)
Author: other user <other.user@email_address.com>
Date:   Wed Aug 7 15:17:06 2024 +0000

    Merged in other_user/update-mysql8 (pull request #86)
    
    Update to mysql 8
    
    * Update to mysql8
    
    * update pom.xml
    
    
    Approved-by: Manager Name
```

The 3rd commit from the newest with a hash ending in `b4737` was what I was looking for, along with the pull request number that let me see the whole context of the changes. That last bit is an incredibly useful thing to know about as it gives you the power to hunt down when and why anything in your project changed, and who changed it via its Pull Request in GitHub and any linked tickets.

To see this in action in this repo run this command to see the whole history of the Street Art file you modified in the previous sections:

```bash
git log /src/street_art.md
```

Assuming you are still on your own branch we made in the previous sections "Making Changes", you should see your own commit with its message in there at the top, along with everyone elses commits who have every touched that file.

### Exercise: using `log` to power `diff`

Go ahead and run the `git log` command against any file in any project you're working on to see the full commit history of it. Then here's the powerful bit. To see _what_ changed between those two commits, copy the commit hashes of the last two commits and run them through a `git diff` like this

```bash
 git diff commit-hash1-here commit-hash2-here -- path/to/your/file/here
```

and you will now see a diff of what changed for that file for those two commits you were looking at in history. This is incredibly powerful and can be used to narrow down on how files have changed between any two points in time in the system, all from the command line on any system, even a server without any UI available to browse files. You can also give the diff command branch names instead of commit hashes to compare changes that way.

## Seeing what the last changes were for a line or file (git blame)

A similarly useful but more focused tool for finding out when and how something changed is `git blame` combined with `git diff`. This sounds much more mean that actually it is. No, `blame` doesn't automatically email your coworkers saying a bug is their fault. `blame` is like a more focused version of `log` where instead of seeing the whole commit history for a whole file, you can see the most recent change info for just one line of code (or a range of lines), which is often all you want in day to day software work anyway. "Who changed this darn color hex to orange and when???" You can answer that question now.

### Exercise: using `blame` to power `diff`

Get the line number of this line of text you're reading right now. In the console run

```bash
git blame -L line_number_here, line_number_here path/to/file
```

(yes the same line number twice, the two values represent a start and end point and we're after just one line) and you'll get an output kind of like this

```bash
b2fd2d1c (User Name 2023-07-01 14:13:27 -0400 7)     "dockerComposeFile": [
```

the above `blame` is from another, real project. The important information here you will make use of is that first combination of letters and numbers. That's a commit hash. Now that you have it you know what commit last changed that line, and you can feed it into `git diff` to see _what_ changed, like this

```bash
git diff commit_hash_here^! -- path/to.file
```

and you will now have a diff targeted at your file and informed by the commit that changed just the line you care about, between your current version and the last commit. The `^!` is a shorthand for `commit^..commit` which specified the changes introduced by that commit. This is different than just using a `log` command, as with that output you have no idea which commit changed the line are interested in, and in large file there could be _years_ commits that went by not touching the line in question. `blame` lets you hunt with precision.
