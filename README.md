# Git training grounds

Welcome!

When I first started using `git` at the beginning of my career change towards software I struggled with it a lot. I distinctly remember the first time I saw a "detached head" message in the console and thought it was calling me an idiot. This guide is _the guide that I would have wanted_ when I started out and is structured as such in the order it teaches things. This repo can be treated as a safe place to do silly things with `git`, make branches, push changes, try to merge things, etc all without consequence and with realistic workflows. Feel free to open PR's against the repo here to integrate the exercise changes contained within, no promises I'll be super quick to merge them but I'll do my best.

> "I like that the first 3 commands you teach (status, diff, and reset) I had never used before this and yet they're now my favorite ones."
>
> "He's my go to for git questions"
>
> - Yes Energy co-workers

**Authors note:** Large Language Models like Co Pilot, Gemini, and Chat GPT are _extremely_ good for explaining `git` concepts and its sometimes cryptic errors. That said, I did not write the words of this guide with any AI assistance beyond markdown syntax help and ASCII diagrams. I mention this not as a manifesto against LLM tools (on the contrary as you'll see in later sections you can use `git` to feed data to them for useful workflows) but to emphasize that the order of sections and commands was intentional and not just AI slop that is so prevalent today in guides, and I felt that each section, sentence, and word was important and helpful to you one engineer to another, and not something to just be "summarized" away. Typos and all.

## Cloning down the repo

To get started on learning and experimenting with `git`, assuming you're reading this on GitHub, you will first need to clone this repo down onto your local machine. If you know what a fork is and prefer that, feel free but I don't give explicit instructions on forked work flows here.

You'll first need to decide where to put the contents of this project on your machine, as everything you're looking at in GitHub right now is going to soon be stored on your computer along with presumably other source controlled projects. Personally I like to use the `/Documents/` folder on my computer to hold a directory called `Documents/repos/`, but the exact location doesn't matter. Some people like to use the home directory or `/usr/`, or if you're on a Windows machine with a Linux subsystem you can put it in there too. You'll be able to freely move it later if need be. You could even have multiple cloned copies on your machine in different spots and sync them all with the commands we'll soon be learning. Bonus points for doing that. In case you need examples to follow along I'll use my own setup as example commands.

### Installing git

To install `git` start [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). The official `git` documentation has guides on how to install `git` on Linux, macOS, and Windows, and in some situations your machine may already have it installed. You can check by running

```bash
git --version
```

and seeing if you get an error or a version number. If you get a version number, you're set. If not, install it via the instructions in the above link. While you're at it, bookmark [the docs homepage](https://git-scm.com/doc) for future reference.

### Cloning the repo

To clone this project down onto your machine, change directories to where you want your repos to live via `cd`:

```bash
➜  ~ cd ~/Documents/repos
➜ pwd # this prints the current directory to double check you're in the desired spot
/Users/your_user_name_is_probably_here/Documents/repos 
```

Then run

```bash
git clone git@github.com:RM-Terrell/git-training-grounds.git
```

and then run an `ls` or `dir` (unix / windows respectively) to see the freshly cloned directory.

```bash
➜ ls
git-training-grounds
```

Then `cd` into `git-training-grounds` and if you're using VSCode with it installed on your system PATH run `code .` to open it in a new VSCode window. If you're using a different editor open up that new folder in it and open the file `docs/1-git_fundamentals.md` to get started from there. Whatever editor you're using probably has either a folder history or project history so that you can easily find this cloned project again.
