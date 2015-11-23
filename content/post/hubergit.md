+++
date = "2015-11-23T18:06:30+01:00"
draft = true
title = "Hubergit"
categories = [ "Uberspace", "Hugo", "Git" ]

+++

## ...hä?

Hubergit - das ist eine Mischung aus drei Wörtern:

* [Hugo][]: Ein statischer Website-Generator
* [Uberspace][]: Der beste Hoster der Welt (wenn ihr mir nicht glaubt, [glaubt anderen Leuten][1] oder probiert es selbst)
* [Git][]: Eine Software zur Versionsverwaltung

Wie man es schafft, dass diese drei zusammenarbeiten (und warum man das wollen sollte), beschreibt dieser Artikel.

<!--more-->

Dieser Blog ist eine (größtenteils) statische Webseite. Soll heißen, hier steht kein klassisches Content-Management-System (CMS) wie etwa [Typo3][] oder [Joomla][] dahinter, welches jede Seite beim Aufruf generiert und dann an den Browser ausliefert, sondern der Blog besteht ganz "oldschool" aus statischen HTML-Seiten, die halt einfach auf dem Server rumliegen.

Dieses Vorgehen bietet mehrere Vorteile, Stabilitaet und Geschwindigkeit sind zwei von ihnen. Klar, die Seiten muessen nicht erst durch einen PHP-Interpreter (oder sonst irgendwas) gejagt werden und es sind keine Datenbankzugriffe noetig, schon alleine dadurch werden viele potentielle Fehlerquellen ausgeschaltet. Generell sollte man stets versuchen, seine Webseite so statisch wie moeglich zu gestalten und nur dort auf dynamische Features zurueckzugreifen, wo es nicht anders geht.

Nun *haette* dieses Vorgehen aber auch gravierende Nachteile (ich schreibe bewusst "haette" wegen dem, was noch kommt): Webdesign wie in den 90ern. Jede einzelne Seite muss von Hand angelegt werden, jede einzelne Verlinkung von Hand eingetragen. Abhilfe schafft hier [Hugo][], ein statischer Website-Generator. Ich moechte hier nicht zu sehr ins Detail gehen, nur so viel: Hugo uebernimmt die ganzen unangenehmen, repetetiven und langweiligen Schritte fuer mich, so dass ich mich nur um die Produktion des Contents kuemmern brauche.

## Schoen! Und wie funktioniert das nun[^jekyll]?

### Hugo lokal

Zunaechst einmal braucht ihr [Hugo][] auf eurem Rechner. Angenehmerweise besteht Hugo nur aus einem Executable, so dass keine  Installation erforderlich ist. Einfach [hier][2] herunterladen und dann im System verfuegbar machen, fertig. Exemplarisch hier die Schritte, die dafuer auf meinem Ubuntu-Rechner noetig waren.

Zunaechst das Archiv herunterladen:

    [christoph@RON ~]$ wgethttps://github.com/spf13/hugo/releases/download/v#VERSION#/hugo_#VERSION#_#SYSTEM#_#ARCHITEKTUR#.tar.gz

Hierbei `#VERSION#`, `#SYSTEM#` und `#ARCHITEKTUR#` durch die zutreffenden Werte ersetzen. Die im Moment (November 2015) aktuellste Version von Hugo ist 0.14, auf meinem Laptop laeuft ein Linux-System, und da die Gurke schon etwas aelter ist, ein 32-bit-Linux-System. Fuer meinen Fall also lautete der Aufruf:

        [christoph@RON ~]$ wgethttps://github.com/spf13/hugo/releases/download/v0.14/hugo_0.14_linux_386.tar.gz

Nach dem der Download abgeschlossen ist, das Archiv extrahieren:

        [christoph@RON ~]$ tar -xzf hugo_0.14_linux_386.tar.gz

Hierbei nicht vergessen, den Dateinamen zu ersetzen, sollte er sich bei euch unterscheiden. Dies sollte dann in einer Datei `hugo_0.14_linux_386` resultieren (Dateiname kann je nach System abweichen). Diese kopieren wir nun an einen Ort, an dem das System nach ausfuehrbaren Dateien sucht, also einem Ordner, der sich in der `$PATH`-Variablen befindet:

        [christoph@RON ~]$ cp hugo_0.14_linux_386 /usr/bin

Nun ist das Programm von ueberall im System aus aufrufbar. Es waere aber nett, wenn wir es einfach nur durch Eingabe von `hugo` aufrufen koennten, anstatt jedes Mal kompliziert `hugo_0.14_linux_386` eingeben zu muessen. Hierzu koennten wir die Datei einfach umbenennen. Eleganter ist jedoch, einen symbolischen Link anzulegen und ihn auf diese Datei zeigen zu lassen. Auf diese Weise ist es beispielsweise moeglich, mehrere Versionen von Hugo in `/usr/bin` zu haben und die Verknuepfung auf die Version zeigen zu lassen, mit der man gerade arbeiten moechte. Also:

        [christoph@RON ~]$ ln -s /usr/bin/hugo_0.14_linux_386 /usr/bin/hugo

Das erzeugt den symbolischen Link und sorgt dafuer, dass ab sofort der Befehl `hugo` systemweit bekannt ist.

Nun ist es an der Zeit, etwas mit Hugo herumzuspielen mit dem [Hugo Quickstart Guide][3] erstellt ihr eine Hugo-Website in unter zwei Minuten und bekommt ein Gefuehl dafuer, wie Hugo funktioniert.

### Uberspace

Als naechstes braucht ihr einen [Uberspace][]. Der ist schnell und ohne Angabe persoenlicher Daten angelegt, im ersten Monat kostenlos, kostet danach so viel, wie ihr geben moechtet (mindestens 1 €) und kuendigt sich automatisch, falls ihr nichts bezahlt. Ich weiss, klingt zu gut, um wahr zu sein, [stimmt aber][4].

### Hugo auf dem Uberspace

Eigentlich waeren wir an dieser Stelle bereits fertig. Das Ergebnis des oben erwaehnten Quickstart Guides koenntet ihr nehmen, per SFTP auf euren Uberspace hochladen und haettet eine Website.

Gut, bei jeder Aenderung muesst ihr sie mit Hugo neu generieren, aber das dauert nur wenige Millisekunden. Nur: jedes Mal den FTP-Client starten, sich mit Uberspace verbinden und die ganze Website wieder hochladen? Ich rieche 90er-Jahre-Webdesign-Flair. Es muesste eine Moeglichkeit geben, die Website serverseitig neu generieren zu lassen, sobald sich etwas geaendert hat...

Und hey, die gibt es natuerlich. Zunaechst brauchen wir dafuer Hugo auch auch auf dem Uberspace, alles weitere regeln Git und Git Hooks, aber dazu spaeter mehr. Um Hugo auf eurem Uberspace zu installieren sind im Grunde die gleichen Schritte notwendig wie auf eurem Rechner, nur eben auf dem Uberspace-Server. Hier exemplarisch und im Schnelldurchlauf die Schritte, die in meinem Fall noetig waren (vorher habe ich mich per SSH zu meinem Uberspace-Server verbunden).

Hugo herunterladen:

        [harmsens@betelgeuse ~]$ wgethttps://github.com/spf13/hugo/releases/download/v0.14/hugo_0.14_linux_amd64.tar.gz

(wie hier zu sehen ist, laeuft auf den Uberspace-Servern selbstverstaendlich ein 64-bit-Linux)

Entpacken:

        [harmsens@betelgeuse ~]$ tar -xzf hugo_0.14_linux_amd64.tar.gz

An einen Ort im `$PATH` kopieren (in diesem Fall `~/bin`):

        [harmsens@betelgeuse ~]$ cp hugo_0.14_linux_amd64 ~/bin

Symbolische Verknuepfung anlegen:

        [harmsens@betelgeuse ~]$ ln -s ~/bin/hugo_0.14_linux_amd64 ~/bin/hugo

...und fertig! Nuetzt uns nur so noch nichts. Wir muessten auf dem Server entwickeln, dann mit Hugo die Seite generieren und sie womoeglich noch selbst in den Document Root verschieben. Und das jedes Mal, wenn sich etwas aendert oder etwas hinzukommt. Suboptimal. Und wie loesen wir das Problemchen? Na klar, mit...

### Git

Git wird sowohl auf dem Server als auch auf eurem Rechner benoetigt. Da [Uberspace][] nunmal der beste Hoster der Welt ist, ist Git dort bereits vorinstalliert. Hier muessen wir uns also um nichts weiter kuemmern. Die Installation auf dem Rechner haengt von eurem Betriebssystem ab, fuer Ubuntu genuegt ein simples `sudo apt-get install git`[^git].

Auf dem Server richten wir nun ein sogenanntes Bare-Repository in einem Ordner unserer Wahl ein, also ein Git-Repository ohne Arbeitskopie, in das ihr eure lokal entwickelte Seite pushen koennt. Wie das bei Uberspace geht, ist im [dortigen Wiki][6] wirklich ganz hervorragend beschrieben, so dass ich darauf hier nicht eingehe.

Also, nach dem Lesen und Ausfuehren dieser Anleitung habt ihr ein Bare-Repo auf eurem Uberspace und ein lokales Repo mit Arbeitskopie auf eurem Rechner, beide miteinander verbunden, sodass ein `git push origin master` die neuesten Aenderungen von eurem Rechner an den Uberspace-Server uebertraegt.

Nur... das bringt uns ja immer noch nicht weiter! Klar, wir haben jetzt eine Versionsverwaltung mit allen Vorteilen. Aber wir muessen immern noch von Hand

* eine Arbeitskopie auf dem Server klonen
* Hugo daraus die Website generieren lassen und
* diese dann in den Document Root verschieben

Um diese Schritte zu automatisieren, benutzen wir einen "Git Hook". Git Hooks sind Scripts, die automatisch von Git zu bestimmten Zeitpunkten ausgefuehrt werden. Da gibt es verschiedene, aber der, den wir fuer unsere Zwecke brauchen, heisst "post-recieve" und wird -- falls vorhanden -- jedes Mal ausgefuehrt, nachdem eine neue Version unserer Seite an unseren Server uebertragen wurde.

In eurem Bare-Repository auf dem Uberspace-Server findet ihr -- im Unterordner `hooks` -- die Datei `post-receive.sample`, welche ihr in einem Editor eurer Wahl oeffnet und den Inhalt dann durch folgendes ersetzt[^githook]:

        #!/bin/sh
        # Die folgende Variable speichert den Pfad zum Repository um das es geht.
        # Hier meinprojekt.git mit dem Namen Deines Repos ersetzen und ggfs.
        # den Pfad zum Repo
        GIT_REPO=$HOME/git/meinprojekt.git

        # Die folgende Variable speichert den Pfad zum tmp Ordner in dem dann der Hugo
        # Befehl ausgefuehrt wird um die deine Seite in den Webroot zu befoerdern.
        # Hier wieder "meinprojekt" mit dem Namen des Repos ersetzen ohne ".git" am Schluss.
        TMP_GIT_CLONE=$HOME/git/tmp/meinprojekt

        # Die folgende Variable speichert den Pfad zum Webroot
        # Je nach URL bitte den richtigen Pfad eintragen Wie sich das mit
        # den Webroots auf Uberspace verhaelt
        # Steht sehr ausfuehrlich im Uberspace-Wiki:
        # https://uberspace.de/dokuwiki/start:domain
        # Ersetze hier DEINUSERNAME und WEBROOTORDNER bitte mit den richtigen Namen.
        PUBLIC_WWW=/var/www/virtual/DEINUSERNAME/WEBROOTORDNER

        # Hier geht's dann ans eingemachte:
        # Mit "git clone" wird Dein Repository in das tmp-Verzeichnis geklont
        git clone $GIT_REPO $TMP_GIT_CLONE

        # Dein persoenliches .bash_profile wird aktiviert damit der
        # Hugo-Befehl benutzt werden kann.
        # Ersetze DEINUSERNAME mit deinem Uberspace Benutzernamen.
        . /home/DEINUSERNAME/.bash_profile

        # Hugo generiert die Seite aus dem tmp-Verzeichnis heraus
        # in den Webroot hinein. Ersetze DEINHUGOTEMPLATE mit dem korrekten
        # Namen des Templates, das Hugo fuer deine Seite verwenden soll.
        hugo -s $TMP_GIT_CLONE -d $PUBLIC_WWW -t DEINHUGOTEMPLATE

        # Das tmp-Verzeichnis wird geloescht und das Shell-Programm beendet.
        rm -Rf $TMP_GIT_CLONE
        exit

Diese Datei speichert ihr nun **ohne die Endung ".sample"**, also einfach als `post-receive` wieder im `hooks`-Ordner auf eurem Uberspace. Das ganze dann noch mit

        [harmsens@betelgeuse ~]$ chmod +x meinprojekt.git/hooks/post-receive

ausfuehrbar machen, und Voilà! Jedes mal, wenn ihr eure Hugo-Website lokal bearbeitet und die Aenderungen per `git push` an euren Uberspace uebertragt, wird automatisch mit Hugo die Seite neu generiert und in den Document Root verschoben. Das war's!

Ich freue mich ueber jede Art von Feedback (ja, ich werde auch sehr gerne auf Fehler aufmerksam gemacht), entweder hier in den Kommentaren oder ueber eine der vielen anderen Kontaktmoeglichkeiten in der Leiste links.

Christoph

[Hugo]: http://gohugo.io
[Uberspace]: https://uberspace.de
[Git]: http://git-scm.com/
[Typo3]: https://typo3.org
[Joomla]: http://www.joomla.de
[Franz]: https://twitter.com/laerador
[Jekyll]: https://jekyllrb.com/

[1]: https://uberspace.de/opinion
[2]: https://github.com/spf13/hugo/releases
[3]: http://gohugo.io/overview/quickstart/