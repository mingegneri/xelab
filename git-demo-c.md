## INITIAL SETUP
```bash
git --version
git config --global --list
git config --global user.name "YOUR NAME AND SURNAME"
git config --global user.email YOUR@PUBLIC.EMAIL
git config --global core.editor notepad     #FOR MAC USE nano
git config --global -e
```

ADD THOSE LINES AND SAVE .gitconfig
```
[alias]
  treegraph = log --graph --decorate --abbrev-commit --all --pretty=oneline
  conflicts = !git diff --name-only --diff-filter=U | xargs $EDITOR 
```

---

####FOR MAC 
Customize your bash to get some useful git helps:
- [git-prompt.sh](http://git-prompt.sh/)
- [git-completion.sh](https://git-scm.com/book/it/v2/Git-in-Other-Environments-Git-in-Bash)

####FOR PC 
Use **GIT BASH** installed from official [GIT Download](https://git-scm.com/download) so we can use the same unix command that follow... 

---

###RUNNING DEMO
```bash
# CREATE FIRST REPO, FILE AND SHOW STATUS (UNTRACKED)
git init PROJNAME
cd PROJNAME
touch readme.txt
git status

# STAGE FILE
git add readme.txt
git status

# UPDATE FILE, SHOW STATUS CHANGE AND FIRST COMMIT 
echo 'Sample repo for XE GIT LAB 22 Apr 2016' >> readme.txt
git status
git add readme.txt
git commit -m "initial commit"
git status
git log

# ADD SOME OTHER STUFF AND SHOW ANOTHER WAY TO ADD FILES IN STAGING 
echo mainsite >> index.html
mkdir pages
echo home >> pages/rename.html
echo delete >> pages/delete.html

# *.html DON'T WORK FOR SUB DIRECTORY
git add *.html
git status
# TO USE REGEX ENCLOSE IN '' - THIS ADD ALL FILE *.html IN EVERY PATH
git add '*.html'
git status
```


###WORKING WITH STAGE
```bash
# COMMON ERRORS: RENAME AND DELETE FILE FROM DISK (OR EDITOR)
mv pages/rename.html home.html
rm pages/delete.html
git status    #SHOW UNTRACKED DELETE, AND OLD FILE ARE STILL IN STAGE!

# ROLLBACK 
git checkout -- pages/delete.html
mv home.html pages/rename.html
ls pages/
git status
# AND DO IT RIGHT
git mv pages/rename.html pages/home.html
git rm pages/delete.html -f	    #(OR --cached TO KEEP FILE UNTRACKED)


# USE STAGE TO UNDO LAST CHANGES
echo asgdhjasdgj >> pages/home.html   #(OR MODIFY IT IN EDITOR)
git checkout -- pages/home.html


# DO SOME MORE CHANGE + FILE AND STAGE ENTIRE DIR CONTENT + COMMIT
echo ok >> pages/home.html
echo js >> pages/script.js
git add pages/
git status
git commit -m "added pages dir"


# USE GITIGNORE TO EXLUDE FILES
touch styles.css
touch test.log
touch pages/list.html
git status
echo '*.log' >> .gitignore
git status

# ADD EVERYTHING NOT .gitignored AND COMMIT
git add .
git status
git commit -m "added all files not .gitignored"
echo 'this is not tracked' >> test.log
git status
git log
```


###WORKING WITH BRANCH
```bash
# CREATE BRANCH 
git branch myfixcss
git checkout myfixcss
# OR ALL IN ONELINE COMMAND   git checkout -b myfixcss

# DO CHANGE INTO BRANCH (myfixcss) AND COMMIT 
echo .main >> styles.css
echo link styles >> index.html
git commit -am "main site styles"

# ADD FILE TO BRANCH (myfixcss) AND COMMIT 
touch pages/home.css
git add .
git commit -m "empty home style"

# SWITCH BRANCH TO master (THERE IS NOT CSS)
git checkout master
git status
ls pages/*.css
cat styles.css

# SWITCH BACK TO myfixcss AND FILES COMEBACK (MAGIC)
git checkout myfixcss
cat styles.css
ls pages/*.css


# MERGE BRANCH myfixcss TO master (FASTFORWARD) 
git checkout master
git merge myfixcss
ls pages/*.css
cat styles.css
```


###MERGE CONFLICT
```bash
# COMMIT A CHANGE IN myfixcss ON A SINGLE FILE home.css
git checkout myfixcss
echo .infix >> pages/home.css
git commit -am "mod in fix"

# COMMIT A CHANGE IN master ON SAME FILE-ROW (THIS WILL BE A CONFLICT)
git checkout master
echo from master >> readme.txt
echo .home > pages/home.css
git commit -am "mod in master"

# TRY TO MERGE master CHANGE IN myfixcss BRANCH (CONFLICT)
git checkout myfixcss
git merge master
# CONFLICT
git status
git conflicts     # alias.conflicts = !git diff --name-only --diff-filter=U | xargs $EDITOR 
git mergetool

# RESOLVE CONFLICT
git add pages/home.css
echo ok merged >> readme.txt
git commit -am "resolve merge conflict"
git status
git clean -f


# SYNC FAST-FORWARD AND DELETE BRANCH
git checkout master
git merge myfixcss
git branch -d myfixcss
git treegraph   # alias.treegraph = log --graph --decorate --abbrev-commit --all --pretty=oneline
```


## WORKING WITH GITHUB + SOURCETREE
```bash
# // **ON GITHUB: CREATE EMPTY xelab REPO
# // **ON GITHUB: ADD DAVIDE AS COLLABORATOR
git checkout master
cp ../slides.pptx
cp ../git-demo-c.md
git add .
git commit -m "add demo commands and slides"

# ADDING REMOTE SOURCE
git remote add origin https://github.com/dmorosinotto/xelab.git
git remote 
git remote -v

# BRING REMOTE UPDATE TO LOCAL MASTER AND MARGE IT
git checkout master
git fetch origin
git merge origin/master   
# IT'S === TO THIS SIMPLE ONE-LINE COMMAND
git pull origin

# SEND-PUBLISH LOCAL CHANGE TO REMOTE
echo ready to publish >> readme.txt
git commit -am "ready to publish"
git push -u origin master
```

### CENTRALIZED WORKFLOW (ONLY FOR INTERAL USE / SMALL TEAM)
```bash
# //**PC DAVIDE
# CLONE THE MAIN REPO TO LOCAL DISK
git clone https://github.com/dmorosinotto/xelab.git
# AND ADD SOMETHING
mkdir shared
echo "Chi e' il piu' bel socio di XE?" >> shared/domanda.txt
git add .
git commit -m "added question"
# PUSH TO MASTER (WITHOUT CONFLICT)
git push origin master


# //** PC DANIELE
# DO SOME STUFF ON OTHER FILES (THIS IS NOT A CONFLICT)
git checkout master
echo 'start collaborating... this is not a conflict' >> readme.txt
git commit -am "local commit, waiting for someone..."
# BRING REMOTE UPDATE TO LOCAL AND REBASE MASTER LOCAL CHANGE ON TOP OF REMOTE UPDATE
git fetch origin
git rebase origin/master
# IT'S === TO THIS SIMPLE ONLINE COMMAND (REBASE REWRITE HISTORY TO BE MUCH MORE LINEAR)
git pull --rebase origin
# SUCCESSFULLY INTEGRATE UPDATE
```

#### HANDLE CONFLICT FETCH / PULL / PUSH
```bash
# //**PC DANIELE
# RESPONSE AND PUSH WITHOUT CONFLICT
git checkout master
echo Daniele >> shared/domanda.txt
git commit -am "Answer question"
git push 



# //**PC DAVIDE: (MEANWHILE AT THE SAME TIME!)
# TRY RESPONSE AND PUSH WITH MERGE CONFLICT --> RESOLVE AND PUSH FINAL
git checkout master
echo Davide >> shared/domanda.txt
git commit -am "My response"
git push 

# MERGE CONFLICT BECOUSE REPO SERVER STATE IS CHANGED
git fetch
git merge
# RESOLVE MERGE CONFLICT ON EDITOR shared/domanda.txt --> Davide! - Ma Daniele e' quello Bravo :-)
git add shared/domanda.txt
git commit "Final response"
# MERGE RESOLVED AND PUSH OK
git push origin master

  
  
# //**PC DANIELE
# SYNC BACK FINAL RESPONSE
git pull
```

### PREPARATION FOR FEATURE BRANCH WORKFLOW + GITHUB PR
```bash
# //**PC DANIELE
git checkout -b partecipanti
mkdir list
cp ../help.md list/help.md
git add .
git commit -am "add PR istructions"


# SEND BRANCH TO REMOTE
git checkout partecipanti
git push -u origin partecipanti
```

#NOW IS YOUR TURN!
##FOLLOW [help.md](https://github.com/dmorosinotto/xelab/blob/partecipanti/list/help.md) INSTRUCTIONS, AND TRY TO CREATE PR!
