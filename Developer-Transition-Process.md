### How to Update Your Current Stan Directory

If you have no changes, just follow the directions in [Developer Process](Developer-Process) on cloning. If that doesn't work, do:

    > git pull

That will pull down all the current branches. If you were on working on branch "foo", the new branch will be at "feature/foo".

Commit all changes to foo:

    > git checkout foo
    > git commit -m "mesage" -a

Check out the new feature branch named "feature/foo"

    > git checkout feature/foo

Merge the changes in your current branch:

    > git merge foo

Delete old branch:

    > git branch -d foo

You shouldn't get warnings, because you've already merged into `feature/foo`.

Now you're back to the "do work" part of the standard developer process.