
#### Git daemon

To share path to git:
1. For share repo you should add `git-daemon-export-ok` file into **.git** directory:
	`cd .git && touch git-daemon-export-ok`
2. Run git daemon:
	`git daemon --reuseaddr --base-path={SOMEPATH}`


To clone repo by git daemon:
	`git://{IPADDRESS}/{REPOPATH}`
	Result path to repo will be: ***{SOMEPATH}/{REPOPATH}***

