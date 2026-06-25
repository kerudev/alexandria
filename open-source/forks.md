# open-source/forks

I've been trying to contribute to open source projects lately.

I like to learn by doing rather than reading, but after creating and developing
some repos of my own, I think it's time to take the next step and start
contributing to others' codebases.

For me, this accelerates the learning process as you are reading at the same
time you are writing.

It might be intimidating at first, so that's why you've got to start with
projects that you find interesting or that might need a helping hand.

So, how can you start contributing to other's work? It all starts with a fork.

Forking is a GitHub action that clones a repository into your profile. You then
clone the fork on you computer:

```sh
git clone https://github.com/your_user/repo_name.git
```

Which will also link that URL as the project's `origin` branch, but we also
need to setup an `upstream` branch in order to pull and push our changes to the
original repo: 

```sh
git remote add upstream https://github.com/original_user/repo_name.git
```

If you cloned the repo before doing the fork, the `origin` branch will be
pointing to the original repo instead of the fork. To fix that:

```sh
git remote rename origin upstream
git remote add upstream https://github.com/original_user/repo_name.git
```
