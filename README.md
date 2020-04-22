

# gituser - Set the user id in a git working copy quickly when using `user.useConfigOnly = true`


## Why you need this

There can be many reasons why you want to use several git identities. It could be either because you want to separate work and personal stuff (see e.g. [here](https://blog.stevenocchipinti.com/2016/12/28/different-author-email-addresses-per-git-repository/) or [here](https://collectiveidea.com/blog/archives/2016/04/04/multiple-personalities-in-git)), or you are using the private commit email feature the git forges introduced to hide your real email from commits (e.g. on [GitHub](https://github.blog/2017-04-11-private-emails-now-more-private/) and on [GitLab](https://gitlab.com/help/user/profile/index?target=_blank#private-commit-email)).

This means you will have at least one identity per git forge, plus a local one for local-only repos. It also means that you can not use a global user identity configured in your `~/.gitconfig` since the user and email ids will need to be different for each working copy.

If you consult your favourite git forge's documentation on how to use the private commit email feature, it will likely tell you to use the `git config [--global] user.email "email@example.com"` command to set it either for the current repo, or globally. Globally is ruled out, since you may want a different one for each working copy (see previous paragraphs).

Practically, this means that you would need to issue a `git config user.email` command after cloning. But what if you don't/forget? If you still have a global user id configured in your `~/.gitconfig`, it gets worse, because git will silently use that if the working copy configuration (`.git/config`) doesn't specify user and email ids of its own.

To give you a prod to not forget setting the proper user and email ids, helpful souls have introduced the [user.useConfigOnly](https://git-scm.com/docs/git-config?source=post_page---------------------------#Documentation/git-config.txt-useruseConfigOnly) parameter, which you set in `~/.gitconfig`. This will cause git to refuse commits and pushes the working copy configuration in `.git/config` doesn't specify user and email ids of its own.

Now you're safe to no accidentally use your real (or the wrong) email address for commits. But manually issue a `git config user.email` command **every time after you clone a repo.** And what was my alias email on forge *X* again? If you search the interwebs for solutions, you will find many variations of schemes based on [`include`](https://git-scm.com/docs/git-config?source=post_page---------------------------#_includes), [`includeIf`](https://git-scm.com/docs/git-config?source=post_page---------------------------#_conditional_includes), or [`alias`](https://git-scm.com/docs/git-config?source=post_page---------------------------#Documentation/git-config.txt-alias) git configuration options. To add a `include` section you will need to edit the `.git/config` file. `includeIf` can only check for the path of the working copy, or for the current branch; this may or may not work with your way of organising things. `alias` finally will force you to remember many new commands.

This is where `gituser` comes into play. It automates all of this, presents you with sensible options, lets you choose one of them quickly, and updates the working copy configuration in `.git/config` for you. The magic is that `gituser` consults a little database of identities (which described below) you set up once, and matches the remotes configured in the working copy against those. Pick one, hit enter, job done. The good thing: you can run `gituser` as many times as you want on each working copy to switch to a different identity should you need to.


## How to use it


### Installation

1.  Copy or clone the `gituser` script to a directory that's in your `PATH`
2.  `cpanm File::HomeDir Path::Class Path::Class::Iterator Config::GitLike::Git Text::TabularDisplay IO::Prompter`

That's all.


### Configuration

1. `mkdir -p ~/.git/id` (or any other path of your choice)
2. \[Optional\] `export GIT_ID_DIR = ${HOME}/.git/id` (only needed if you chose a different path in the previous step)
3. Create a file named `~/.git/id/local` (or `${GIT_ID_DIR}/local`) that contains a git configuration file section with the user and email ids you want to use for local repos:
   <pre>
   [user]
        name = John Doe
        email = j.doe@gmail.com
   </pre>
4. Repeat step 3 for each git forge account you have, and name the files in the `/.git/id` according to the naming convention username@forge (for example `jdoe@github.com`).
5. \[Optional\] You can use symbolic links to use the same ids for more than one account.


### Using it

#### Example 1: local id
<pre><b>example$</b> <i>git init</i>
Initialized empty Git repository in /path/to/repo/.git/
<b>example$</b> <i>vi README.md</i>
<i># edit README.md</i>
<b>example$</b> <i>git status</i>
On branch master

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
<b>example$</b> <i>git commit -a -m "initial commit"</i>

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: no email was given and auto-detection is disabled
<b>example$</b> <i>gituser</i>
No remotes configured. Using local id from /home/jdoe/.git/id/local.
Setting git user id from /home/jdoe/.git/id/local
<b>example$</b> <i>git commit -a -m "initial commit"</i>
[master d74d1bf] initial commit
 1 file changed, 1 insertion(+)
<b>example$</b> |</pre>

#### Example 2: matching remote
<pre><b>example$</b> <i>git clone git@github.com:jdoe/myrepo.git</i>
Cloning into 'myrepo'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 6 (delta 1), reused 3 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), 4.23 KiB | 4.23 MiB/s, done.
Resolving deltas: 100% (1/1), done.
<b>example$</b> <i>cd myrepo</i>
<b>example$</b> <i>git remote add...</i>
<i># add further remotes</i>
<b>example$</b> <i>gituser</i>
This repo has 4 remotes on forges for which you have a user ID:
+------------+------------------------------+
| remote     | forge repo                   |
+------------+------------------------------+
| upstream   | github.com/c-alpha/gituser   |
| my-sandbox | github.com/flurrycat/gituser |
| origin     | github.com/jdoe/gituser      |
| playground | gitlab.com/johnd/gituser     |
+------------+------------------------------+
These are your identities for these forges:
+----------------------+
| ID                   |
+----------------------+
| jdoe@github.com      |
| flurrycat@github.com |
| johnd@gitlab.com     |
+----------------------+
ID for future commits (TAB completes/cycles) [jdoe@github.com]: <i>ENTER</i>
Setting git user id from /home/jdoe/.git/id/jdoe@github.com
<b>example$</b> |</pre>

This example shows two features.

First, the script supports more than one account per git forge. In
the above, hypothetical example, the user has an account `jdoe`
(which we assume is used for work), and an account `flurrycat`
(which we assume is used for personal stuff) on the same git
forge. Both user accounts are offered for that git forge. In our
example, the user has also configured a remote for a second forge,
but for which there is only one identity. Hence, three options are
offered.

Secondly, if there is a remote cunningly called `origin`, and the
user name associated with that remote in the current repository
matches a configured identity, that identity is offered as the
default at the prompt. I.e. in the above example you will simply
have to hit the Enter key to select the identity
`jdoe@github.com`.

In all cases, the prompt offers completion and option cycling with
`TAB`.
