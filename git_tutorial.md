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
