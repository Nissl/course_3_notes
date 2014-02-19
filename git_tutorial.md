@githubtraining
training.github.com
@matthewmccull
@brntbeer

Two major topics: git and github
Why git, setup git, using git, using github

Why git?
The Linux world was on Bitkeeper, got removed. Linus invented git. "Next step" in version control. Works both on machine and has centralized repo. 

Perforce, Subversion, CVS have all been good in the past. 

Git
- developed in internet era
- works well in high latency, constrained bandwidth networks - working on road, etc.
- small footprint, on disk and in memory
- simple (causes arguments!) - to get started, but massive battery of stuff you can learn
- still things instructor is learning every month (eep!)
- composable - from linux/unix roots - built out of several small programs that work together
	- you can compose new ways of working with batch/zsh/csh/bash script

Setting up git 
- running installer
- can just copy binaries onto $path, no admin required
http://help.github.com - has probably been updated - installers
- git --version
- linux distro is core, mac and windows hours to days later
- command line is a first class citizen, not second (Eclipse, Visual Studio, etc.)

- create a git repository (unlike old systems, do it first, then place it after)

ls -al
tree .git | more

actual data in info, objects, and refs
there is one and only one .git folder per repository, at top level of project (unlike CVS, subversion)
if you want to un-git something, just delete the .git folder

git status

my text file is an untracked file. Git tracks content. See if any files are modified or have not been seen before. Git add file to version control.

Unlike other VCS systems, this just sets up to participate in next change set.

git status
git commit -m "my first commit"
master branch, first 7 characters of global unique identifier
100644 "umask - UNIX": 44 means user can read & write, others can read

git status - clean - nothing in background, examines everything
no need to refresh, redo database

free book: http://git-scm.com/book, Pro Git by Scott Chacon
http://teach.github.com

Now, we intentionally left it this long to get the net side going. Could do all this stuff locally.

Firefox and Chrome - two different repositories, githubtrainer, githubstudent. Trainer is the project leader, student is the minion/collaborator/open source contributor

githubtrainer runs hellogitworld

First, publish your own project. The local project has no knowledge of any network destination.

Click "create a new repository"
git remote add origin git@github.com:Nissl/course_3_notes.git
git push -u origin master
(type username, password)

git remote adds a "bookmark" at the location
push is the verb that syncs

git preserves timestamp on local machine

working on, joining, etc. project that already exists
we don't have privileged access to the repo - do we ask for credentials?
nope! on many open source projects, start helping, ask for code to be folded in once you get a meaningful contribution. go to repository, fork. get a photocopy to personal account.

As soon as we start making changes, will offer them back to project.

Step back out of the project. 

git clone (repository address) - clones into the folder name

Full-fledged repo - all history and branches on local disk
Emphasize - local instructions

git branch novclassfeature
git checkout novclassfeature

Now directory is on a branch
(Testing something.)

Go to website. To offer: pull request
A pull request is a fundamental unit of work on the Github side of things.
Why is this called a "pull request" - inverted permissions - changes not folded in until admin decides to. Open window, describe why to add
Queued up for core project maintainer to act on.

Several pull requests open
Merge one into project

This ability is very transformative! Don't have to worry about giving access to the project to people. Give the other people the ability to issue pull requests.

Can have a debate on changes - fires a message back to author of pull request.