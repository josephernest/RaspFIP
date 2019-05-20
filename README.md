RaspFIP
=======

![](https://upload.wikimedia.org/wikipedia/commons/archive/5/50/20131010030407%21FIP_logo.png)

Soyons clairs : [FIP](http://fipradio.fr/player) est l'une des meilleures radios en France (merci Radio France) à écouter pendant des heures sans se lasser, par exemple quand on travaille : peu de distraction en parole (une ou deux fois par heure et avec des voix qui font la [réputation de la station](https://fr.wikipedia.org/wiki/FIP_(radio)#1971_-_1972_:_cr%C3%A9ation_d'une_radio_atypique)), et surtout de la musique. Et de la bonne musique - on peut passer en 15 minutes d'un jazz hyper pointu à une nouveauté indie-pop (qu'on a juste envie de Shazamer pour découvrir l'album), et avoir ensuite du J.S. Bach suivi par du hip-hop. Les années étudiantes où j'écoutais beaucoup FIP je découvrais des nouveaux albums chouettes toutes les semaines.

Seul problème : la station n'émet en FM qu'à Paris (105.1 Mhz), Strasbourg (92.3 Mhz), Bordeaux, et cinq ou six grandes villes, **mais pas ailleurs**, hélas! Pendant longtemps j'ouvrais un onglet dans le browser à la page [FIP](http://fipradio.fr/player), mais après il faut brancher son laptop à des haut-parleurs pour avoir un bon son ou alors se connecter via Bluetooth, de même on peut utiliser l'app FIP sur téléphone, mais finalement, son téléphone est occupé par ça, et j'ai remarqué *qu'à l'usage*, **le fait d'avoir *plusieurs actions à faire* pour démarrer la radio** faisait que je l'écoutais moins souvent que quand j'étais dans une ville couverte en FM, **où il suffisait d'appuyer sur ON** sur son "poste de radio".

Comme dans la plupart de mes projets open-source (exemple [celui-ci](https://github.com/josephernest/Yopp)) où je suis convaincu que **le taux d'utilisation au quotidien d'un outil donné est inversement proportionnel au nombre d'actions nécessaires**, j'ai cherché une solution où je peux démarrer cette radio en UNE SEULE ACTION.

![](https://i.imgur.com/WdkPj3tm.jpg)

Voici donc : **RaspFIP - Un FIP player sur Raspberry Pi**. Il existe sans-doute des centaines de media player sur Raspberry Pi, mais j'ai cherché à faire un truc simple et qui fait qu'une seule chose : 

* au boot ça démarre automatiquement la radio FIP...
* ... et quand on veut éteindre on coupe l'alim et c'est tout,
* pas d'écran ni de clavier ni de souris nécessaire

L'utilisation est donc la suivante : quand on arrive dans la pièce, on appuie sur ON sur sa multiprise, et 10 secondes plus tard, la radio démarre (Raspberry Pi branché une fois pour toutes sur un ampli ou chaîne Hifi ou haut-parleurs). Rien d'autre à faire.

Après quelques années d'utilisation, je constate qu'avec cette méthode j'écoute à nouveau cette radio autant que quand j'avais juste à allumer ma radio FM calée sur la bonne fréquence (dans une ville couverte en FM).

Comment faire une RaspFIP ?
===========================

Ca se fait en quelques étapes simples en 15 minutes qu'on fait seulement une fois pour toutes. Je précise qu'on a pas besoin d'avoir Linux sur son ordinateur pour faire cela.

* Avoir un Raspberry Pi sous la main (RPi 3 ou RPi 2 avec un dongle Wifi)

* Télécharger une Raspbian Strech Lite [ici](https://www.raspberrypi.org/downloads/raspbian/) et le mettre sur une carte micro SD de 2 Go au moins avec [Win32Diskimager](https://sourceforge.net/projects/win32diskimager/) ou [Etcher](https://www.balena.io/etcher/) (qui a le bon goût de fonctionner avec des images zippées)

* Ouvrir la partition `boot` (visible depuis Windows) et créer deux fichiers :

    - Un fichier vide nommé `ssh` (astuce pour créer un fichier sans extension sous Windows : le nommer `ssh.`)

    - Un fichier nommé `wpa_supplicant.conf` contenant votre connexion Wifi avec mot de passe:

            ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
            update_config=1
            country=FR
            network={
                ssid="Livebox-ABCD"
                psk="1f0c1164b05329ac382a"
            }

* On met la carte micro SD dans le Raspberry Pi et on le boote, ensuite on se connecte en SSH, par exemple avec [`putty.exe`](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe) sous Windows :

        D:\tools\putty.exe pi@192.168.1.48 -pw raspberry

* On entre les commandes suivantes 

        sudo apt-get update
        sudo apt-get install mplayer
        sudo nano /etc/systemd/system/radio.service

    et entrer le texte suivant:

        [Unit]
        Description=Radio
        Wants=network-online.target
        After=network.target network-online.target

        [Service]
        Type=oneshot
        ExecStart=/usr/bin/mplayer http://direct.fipradio.fr/live/fip-midfi.mp3 &
        RemainAfterExit=yes

        [Install]
        WantedBy=multi-user.target

    on quitte cet éditeur avec CTRL+X. 

    On termine ensuite avec:

        sudo systemctl enable radio
        sudo alsamixer     # mettre le volume au max ici une fois pour toutes, et on quitte avec F4
        sudo alsactl store
        sudo reboot

* Voilà ! Désormais il suffit d'allumer le Pi (branché avec prise la jack sur des haut-parleurs ou ampli) et ça démarre. 

PS : *"Eteindre un Pi en enlevant le courant ça se fait pas"*. Certes, mais je le fais depuis plusieurs années, aucun problème jusque là. Au pire je changerai la carte micro SD un jour si nécessaire. (J'avais testé un jour de configurer pour monter le filesystem en `read-only` pour éviter les problèmes éventuels, mais ça ne fonctionnait pas, car le DHCP ou la connection Wifi a besoin d'écrire à un moment je crois ?)
