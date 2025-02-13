# git rebase vs git merge vs git pull origin main
Initial Commit history
```css
A - B - C (main)
     \
      D - E - F (feature-branch)
```
Meanwhile X - Y also get pushed to main branch by other developers.

✅ Rebase : If using rebase, it will apply your feature branch commits on top of update main branch commits
so it will be A - B - C - X - Y - D' - E' - F' where D E F are rebased commits. Also no new merge commit is created keeping
linear history and cleaner history.

```css
git checkout feature-branch
git rebase main

What happens internally:
Git temporarily removes your commits (D - E - F) from feature-branch.
It applies the latest commits from main (X - Y) to feature-branch.
It reapplies your commits (D - E - F) on top of G - H, one by one.
Why is this Useful?
  ✅ Keeps history clean → No unnecessary merge commits.
  ✅ Easier to understand → It looks like you worked on the latest code from main all along.
  ✅ Good for feature branches → Makes it easier to merge back into main.
If Conflict then manually resolve conflicts
 ✅git add <changed_file>
 ✅git rebase --continue
```

✅Merge / pull origin same: merge will pull changes in features branch from main branch preserving original commit history.
A - B - C - D - E - F - X - Y - M (merge commit)
If Conflict then manually resolve conflicts
 ✅git add <changed_file>
 ✅git merge --continue

# How do you squash multiple commits into one?
git rebase -i HEAD~3  # Interactive rebase for last 3 commits

