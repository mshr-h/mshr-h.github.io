# blog

## clone repo

```sh
$ git clone git@github.com:mshr-h/mshr-h.github.io.git
```

## new post

```sh
$ hugo new posts/2020-04-01-title.md
```

## deploy

```sh
$ git add .
$ git commit -m "add new post"
$ git push
```

After pushing to GitHub repo, GitHub Actions automatically build and deploy to `master` branch.

## development

```sh
$ hugo server -D
```
