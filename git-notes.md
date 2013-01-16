Git för sysadmins
==================================
***
* På RedBridge används git för versionshantering. Git är ett **DVCS**, ett **D**istributed **V**ersion **C**ontrol **S**ystem. Detta innebär att alla repon i princip är likvärdiga. Ett lokalt repo  är innehåller samma information som ett server repo (jmfr med *SVN*).

* För att hantera server repon (eller snarare att lagra repon för gemensam åtkomst) används **gitolite**. Gitolite hanteras med hjälp av ett git repo.

         $ git clone rb-vcs.redbridge.se:gitolite-admin

Baserna i Git
------------------------------

### .gitconfig

* defaults, alias och colors

        [user]
            name = magnus
            email = magnus@cloudmonkey.org
        [alias]
            co = checkout
            st = status
            ci = commit
            unstage = reset HEAD --
            last = log -1 HEAD
            lastdiff = whatchanged -1 -p
            gitx = !GitX
            br = branch
            logstat = log --stat --graph
            loggrep = log -g --stat --grep
            ch = checkout
            undocommit = !git reset --soft HEAD && git unstage && git pull && git reset --hard origin/master
            logpretty = log --pretty=oneline --abbrev-commit
        [core]
            editor = vim
            whitespace=fix,-indent-with-non-tab,trailing-space,cr-at-eol
            quotepath = false
        [color]
            ui = auto
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
        [instaweb]
            local = true
            httpd = webrick
        [github]
            user = cldmnky
        [push]
            default = simple

* per repo config:

        git config user.name "Your Name Here"
        git config user.email your@email.com


### Git arbetsflöde
1. clone/init

        $ git clone git@git.redbridge.se:repo
        # git init . && git add .
        # git clone git@git.redbridege.se:personal/mbengtsson/my_personal_repo

2. commit

        $ git commit -a -m "My commit message"

3. push

        $ git push origin remote

#### clone

* lägger till en remote:


        $ git clone git.redbridge.se:rb-puppet
        ...
        $ git remote -v
        origingit@git.redbridge.se:rb-puppet (fetch)
        origingit@git.redbridge.se:rb-puppet (push)


#### commit

* Filer är "untracked", "unstaged", "staged" eller "comitted"

* comittar till lokalt repo. (-a lägger till alla filer)

* Ett arbetsflöde. Lägg till en fil
        
        $ git status
        # On branch master
        nothing to commit, working directory clean
        $ ls
        README.md classify  lib       manifests
        $ touch new_file
        $ git st
        # On branch master
        # Untracked files:
        #   (use "git add <file>..." to include in what will be committed)
        #
        #   new_file
        nothing added to commit but untracked files present (use "git add" to track)
        $ git add new_file
        $ git st
        # On branch master
        # Changes to be committed:
        #   (use "git reset HEAD <file>..." to unstage)
        #
        #   new file:   new_file
        #
        $ git commit -m "Added new_file"
        [master 5688b91] Added new_file
         1 file changed, 0 insertions(+), 0 deletions(-)
         create mode 100644 new_file

* Ändra commit meddelandet:

        $ git commit --amend -m "Added the new_file"
        [master 69db67c] Added the new_file
         1 file changed, 0 insertions(+), 0 deletions(-)
         create mode 100644 new_file

* Ändra en fil

        $ echo "change file content" > new_file
        $ git st
        # On branch master
        # Your branch is ahead of 'origin/master' by 1 commit.
        #
        # Changes not staged for commit:
        #   (use "git add <file>..." to update what will be committed)
        #   (use "git checkout -- <file>..." to discard changes in working directory)
        #
        #   modified:   new_file
        #
        no changes added to commit (use "git add" and/or "git commit -a")
        $ git commit -a -m "Changed new_file"
        [master 174ddc6] Changed new_file
         1 file changed, 1 insertion(+)

* Gå tillbaka en revision på filen (HEAD^ ), varje ^ motsvarar en revision. HEAD är en refspec, origin/master

        $ git co HEAD^ new_file
        $ cat new_file
        $ git rm -f new_file
        rm 'new_file'
        $ git st
        # On branch master
        # Your branch is ahead of 'origin/master' by 2 commits.
        #
        # Changes to be committed:
        #   (use "git reset HEAD <file>..." to unstage)
        #
        #   deleted:    new_file
        #
        $ git commit -m "Removed new_file"
        [master 6827da8] Removed new_file
         1 file changed, 1 deletion(-)
         delete mode 100644 new_file
        $

#### Git revisioner och annat

* HEAD är en referens till den senaste commiten i den utcheckade branchen.
    * HEAD^ är den commit innan ( HEAD^^^ tre commits innan)
    * Eller: HEAD~1, HEAD~3

* En serie av commits anges genom:

        $ git log master..local_branch
        # visar commits som skett utanför master branchen
        $ git log origin/master..HEAD
        # Visar vad som skiljer sig mellan din lokala branch och remote's master. Användbart!!
        $ git log 4717a5c..7214d6a

#### branch

* En branch skapas genom att:

        $ git checkout -b new_feature
        # do some work

* Visa branches:

        $ git branch
        # Visa remote branches
        $ git branch -r

* Push:a en lokal branch till ett remote repo:

        $ git push origin local_branch

* Ta bort en lokal branch:

        $ git branch -d local_branch

* Ta bort en remote branch:

        $ git push origin :local_branch

#### Att ångra sina ändringar

* Ångra en valfri commit genom en "revert".
    
    * Gör en "undo på commiten" sedan en ny commit med "undo:en"

            $ git revert 4717a5c

* Flytta HEAD bak två commits:

        $ git reset --hard HEAD^^

* Att leka historie revisionist:

    * Skriver om historien i git. Farlig om branch push:at's till remote. Alla som baserat arbete på en revision innnan historiken skrevs om är
    up-shit-creek.

    * Går bra att göra innan man pushar till remote. Bäst att göra i en egen lokal feature branch innnan merge -> push

            $ git rebase -i HEAD^^^^^

* Ändra senaste commit:

        $ git commit -m 'initial commit'
        $ git add forgotten_file
        $ git commit --amend


#### Konflikter

* Att lösa konflikter.

* Uppstår när en merge skall ske och förändringar skett som gör att det finns en konflikt.

* vimdiff som mergetool:

        $ git config --global diff.tool vimdiff
        $ git config --global merge.tool vimdiff

* fugitive.vim Git i vim. http://vimcasts.org/episodes/fugitive-vim-resolving-merge-conflicts-with-vimdiff/

    * Att lösa en konflikt med fugitive/vimdiff: Öppna filen i vim (ev. sedan :Gdiff på filen)
    * :diffget //2 eller :diffget //3

Git & puppet
-------------------------------


* Puppet har stöd för environments: Exekverings-miljöer. puppet agent -t --environment=my_feature_branch

    * Environments specas i puppet.ini.

* Git branching matchar miljöer perfekt.

        $ git branch
        *master
        dev
        $ git checkout -b new_feature
        # code, code, code
        $ git push origin new_feature
        # om man använder dynamiska env's i puppet kommer en environmnet skapas för denna branch
        # på en eller flera node kan sedan förändringarna testas
        # när allt passerat nålsögat:
        $ git checkout master && git merge new_feature
        $ git push && git branch -d new_feature && git push origin :new_feature

* Vi har endast 2 environments: production + dev.

    * Använd **alltid** dev branchen...! 

### Git-hooks

* git hooks kan utföra saker innan commits eller push.

    * pre-commit är användbar i det lokala repot:

            .git/hooks/pre-commit


            #!/bin/bash
            # pre-commit git hook to check the validity of a puppet manifest
            # 
            # Prerequisites: 
            #   gem install puppet-lint puppet
            # 
            # Install: 
            #  /path/to/repo/.git/hooks/pre-comit
            
            # Source RVM if needed
            [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
            
            echo "### Checking puppet syntax, for science! ###"
            # for file in `git diff --name-only --cached | grep -E '\.(pp|erb)'`
            for file in `git diff --name-only --cached | grep -E '\.(pp)'`
            do
                # Only check new/modified files
                if [[ -f $file ]]
                then
                    puppet-lint \
                        --no-80chars-check \
                        --no-autoloader_layout-check \
                        --no-nested_classes_or_defines-check \
                        --no-only_variable_string-check \
                        --no-2sp_soft_tabs-check \
                        --with-filename $file
            
                    # Set us up to bail if we receive any syntax errors
                    if [[ $? -ne 0 ]]
                    then
                        syntax_is_bad=1
                    else
                        echo "$file looks good"
                    fi
                fi
            done
            echo ""
            
            echo "### Checking if puppet manifests are valid ###"
            # validating the whole manifest takes too long. uncomment this
            # if you want to test the whole shebang.
            # for file in `find . -name "*.pp"`
            # for file in `git diff --name-only --cached | grep -E '\.(pp|erb)'`
            for file in `git diff --name-only --cached | grep -E '\.(pp)'`
            do
                if [[ -f $file ]] 
                then
                    puppet parser validate --mode user $file
                    if [[ $? -ne 0 ]]
                    then
                        echo "ERROR: puppet parser failed at: $file"
                        syntax_is_bad=1
                    else
                        echo "OK: $file looks valid"
                    fi
                fi
            done
            echo ""
            
            if [[ $syntax_is_bad -eq 1 ]]
            then
                echo "FATAL: Syntax is bad. See above errors"
                echo "Bailing"
                exit 1
            else
                echo "Everything looks good."
            fi
