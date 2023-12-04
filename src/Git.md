# Git

Owner: Mojtaba Safaei
Verification: Verified

- Branch
    
    
    - Concepts
        
        # Git Branch
        
        Branching is a feature available in most modern version control systems. Branching in other VCS's can be an expensive operation in both time and disk space. In Git, branches are a part of your everyday development process. Git branches are effectively a pointer to a snapshot of your changes. When you want to add a new feature or fix a bug—no matter how big or how small—you spawn a new branch to encapsulate your changes. This makes it harder for unstable code to get merged into the main code base, and it gives you the chance to clean up your future's history before merging it into the main branch.
        
        ![https://wac-cdn.atlassian.com/dam/jcr:746be214-eb99-462c-9319-04a4d2eeebfa/01.svg?cdnVersion=774](https://wac-cdn.atlassian.com/dam/jcr:746be214-eb99-462c-9319-04a4d2eeebfa/01.svg?cdnVersion=774)
        
        The diagram above visualizes a repository with two isolated lines of development, one for a little feature, and one for a longer-running feature. By developing them in branches, it’s not only possible to work on both of them in parallel, but it also keeps the main `master` branch free from questionable code.
        
        The implementation behind Git branches is much more lightweight than other version control system models. Instead of copying files from directory to directory, Git stores a branch as a reference to a commit. In this sense, a branch represents the tip of a series of commits—it's not a container for commits. The history for a branch is extrapolated through the commit relationships.
        
        As you read, remember that Git branches aren't like SVN branches. Whereas SVN branches are only used to capture the occasional large-scale development effort, Git branches are an integral part of your everyday workflow. The following content will expand on the internal Git branching architecture.
        
        # How it works
        
        A branch represents an independent line of development. Branches serve as an abstraction for the edit/stage/commit process. You can think of them as a way to request a brand new working directory, staging area, and project history. New commits are recorded in the history for the current branch, which results in a fork in the history of the project.
        
        در حالت عادی تغییرات در پروژه به صورت تک خطی و معمولا توسط یک نفر ایجاد و اعمال می شود اما با استفاده از
        
         branch
        
         در واقع یک 
        
         instance 
        
         می گیریم و تغییرات دلخواه را روی آن انجام می دهیم. این تغییرات حتی اگر شامل پاک کردن یا اضافه کردن فایل های جدید هم باشد روی برنچ اصلی تاثیری ندارد تا زمانی که این تغییرات را
        
         merge 
        
         کنیم.
        
    - Commands
        
        
        ```bash
        git branch
        
        ll  .git/refs/heads/
        ```
        
        List all of the branches in your repository. This is synonymous with `git branch --list.`
        
        ```bash
        git branch <branch>
        ```
        
        Create a new branch called `<branch>`. This does *not* check out the new branch.
        
        ```bash
        git branch -d <branch>
        ```
        
        Delete the specified branch. This is a “safe” operation in that Git prevents you from deleting the branch if it has unmerged changes.
        
        ```bash
        git branch -D <branch>
        ```
        
        Force delete the specified branch, even if it has unmerged changes. This is the command to use if you want to permanently throw away all of the commits associated with a particular line of development.
        
        ```bash
        git branch -m <branch>
        ```
        
        Rename the current branch to `<branch>`.
        
        ```bash
        git branch -a
        ```
        
        List all remote branches.
        
        ```bash
        git branch -r
        ```
        
        List all remote repo branches.
        
- Commands
    
    
    **To start working with git:**
    
    # git init
    
    *This command will make a git repository in current directory and save data in a .git hidden directory*
    
    **To Remove git repo from current dir:**
    
    #rm -r -d  .git
    
    **View git current directory content:**
    
    #ls  -Lthra
    
    **View overall status:**
    
    #git status
    
    this command will show all untracked and staged files
    
    **To add a changed file mode to staging mode:**
    
    #git add filename
    
    will add all of  files:
    
    #git add    -A      
    
    **To commit staged files:**
    
    #git commit -m "comment"   filename
    
    **To check changed file's changes:**
    
    #git diff  
    
    **To exit a file from staged mode to untracked mode:**
    
    #git reset   filename
    
    **To revert a changed file's content to the last committed content:**
    
    #git checkout   —filename
    
    **Check a specific commit:**
    
    git show  commit-id
    
- Tag
    
    
- Remote Repo
    
    
    - Clone a remote project and edit
        
        
        **To pull or clone a project from a remote repo:**
        
        #git clone [https://github.com/MojtabaSafa/git.git](https://github.com/MojtabaSafa/git.git)
        
        #cd git
        
        **To check current remote repo:**
        
        #git remote -v
        
        خروجی دستور بالا :
        
        origin	[https://github.com/MojtabaSafa/git.git](https://github.com/MojtabaSafa/git.git) (fetch)
        origin	[https://github.com/MojtabaSafa/git.git](https://github.com/MojtabaSafa/git.git) (push)
        
        **MAKE YOUR CHANGES:**
        
        #git add  -A
        
        #git commit -m "comment"    filename
        
        **To Push your changes to remote (github):**
        
        #git push -u origin master
        
        **To Pull new changes that other people have made on the project:**
        
        #git pull origin master
        
        کلمه اوریجین درواقع به همان منبع اصلی پروژه که در اینجا روی گیت هاب است اشاره می کند. مستر هم که برنچ اصلی خودمان است.
        
    - Add remote manually
        
        
        #git remote add origin [https://github.com/MojtabaSafa/git.git](https://github.com/MojtabaSafa/git.git)
        
        #git remote -v
        
    - more
        
        Inspect a remote repo:
        
        ```bash
        git remote show repo1
        ```
        
        Renaming and Removing Remotes:
        
        ```bash
        git remote rename <old-name> <new-name>
        
        git remote rm repo-name
        ```
        
        List remote repository branches:
        
        ```bash
        git  branch  -r
        ```
        
    - Pull/Fetch/Push
        
        **Pull vs Fetch:**
        
        When downloading content from a remote repo, git pull and git fetch commands are available to accomplish the task. You can consider git fetch the 'safe' version of the two commands. It will download the remote content but not update your local repo's working state, leaving your current work intact. git pull is the more aggressive alternative, it will download the remote content for the active local branch and immediately execute git merge to create a merge commit for the new remote content. If you have pending changes in progress this will cause conflicts and kickoff the merge conflict resolution flow.