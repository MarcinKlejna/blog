---
title: "Problem z aplikacjami AppImage w Debianie"
date: 2020-02-25T02:39:27+01:00
lastmod:
description: "Szybki sposób na uruchomienie AppImage"
draft: false

tags: [debian, appimage,]
categories: ["szybki tip"]

images: [
	"featured-image.webp"
]
---

Nie moglem uruchomić na swoim systemie aplikacji AppImage.
Dostawałem błąd związany z uprawnieniami:

```
The setuid sandbox is not running as root. Common causes:
  * An unprivileged process using ptrace on it, like a debugger.
  * A parent process set prctl(PR_SET_NO_NEW_PRIVS, ...)
Failed to move to new namespace: PID namespaces supported, Network namespace supported, but failed: errno = Operation not permitted
Pułapka debuggera/breakpoint
```

Googlując w poszukiwaniu rozwiązania natrafiłem na solucje proponującą dodanie prefiksu `--no-sandbox`.
Jest to niewskazane ze względów bezpieczeństwa. Lepszym(?) sposobem jest stworzenie pliku `/etc/sysctl.d/00-local-userns.conf` i dodanie `kernel.unprivileged_userns_clone=1`

```
# echo 'kernel.unprivileged_userns_clone=1' > /etc/sysctl.d/00-local-userns.conf
```
Po restarcie powinno wszystko działać.
