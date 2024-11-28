# Oh My Posh

Nincs Windows command line támogatás.

```sh
choco install oh-my-posh
```

# Git

## Tematika

* Lokális használat:
    * `git init`, 
    * `git add`, `git commit`, `git status`, `git diff`
    * `git log`, `git blame`
* Távoli repository használata
    * `git config`
    * `git remote`, `git push`, `git pull`, `git fetch`, `git clone`
    * `git stash`
* Branch-ek létrehozása és váltása
    * `git branch`, `git checkout`
    * `git switch`
* Merge és konfliktuskezelés
    * `git merge`
    * konfliktusok feloldása
* Visszaállítás: 
    * `git reset`, `git checkout`, `git restore`
* Commit módosítása
    * `git commit --amend`
* Git munkafolyamatok:
    * Feature branching, Git Flow vagy trunk-based development.
* Pull request
* Tagek használata
    * `git tag`

## Részletek

git diff: a megadott commit és a munkakönyvtár közötti különbségeket is mutatja

git fetch: csak lehozza, de nem módosítja az aktuális munkakönyvtárat

(Weben módosítás, legyen nem commitolt módosítás)

```
git log origin/master
git diff origin/master
git diff <commit hash>
```

Conflict

```
git stash
git stash list
git stash pop
git stash push -m "My unfinished changes"
git stash apply
git stash drop
git stash clear
git stash show -p stash@{0}
```

### GH CLI

Windows PowerShell - adminisztrátor

```sh
choco install gh
```

```sh
gh auth login
```

```
gh browse
```

### Branch

git branch ottmarad

git checkout -b

git push -u origin <branch neve>

-u --set-upstream - beállítja az upstream branchet, ekkor nem kell megadni a további push, pull parancsoknál a branch nevét

git branch -vv kiírja az upstreamet

### git reset

* `--soft`: Ha szeretnéd visszavonni az utolsó commit-ot, de szeretnéd megtartani a módosításokat, hogy újra commit-olhasd őket.
* `mixed`: Ha vissza szeretnél állni egy korábbi commit-ra, de nem akarod, hogy a változások a staging area-ban maradjanak.
* `--hard`: Ha teljesen vissza akarsz állni egy korábbi commit-ra, és el akarod törölni az összes nem commitált változást.

Utolsó commit visszavonása:

```
git reset --soft HEAD~
```

A `HEAD~` a `HEAD` előtti commit, `HEAD~2` a kettővel előtti, stb.

### git restore

`git checkout` ide vágó funkcióinak lecserélésére

Fájl visszaállítása a commitolt állapotra:

```
git restore <file>
```

Egy commitban szereplő állapotra:

```
git restore --source <commit-hash> <file>
```

### amend

Csak ha nem volt push

ha addolok módosítást, akkor az is bekerül a commitba, amúgy csak a nevet módosítja - default szövegszerkesztővel

### PR

fetchelni kell egy branchre, de egyszerűbb a GitHub CLI használata

```
gh pr create --title "Lokál változtatás" --body "Elég jó"

gh pr list

gh pr checkout 1 

gh pr merge -b "Szuper" 1  
```

# Config

```
git config --global --list
git config --list
```

```
git config --global push.default simple
```

Szövegszerkesztő:

```
git config --global core.editor "code --wait"
```

(A `--wait` kapcsoló biztosítja, hogy a Git megvárja, amíg befejezed a szerkesztést, és bezárod a szerkesztőt.)

# Visual Studio Code UI

VS Code GitHub Pull Requests extension

# GitHub Desktop

# Git tag

* Tagek speciális referenciák, amelyeket egy adott commit azonosítójához rendelhető
* Leggyakrabban verziószámokat vagy kiadásokat (releases) jelölnek velük.
* Annotálható is: tartalmazhat üzenetet, szerzői információt és dátumot

```
git tag v1.0.0
git tag -a v1.0.0 -m "Első kiadás"
git tag <tag-név> <commit-hash>
git tag
git push origin <tag-név>
git push origin --tags
git show <tag-név>
```