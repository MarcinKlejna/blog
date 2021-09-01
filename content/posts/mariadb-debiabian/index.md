---
title: "Instalacja i konfiguracja bazy MariaDB w Debiabie"
date: 2020-02-25T00:39:15+01:00
draft: false
tags: [mariadb, debian]
categories: ["baza danych", "serwery"]
images: [
	"featured-image.webp"
]
---
![](featured-image.webp)

## Instalacja

Wydajemy polecenie:

```bash
# aptitude install mariadb-server`
```
Spowoduje to zainstalowanie bazy MariaDB, w tym momencie nie musimy ustawiać hasła.

Uruchamiamy dołączony z bazą skrypt bezpieczeństwa, możemy za jego pomocą ustawić hasło użytkownika **root** bazy.

```bash
# mysql_secure_installation
```

To są opcje bezpieczeństwa instalacji MariaDB. Na początku prosi o wprowadzenie hasła **root** bazy danych, ponieważ jeszcze go nie ustawiliśmy, wciskamy `ENTER` aby wskazać „brak”

W następnym pytaniu skrypt pyta czy chcemy ustawić hasło **root** bazy danych, wpisujemy `N` i klikamy `ENTER`. W Debianie konto **root** dla MariaDB jest ściśle powiązane z automatyczną konserwacją systemu.  Umożliwiłoby to dostęp do systemu poprzez dostęp do konta  administracyjnego. Później omówimy sprawę uwierzytelnienia.

W następnym pytaniu wciskamy `Y` aby usunąć anonimowych użytkowników i testowe bazy danych i zdalne logowanie jako **root**.

## Uprawnienia użytkownika (opcjonalnie)

W systemach Debian z MariaDB 10.1 użytkownik **root** MariaDB jest ustawiony domyślnie na uwierzytelnianie przy użyciu wtyczki `unix_socket` , a nie za pomocą hasła. Pozwala to na większe bezpieczeństwo i  użyteczność w wielu przypadkach, ale może również skomplikować sytuację, gdy trzeba zezwolić na uprawnienia administracyjne do zewnętrznego  programu (np. [phpMyAdmin](https://www.phpmyadmin.net/)).

Ponieważ serwer używa konta **root** do zadań takich jak  rotacja dziennika, uruchamianie i zatrzymywanie serwera, najlepiej nie  zmieniać szczegółów uwierzytelniania konta **root** . Zmiana poświadczeń konta w `/etc/mysql/debian.cnf` może początkowo działać, ale aktualizacje pakietów mogą potencjalnie zastąpić te zmiany. Zamiast modyfikować konto **root**, opiekunowie pakietów zalecają utworzenie oddzielnego konta  administracyjnego, jeśli trzeba skonfigurować dostęp oparty na hasłach.

W tym celu stworzymy nowe konto o nazwie **admin** z tymi samymi możliwościami, co konto **root** , ale skonfigurowane do uwierzytelniania hasłem. Aby to zrobić, otwórz znak zachęty MariaDB na swoim terminalu:

```bash
$ sudo mysql
```

Teraz możemy utworzyć nowego użytkownika z uprawnieniami **root** i dostępem opartym na hasłach. Zmień nazwę użytkownika i hasło, aby dopasować je do preferencji:

```bash
MariaDB [(none)]> GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

Opróżnij uprawnienia, aby upewnić się, że są one zapisane i dostępne w bieżącej sesji:

```bash
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Na koniec przetestujmy instalację MariaDB.

## Testowanie MariaDB

Po zainstalowaniu z domyślnych repozytoriów MariaDB powinien uruchomić się automatycznie. Aby to sprawdzić, sprawdź jego status.

```bash
$ sudo systemctl status mariadb
```

Zobaczysz dane wyjściowe podobne do następujących

```bash
mariadb.service - MariaDB database server
 Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
 Active: active (running) since Tue 2018-09-04 16:22:47 UTC; 2h 35min ago
Process: 15596 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSIT
Process: 15594 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
Process: 15478 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||
Process: 15474 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITI
Process: 15471 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysql
Main PID: 15567 (mysqld)
 Status: "Taking your SQL requests now..."
  Tasks: 27 (limit: 4915)
 CGroup: /system.slice/mariadb.service
         └─15567 /usr/sbin/mysqld

Sep 04 16:22:45 deb-mysql1 systemd[1]: Starting MariaDB database server...
Sep 04 16:22:46 deb-mysql1 mysqld[15567]: 2018-09-04 16:22:46 140183374869056 [Note] /usr/sbin/mysqld (mysqld 10.1.26-MariaDB-0+deb9u1)   starting as process 15567 ...
Sep 04 16:22:47 deb-mysql1 systemd[1]: Started MariaDB database server.
```

Jeśli MariaDB nie działa, możesz go uruchomić za pomocą `sudo systemctl start mariadb` .

Aby uzyskać dodatkową kontrolę, możesz spróbować połączyć się z bazą danych za pomocą narzędzia `mysqladmin` , które jest klientem, który pozwala uruchamiać polecenia  administracyjne. Na przykład ta komenda mówi o połączeniu się z MariaDB  jako **root** i zwróceniu wersji przy użyciu gniazda Unix:

```bash
$ sudo mysqladmin version
```

Powinieneś zobaczyć wynik podobny do tego:

```
mysqladmin  Ver 9.1 Distrib 10.1.26-MariaDB, for debian-linux-gnu on x86_64
Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Server version      10.1.26-MariaDB-0+deb9u1
Protocol version    10
Connection      Localhost via UNIX socket
UNIX socket     /var/run/mysqld/mysqld.sock
Uptime:         2 hours 44 min 46 sec

Threads: 1  Questions: 36  Slow queries: 0  Opens: 21  Flush tables: 1  Open tables: 15  Queries per second avg: 0.003''
```

Jeśli skonfigurowałeś oddzielnego użytkownika administracyjnego z  uwierzytelnianiem za pomocą hasła, możesz wykonać tę samą operację,  wpisując:

```bash
$ mysqladmin -u admin -p version
```

Oznacza to, że MariaDB działa i działa poprawnie, a użytkownik może pomyślnie się uwierzytelnić.

Źródło: https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-debian-9
