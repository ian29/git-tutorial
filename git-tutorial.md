# Set Up

If you haven't already, be sure to [get set up with Git](http://help.github.com/set-up-git-redirect). 

Although all of these are optional, these steps will improve your workflow and make learning git much more manageable. 

Also checkout http://gitref.org/ for more examples and explanations. 

### Colors

Colors can let you know whether you're in a git working copy, and which branch your working on. Simply paste the following code into ~/.bash_profile using a text editor like nano. 

><pre>
>
>parse_git_branch() {
>  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(git::\1)/'
>}
>
>if [ $TERM = 'dumb' ] ; then
>    # No color, no unicode.
>    export PS1="\w \$(parse_git_branch) > "
>else
>    export PS1="\[\033[01;34m\]\w \[\033[32m\]\$(parse_git_branch)\[\033[00m\] ▶ "
>fi
></pre>

and whenever you're in a git working copy, your command prompt should look like this: 

![](https://img.skitch.com/20111201-gghbf2a8sp4pcwspht394dup26.png)

Colors can also show which lines have been added, deleted, ignored, and more by adding this code to ~/.gitconfig :

><pre>
[color]
	branch = auto
	diff = auto
	status = auto
[color "branch"]
	current = yellow reverse
	local = yellow
	remote = green
[color "diff"]
	meta = yellow bold
	frag = magenta bold
	old = red bold
	new = green bold
[color "status"]
	added = yellow
	changed = green
	untracked = cyan
</pre>

### Default Text Editor

Setting up a default text editor is helpful, especially when making commit comments that can't be capture with the -m option. To do so, simply copy the following code into ~/.bash_profile. 

><pre>export EDITOR=nano</pre>

In this example, I'm setting my default text editor to nano, but most text editors will work. 

### Merge Tools

In the event git can't resolve merge conflicts, it's a good idea to have a default merge tool like [DiffMerge](http://www.sourcegear.com/diffmerge/downloads.php) on hand. For Mac OS X, be sure to download the installer, and __not__ the DMG.

![diffmerege](https://img.skitch.com/20111201-85muwrcxdp7cte8cc6s476f4ak.png)

To set this as your default, run 

> nano ~/.gitconfig

and past in the following code:

><pre>[mergetool "diffmerge"]
     cmd = diffmerge --merge --result=$MERGED $LOCAL $BASE $REMOTE
     trustExitCode = false
</pre>

then run:

<pre>
git config --global merge.tool diffmerge
</pre>

and you should be good to go!

# Tracking a TileMill project with git

### Creating a repo

To create a (new, public) repository, go to you're github dashboard (simply github.com, once you're logged in to your account) and click on the "New Repository" button

![](https://skitch.com/sbysp/gc74q/your-dashboard-github)

Next, you'll need to run <code>git init</code> in Terminal in the directory you want to be add to your repo. In this case, this will be the relevant TileMill project folder in <code>/Users/[you]/Documents/project</code> This __init__ializes your git repo, creating a .git file in your current directory where all of your changes will be tracked. 

Now run <code>git add your-files.ext</code> for any files or directories you want tracked by git. If you run <code>git status</code>, now you should see a list of the files you just added. Running <code>git status</code> is a good way to make sure only the files you have staged will be committed. If you added have added color highlighting to your .gitconfig file, the should be yellow:

![git status](https://img.skitch.com/20111201-cpbmy5qhmeaxj2ekn3asns9p86.png)

You're ready for your first commit. Running <code>git commit</code>, should open your default text editor as a way of prompting you for a comment to describe the commit you are making. By convention, the first line of your first commit is appropriately "First Commit" followed by a description of the changes you are making  (in this case you are adding files to your repo). After you've outlined your changes, save and close the editor, and your commit will be complete. 

For subsequent commits, you can use the <code>-m</code> flag to make shorter comments:

><pre>git commit -m "a shorter comment"</pre>

Although you've committed your changes, they still only exist on your local branch of the repo that lives on your computer. This local repo can contain a list of any remote versions of the repo. To add a remote location to this list (i.e. one that others can access), run <code>git remote add origin git@github.com:_your-username_/_your-repo.git_</code>. You can copy the url from the set up page you saw after creating the repo at GitHub:
![](https://img.skitch.com/20111201-ci4ewepq7uhhyghde71gbwrpqt.png)

Now you can run <code>git push -u origin master</code>. Note that on your first push, you need to include the <code>-u</code>, ????adding an upstream reference so that all subsequent pulls will reference this remote version????

### The patch option

Now that git is tracking your TileMill project, making frequent commits will make it easier to track the evolution of the project. In order to track smaller changes, git allows you to selectively add "hunks" of altered code while ignoring others, using the <code>-p</code> flag:

![](https://img.skitch.com/20111201-mqkmqasnfuksautw743ff9tmxd.png)

In this case, the  minus sign and red color indicate that I've deleted the comment /*light blue*/ and a green plus sign indication I've added a line of white space. If I wanted to add this, I could type "y" at the prompt. Since white space is pointless and I like that comment, I can also type "n" to decline adding this hunk. Doing this returns a the same prompt for the next hunk of code. Here, I've added a comment labeling the map's background, which I would like to add:

![](https://img.skitch.com/20111201-jwxr4nafuiki4xx6ehagkmb8xe.png)

Entering yes adds the file as there are no more 'hunks', and you can commit your change	s. 

## File management

### `git mv` and `git rm`

Git requires you to use <code>git mv</code> and <code>git rm</code> to make changes in the repo's filesystem. Using the standard unix counterparts <code>mv</code> and <code>rm</code> will preserve changes in your local working tree, but not in the git repo. For example, if you were to accidentally delete a file using <code>rm</code>, you could simply run <code>git checkout _your-deleted-file_</code> in the appropriate directory, and the file will be reinstated. Without running <code>git checkout</code>, subsequent pulls will not reinstate the file to your local working branch.

### Removing symlinks

Unless your data is saved directly to the <code>project/layers/</code> directory, this folder only contains symbolic links to the location of the data. Adding the <code>project/layers/</code> and symlinks that live there to a git repo will fail to add the data, making it inaccessible to any collaborators attempting to work on the same TileMill project. To remove these symlinks run the following two commands

><pre>mv layers layers.orig/ </pre>

><pre>rsync -aLv layers.orig/ layers/</pre>

This will create a new directory called <code>layers.orig</code>, containing the original smlinks, and a new folder called <code>layers/</code> containing copies of the original files. You can now add the entire project to your git repo, allowing collaborators access to your data layers. 

## Other git tools

### Branches

Git allows you create new branches that will still track your changes but not push them to the master branch. This allows you to experiment without jeopardizing the main (and hopefully stable) branch of the project. For example, if you are making serious alterations to an already functional data layer in a project that was already being used to render polished maps and didn't want to corrupt the original data files, branching your project would allow you to play with the data without worrying about whether your pushes would affect the files being used in the master branch. 

To create a branch, run <code>git branch _you-branch-name_</code>. To simultaneously create and switch to a branch <code>git branch -b _you-branch-name_</code> (without invoking the -b option, you will create the new branch, but still be working in the original branch). To switch between branches run <code>git checkout _your-branch-name_</code>. Remember that any pushes you make will now require you to run <code>git push origin _your-branch-name_</code>. Once your now-isolated changes are complete, you will probably want to merge your branch back to the master. To do so run <code>git merge _your-branch-name_</code>. 

### Conflicts 

If git can't resolve and conflicts, it's a perfect opportunity to try out DiffMerge or whatever mergetool you're using. You should see an error like this: 

![merge conflict](https://img.skitch.com/20111202-gjq6s4ty2du3cs7c93ejeuafg7.png)

to resolve, run <code>git mergetool</code> or  if you have not set up a default merge tool
<code>git mergetool -t _your-merge-tool_</code>. You should now see this prompt:

![git mergetool prompt](https://img.skitch.com/20111202-1jisqcy57b1gaha4suf3gbhuwb.png)

Enter yes, pick which changes you want from each file (in diffmerge, the version in your local working copy will be on the left and the conflicting version will be on the right), and save your changes. Once complete, push your changes and run <code>git merge</code> again. No error should appear. You will notice however that in your present working directory you will have a new file for every file in which you resolved a merge conflict. The file as it existed before the merge will now be labeled <code>_your-file.orig_</code> and the new file (inclusive of the merge resolution) will maintain the same naming convention.   

### .gitignore

The .gitignore file that lives in your home directory is a useful way of ensuring that git will not track certain file types. If you were editing a large GeoTiff creating changes that would result in unnecessarily large pulls for your collaborators, you could simple edit .gitignore by running <code>nano ~/.gitignore</code> and adding .tif to the file contents. This would ensure that all files with the .tif extension would not be tracked by git. 

### Git Tags

If you want or need to remember important commits, you can use <code>git tag</code>. As you will probably want to include a message relating the significance of your tag, run <code>git tag -a _your-tag-message_</code>. Tags are useful in that they allow you to reference important moments in your project's development. Using <code>git diff _your-tag-mesasge_</code> will display all changes to your current working tree since you made that tag:

![](https://img.skitch.com/20111202-di9jm4d68cw2gbqqc5fjkguj5p.png)
 
