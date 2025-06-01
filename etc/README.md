```
커밋 후...

git checkout -B develop origin/develop
git fetch origin --prune
git merge feat/comp
git merge origin/develop
git push origin develop
````