# FAQ

### How can I add a separate repo as a submodule in a different repo?
[Source of answer](https://github.blog/open-source/git/working-with-submodules/)

Let's say that you have the following repo structure:

```
Repo-A
|
--- Folder-A
  |
  --- Not_much.md
```

You also have a different repo called _Repo-B_ with the following structure:

```
Repo-B
|
--- Folder-B
  |
  --- Still_not_much.md
```

And you want to make the submodule inside of _Folder-A_.

1. Move inside of _Folder-A_.
2. Run `git submodule add https://github.com/<user>/Repo-B "Repo-B Folder"`

This would create the submodule inside of _Folder-A_, resulting in the following structure:

```
Repo-A
|
--- Folder-A
  |
  --- Not_much.md
  |
  --- Repo-B
    |
    --- Folder-B
      |
      --- Still_not_much.md
```

### How can I update a submodule so it points to the last commit of its repository?

First, we move to the submodule's folder inside of our parent repo. Then we run `git fetch` and `git pull`. Then, we go back to the parent repo to stage and commit our changes. It's that easy.