---
title: "Jak zarządzać plikami dotfiles za pomocą git"
date: 2021-01-05T09:19:05+01:00
draft: false
tags: [git]
categories: ["szybki tip", "git"]
images: [
	"featured-image.png"
]
---

![](featured-image.png)

Na podstawie [Hacker News StreakyCobra](https://news.ycombinator.com/item?id=11070797)

Jak sam napisał:

> Żadnych dodatkowych narzędzi, żadnych dowiązań symbolicznych, pliki są śledzone w systemie kontroli wersji, możesz używać różnych gałęzi dla różnych komputerów, możesz łatwo powielić konfigurację na nowej instalacji.

## Zaczynamy

* Utwórz folder `.dotfiles`, którego użyjemy do śledzenia twoich plików konfiguracyjnych
```
git init --bare $HOME/.dotfiles
```
* Tworzymy `alias` do katalogu `.dotfiles`
```
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```
* Ustawiamy status `git` aby nie informował nas o nieśledzonych plikach. W innym wypadku będziemy informowani o całej zawartości katalogu `$HOME`
```
dotfiles config --local status.showUntrackedFiles no
```

I to wszaystko.

## Dodajmy pliki do naszego nowego repozytorium.

Dla przykładu aby dodać konfigurację neovima i wysłanie na serwer:
```
dotfiles add .config/nvim/
dotfiles commit
dotfiles push
```

## Przywracanie repozytorium

Na nowym komputerze instalujemy oczywiście `git`

* Klonujemy nasze repozytorium:
```
git clone --bare git@gitlab.com:marcin-klejna/dotfiles.git
```
* Tworzymy alias
```
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```
* Pobieramy aktualną wersję repozytorium.
```
dotfiles checkout
```
