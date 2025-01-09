# Additional config and tools and "learning how to fish"

## Username and email global config

When setting up a new machine to develop on you'll want to set your username and email globally so that your commits are attributed to you. Here's the commands in case you forget them like I always do:

```bash
git config --global user.email "your_name@your_email.com"
git config --global user.name "Your Name"
```

Run those once and you're set for every repo on the machine. If you need to use multiple `git` accounts on the same machine, you can set the `user.email` and `user.name` at the repo level by running the same commands without the `--global` flag. You'll need to do that for each repo in that case.

## AI Tools when learning

ChatGPT and GitHub Co Pilot have shockingly good data sets around `git`. Often times I have found that asking pointed questions to either of those models (something like "What is the command to cherry pick 2 commits by their hash?") are almost always correct. Combined with the official `git` documentation, these tools can be a great way to learn the more advanced features of `git` and how to use them, along with alternative workflows where there are many ways to achieve the same goal.

Some of the portions of this guide, like how to purposefully create a merge conflict, utilized ChatGPT to generate the series of branch commands, and then verified by running locally. I would utilize extreme caution when doing dangerous commands like a `rebase` from an AI output and make use of options like `--no-commit`, `--no-ff`, and `dry-run` where you can to verify the output before committing. One option is run the suggested command as an input into another AI model and make sure the summarized results match expectation. It's not like humans are infallible though with `git` as I once had a coworker use the commit range argument on a `rebase` wrong on my branch and delete all my work. So, you know, be careful in general.

Gemini specifically (per January of 2025) I've found to be a good tool for `git` as its newer models are able to provide accurate links straight to `git` documentation like a hyper contextualized Google search. Go figure considering who made it.

## Diff to AI tool workflow

**Please do not automate your entire code review process with AI.** Non deterministic systems should never have total authority on the building of deterministic code systems.

With that disclaimer out of the way, it can sometimes be useful to use AI tools as a second set of eyes on a change set to review, summarize, and investigate complex diffs like a little minion. This is especially useful in complex SQL queries or data parsing algorithms that have undergone significant changes without unit tests to verify them in a deterministic way. There is a slick way to do this using commands you now know chained together.

With a branch checked out from a PR that contains changes relative to `master` run the following:

```bash
git diff master..$(git branch --show-current) -- path/to/file > diff.txt
```

This command will generate a `diff` for the branches cumulative changes to a file path relative to the `master` branch and save it to a file called `diff.txt`. You can also change this `diff` to dump all changes across all files. You can then pass the `diff.txt` file into an AI tool with a prompt like "please summarize the changes in this diff". I've found the most interesting results come from leaving the prompt open ended as it will sometimes find consequences of changes you didn't see initially. This `diff` workflow can also be used in your own development where you have a set of unit tests that must be updated for a change by feeding the diff into the AI tool, along with the file of unit tests and asking it update the tests accordingly. Or vise versa if you're the TDD type. Such narrow and specific focus for AI tools usually results in reliable outputs.

Use this responsibly and with great skepticism.

## Resource Links

[Gits official reference docs](https://git-scm.com/docs)

[Gits free online book](https://git-scm.com/book/en/v2)

[GitHubs official docs on git](https://docs.github.com/en/get-started/using-git/about-git)

[Learning Git Roadmap](https://roadmap.sh/git-github)

[An excellent blog post on the working directory vs staging area](https://medium.com/@lucasmaurer/git-gud-the-working-tree-staging-area-and-local-repo-a1f0f4822018)

[GitHub official docs on remote repos](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories)
