+++
date = "2015-11-23T18:06:30+01:00"
draft = false
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

Dieser Blog ist eine (größtenteils[^statisch]) statische Webseite. Soll heißen, hier steht kein klassisches Content-Management-System (CMS) wie etwa [Typo3][] oder [Joomla][] dahinter, welches jede Seite beim Aufruf generiert und dann an den Browser ausliefert, sondern der Blog besteht ganz "oldschool" aus statischen HTML-Seiten, die halt einfach auf dem Server rumliegen.

Dieses Vorgehen bietet mehrere Vorteile, Stabilität und Geschwindigkeit sind zwei von ihnen. Klar, die Seiten müssen nicht erst durch einen PHP-Interpreter (oder sonst irgendwas) gejagt werden und es sind keine Datenbankzugriffe nötig, schon alleine dadurch werden viele potentielle Fehlerquellen ausgeschaltet. Generell sollte man stets versuchen, seine Webseite so statisch wie möglich zu gestalten und nur dort auf dynamische Features zurückzugreifen, wo es nicht anders geht.

Nun *hätte* dieses Vorgehen aber auch gravierende Nachteile (ich schreibe bewusst "hätte" wegen dem, was noch kommt): Webdesign wie in den 90ern. Jede einzelne Seite muss von Hand angelegt werden, jede einzelne Verlinkung von Hand eingetragen. Abhilfe schafft hier [Hugo][], ein statischer Website-Generator. Ich möchte hier nicht zu sehr ins Detail gehen, nur so viel: Hugo übernimmt die ganzen unangenehmen, repetetiven und langweiligen Schritte für mich, so dass ich mich nur um die Produktion des Contents kümmern brauche.

## Schön! Und wie funktioniert das nun[^jekyll]?

### Hugo lokal

Zunächst einmal braucht ihr [Hugo][] auf eurem Rechner. Angenehmerweise besteht Hugo nur aus einem Executable, so dass keine  Installation erforderlich ist. Einfach [hier][2] herunterladen und dann im System verfügbar machen, fertig. Exemplarisch hier die Schritte, die dafür auf meinem Ubuntu-Rechner nötig waren.

Zuerst das Archiv herunterladen:

    [christoph@RON ~]$ wget https://github.com/spf13/hugo/releases/download/v#VERSION#/hugo_#VERSION#_#SYSTEM#_#ARCHITEKTUR#.tar.gz

Hierbei `#VERSION#`, `#SYSTEM#` und `#ARCHITEKTUR#` durch die zutreffenden Werte ersetzen. Die im Moment (November 2015) aktuellste Version von Hugo ist 0.14, auf meinem Laptop läuft ein Linux-System, und da die Gurke schon etwas älter ist, ein 32-bit-Linux-System. Für meinen Fall also lautete der Aufruf:

    [christoph@RON ~]$ wgethttps://github.com/spf13/hugo/releases/download/v0.14/hugo_0.14_linux_386.tar.gz

Nach dem der Download abgeschlossen ist, das Archiv extrahieren:

    [christoph@RON ~]$ tar -xzf hugo_0.14_linux_386.tar.gz

Hierbei nicht vergessen, den Dateinamen zu ersetzen, sollte er sich bei euch unterscheiden. Dies sollte dann in einer Datei `hugo_0.14_linux_386` resultieren (Dateiname kann je nach System abweichen). Diese kopieren wir nun an einen Ort, an dem das System nach ausführbaren Dateien sucht, also einem Ordner, der sich in der `$PATH`-Variablen befindet:

    [christoph@RON ~]$ cp hugo_0.14_linux_386 /usr/bin

Nun ist das Programm von überall im System aus aufrufbar. Es wäre aber nett, wenn wir es einfach nur durch Eingabe von `hugo` aufrufen könnten, anstatt jedes Mal kompliziert `hugo_0.14_linux_386` eingeben zu müssen. Hierzu könnten wir die Datei einfach umbenennen. Eleganter ist jedoch, einen symbolischen Link anzulegen und ihn auf diese Datei zeigen zu lassen. Auf diese Weise ist es beispielsweise möglich, mehrere Versionen von Hugo in `/usr/bin` zu haben und die Verknüpfung auf die Version zeigen zu lassen, mit der man gerade arbeiten möchte. Also:

    [christoph@RON ~]$ ln -s /usr/bin/hugo_0.14_linux_386 /usr/bin/hugo

Das erzeugt den symbolischen Link und sorgt dafür, dass ab sofort der Befehl `hugo` systemweit bekannt ist.

Nun ist es an der Zeit, etwas mit Hugo herumzuspielen. Mit dem [Hugo Quickstart Guide][3] erstellt ihr eine Hugo-Website in unter zwei Minuten und bekommt ein Gefühl dafür, wie Hugo funktioniert.

### Uberspace

Als nächstes braucht ihr einen [Uberspace][]. Der ist schnell und ohne Angabe persönlicher Daten angelegt, im ersten Monat kostenlos, kostet danach so viel, wie ihr geben möchtet (mindestens 1 €) und kündigt sich automatisch, falls ihr nichts bezahlt. Ich weiß, klingt zu gut, um wahr zu sein, [stimmt aber][4].

### Hugo auf dem Uberspace

Eigentlich wären wir an dieser Stelle bereits fertig. Das Ergebnis des oben erwähnten Quickstart Guides könntet ihr nehmen, per SFTP auf euren Uberspace hochladen und hättet eine Website.

Gut, bei jeder Änderung müsst ihr sie mit Hugo neu generieren, aber das dauert nur wenige Millisekunden. Nur: jedes Mal den FTP-Client starten, sich mit Uberspace verbinden und die ganze Website wieder hochladen? Ich rieche 90er-Jahre-Webdesign-Flair. Es müsste eine Möglichkeit geben, die Website serverseitig neu generieren zu lassen, sobald sich etwas geändert hat...

Und hey, die gibt es natürlich. Zunächst brauchen wir dafür Hugo auch auf dem Uberspace, alles weitere regeln Git und Git Hooks, aber dazu später mehr. Um Hugo auf eurem Uberspace zu installieren sind im Grunde die gleichen Schritte notwendig wie auf eurem Rechner, nur eben auf dem Uberspace-Server. Hier exemplarisch und im Schnelldurchlauf die Schritte, die in meinem Fall nötig waren (vorher habe ich mich per SSH zu meinem Uberspace-Server verbunden).

Hugo herunterladen:

    [harmsens@betelgeuse ~]$ wget https://github.com/spf13/hugo/releases/download/v0.14/hugo_0.14_linux_amd64.tar.gz

(wie hier zu sehen ist, läuft auf den Uberspace-Servern selbstverständlich ein 64-bit-Linux)

Entpacken:

    [harmsens@betelgeuse ~]$ tar -xzf hugo_0.14_linux_amd64.tar.gz

An einen Ort im `$PATH` kopieren (in diesem Fall `~/bin`):

    [harmsens@betelgeuse ~]$ cp hugo_0.14_linux_amd64 ~/bin

Symbolische Verknüpfung anlegen:

    [harmsens@betelgeuse ~]$ ln -s ~/bin/hugo_0.14_linux_amd64 ~/bin/hugo

...und fertig! Nützt uns nur so noch nichts. Wir müssten auf dem Server entwickeln, dann mit Hugo die Seite generieren und sie womöglich noch selbst in den Document Root verschieben. Und das jedes Mal, wenn sich etwas ändert oder etwas hinzukommt. Suboptimal. Und wie lösen wir das Problemchen? Na klar, mit...

### Git

Git wird sowohl auf dem Server als auch auf eurem Rechner benötigt. Da [Uberspace][] nunmal der beste Hoster der Welt ist, ist Git dort bereits vorinstalliert. Hier müssen wir uns also um nichts weiter kümmern. Die Installation auf dem Rechner hängt von eurem Betriebssystem ab, für Ubuntu genügt ein simples `sudo apt-get install git`[^git].

Auf dem Server richten wir nun ein sogenanntes Bare-Repository in einem Ordner unserer Wahl ein, also ein Git-Repository ohne Arbeitskopie, in das ihr eure lokal entwickelte Seite pushen könnt. Wie das bei Uberspace geht, ist im [dortigen Wiki][6] wirklich ganz hervorragend beschrieben, so dass ich darauf hier nicht eingehe.

Also, nach dem Lesen und Ausführen dieser Anleitung habt ihr ein Bare-Repo auf eurem Uberspace und ein lokales Repo mit Arbeitskopie auf eurem Rechner, beide miteinander verbunden, sodass ein `git push origin master` die neuesten Änderungen von eurem Rechner an den Uberspace-Server überträgt.

Nur... das bringt uns ja immer noch nicht weiter! Klar, wir haben jetzt eine Versionsverwaltung mit allen Vorteilen. Aber wir müssen immern noch von Hand

* eine Arbeitskopie auf dem Server klonen
* Hugo daraus die Website generieren lassen und
* diese dann in den Document Root verschieben

Um diese Schritte zu automatisieren, benutzen wir einen "Git Hook". Git Hooks sind Scripts, die automatisch von Git zu bestimmten Zeitpunkten ausgeführt werden. Da gibt es verschiedene, aber der, den wir für unsere Zwecke brauchen, heißt "post-recieve" und wird -- falls vorhanden -- jedes Mal ausgeführt, nachdem eine neue Version unserer Seite an unseren Server übertragen wurde.

In eurem Bare-Repository auf dem Uberspace-Server findet ihr -- im Unterordner `hooks/` -- die Datei `post-receive.sample`, welche ihr in einem Editor eurer Wahl öffnet und den Inhalt dann durch folgendes ersetzt[^githook]:

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

Diese Datei speichert ihr nun **ohne die Endung ".sample"**, also einfach als `post-receive` wieder im `hooks/`-Ordner auf eurem Uberspace. Das ganze dann noch mit

    [harmsens@betelgeuse ~]$ chmod +x meinprojekt.git/hooks/post-receive

ausführbar machen, und Voilà! Jedes mal, wenn ihr eure Hugo-Website lokal bearbeitet und die Änderungen per `git push` an euren Uberspace übertragt, wird automatisch mit Hugo die Seite neu generiert und in den Document Root verschoben. Das war's!

Ich freue mich über jede Art von Feedback (ja, ich werde auch sehr gerne auf Fehler aufmerksam gemacht), entweder hier in den Kommentaren oder über eine der vielen anderen Kon&shy;takt&shy;mög&shy;lich&shy;kei&shy;ten in der Leiste links.

Christoph

[Hugo]: http://gohugo.io
[Uberspace]: https://uberspace.de
[Git]: http://git-scm.com/
[Typo3]: https://typo3.org
[Joomla]: http://www.joomla.de
[Franz]: https://twitter.com/laerador
[Jekyll]: https://jekyllrb.com/

[1]: https://uberspace.de/opinion
[2]: https://github.com/spf13/hugo/releases
[3]: http://gohugo.io/overview/quickstart/
[4]: https://wiki.uberspace.de/philosophy:toogoodtobetrue
[5]: http://lc3dyr.de/blog/2012/07/22/Jekyll-auf-Uberspace/
[6]: https://wiki.uberspace.de/development:git#git_als_server
[7]: http://oldarticles.kahlil.co/2011/07/24/uberkyll/

[^jekyll]: Diese Anleitung ist stark inspiriert von [einem anderen Tutorial][5], geschrieben von [Franz][]. Franz erklärt, wie man das gleiche Ziel erreicht, allerdings benutzt er dafür [Jekyll][] statt Hugo. Franz war es übrigens auch, der mich, als ich ihn wegen seines Tutorials auf Twitter anschrieb, auf Hugo aufmerksam machte, wovon ich bis dato gar nichts wusste. Danke dafür!

[^git]: Aus Gründen der Übersichtlichkeit (und sicher auch der mangelnden Kenntnis) verzichte ich in diesem Artikel darauf, genau zu erklären, was Git ist, was es macht und wie es im Einzelnen funktioniert, sondern beschränke mich auf die Schritte, die nötig sind, um das hier beschriebene Ziel zu erreichen. Der/die geneigte LeserIn möge mir verzeihen.

[^githook]: Dieses Skript habe ich geklaut, und zwar [hier][7], und es dann an meine Bedürfnisse angepasst.

[^statisch]: Größtenteils deswegen, weil einige Dinge, wie zum Beispiel die Kommentarfunktion oder die Social-Media-PlugIns, leider nur mit JavaScript funktionieren. Einzige Alternative wäre Verzicht, und dazu bin ich (noch) nicht bereit.