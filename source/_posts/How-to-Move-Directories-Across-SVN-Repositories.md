title: "How to Move Directories Across SVN Repositories"
date: 2011-11-15 15:11:45
tags:

- Misc

---

Recently we were restructuring our project repositories, where I had to move a project from it’s own repository to another repository.

When do a move under SVN you need to make sure that the history is not lost, so I will explain two common scenarios and how you could do the SVN move

### Scenario 1 - Moving files or directories with in a SVN repository

Below is a screen shot of my existing repository and I want to move the **configuration** directory to the **project** directory without loosing the history.

![SVN repository structure](/images/2011/11/20111115081425_Screen1_thumb.png)

First of of all this will work only for the versioned items, so please commit your changes to the directory you want to move before doing this. The steps explained below assumes that you are using Tortoise SVN

Here are the steps

1. Right click on the directory you want to move
2. Drag and drop on to the directory where you want to move this.
3. Choose the **SVN Mover Versioned item(s) here** item from the context menu.
   ![Context Menu with move](/images/2011/11/20111115081432_Screen2_thumb.png)
4. Now commit the changes back to the repository.

Don’t forget to uncheck _Stop on copy/rename_ option in the _Log messages_ dialog if you are trying to see the history of the moved directory.

![Move configuration](/images/2011/11/20111115081442_Screen3_thumb.png)

### Scenario 2 – Moving directories across SVN repositories

In this case I want to move **Project** directory to another repository **Repo2**

The first technique I have mentioned will work only if the source and target are under same repository, but in some cases you have to move the directories across different SVN repositories. This cannot be done from a SVN client slike TortoiseSVN, you need o use the utilities that comes with SVN.

Follow these steps

1. Dump the repository to a file using the `svnadmin` tool – e.g. `svnadmin dump c:\Repositories\Repo1 > Repo1file`
2. Now load the dumped filed into the target repository – e.g. `svnadmin load c:\Repositories\Repo2 < Repo1file`

Here is the full syntax for svnadmin tool

```bash
$ svnadmin dump --help
dump: usage: svnadmin dump REPOS_PATH [-r LOWER[:UPPER] ] [--incremental]

Dump the contents of filesystem to stdout in a 'dumpfile'
portable format, sending feedback to stderr. Dump revisions
LOWER rev through UPPER rev. If no revisions are given, dump all
revision trees. If only LOWER is given, dump that one revision tree.
If --incremental is passed, then the first revision dumped will be
a diff against the previous revision, instead of the usual fulltext.

Valid options:
-r [--revision] arg : specify revision number ARG (or X:Y range)
--incremental  : dump incrementally
--deltas   : use deltas in dump output
-q [--quiet]  : no progress (only errors) to stderr
```
