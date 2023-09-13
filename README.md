# mywebsite

Hello world
some words to describe project

[Terminal]
#Add and Commit
git add .
git commit -m "message here"
git pull
git push

#Add and Commit
1. Add (or stage) a file or group of files.
$ git add NAME-OF-FILE-OR-FOLDER
Add all files.
1 $ git add -A
Add updated files only (modified or
deleted, but not new).
1 $ git add -u
Add new files only (not updated).
1 $ git add .

2. Commit your changes. (with -m message)
1 $ git commit -m "Helpful message"

#Pull and Push
3. Pull from the upstream repository
(i.e. GitHub).
1 $ git pull

4. Push any local changes that you’ve
committed to the upstream repo
1 $ git push

(i.e. GitHub).
#Other Commands
See commit history (hit spacebar to scroll
down or q to exit).
1 $ git log

See changes
1 $ git status




#Branch terminal commands
Create a new branch on your local machine:
1 $ git checkout -b NAME-OF-YOUR-BRANCH

Push the new branch to GitHub:
1 $ git push origin NAME-OF-YOUR-BRANCH

List all branches on your local machine:
1 $ git branch

Switch back to (e.g.) the main branch:
1 $ git checkout main

Delete a branch
1 $ git branch -d NAME-OF-YOUR-BRANCH
2 $ git push origin :NAME-OF-YOUR-BRANCH

[Line endings and different operating systems
During collaboration with a colleague on a project, you may fi nd cases where Git is highlighting differences on seemingly unchanged
sentences.
If that is the case, check whether your partner is using a different operating system to you. The “culprit” is the fact that Git evaluates an
invisible character at the end of every line. This is how Git tracks changes.
For Linux and MacOS, that ending is “LF” For Windows, that ending is “CRLF”
To solve this OS incompatibility, open up the shell/terminal and enter
macOS git config --global core.autocrlf input
Windows git config --global core.autocrlf true]

(latest token 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)
