# h11_prod

Aloitin tehtävän 2.3.2023 klo 19.22 --! MYÖHÄSSÄ!

## a) Palvelinpuoli

Kävin tarkistamassa päivitykset apache2:een.

    sudo apt-get update
    sudo apt-get -y install apache2
    
Testasin, että apache2 sivuni toimii echottamalla etusivulle uuden tekstin:

    echo "NEW FRONTPAGE" | sudo tee /var/www/html/index.html

Tein static-kansion pääkäyttäjäni kotihakemistoon:

    mkdir -p publicwsgi/seikku/static/
    echo "NEW FRONTPAGE" | sudo tee publicwsgi/seikku/static/index.html
    
Loin apacheen uuden virtualhostin:

    sudoedit /etc/apache2/sites-available/seikku.conf
    
.conf-tiedostoon kirjoitin seuraavasti:

![kuva](https://user-images.githubusercontent.com/105205141/222506669-43235cdc-a48e-4564-bc61-cd96bd16be24.png)

Tämän jälkeen otin conf-tiedoston käyttöön komennolla:

    sudo a2ensite seikku.conf

Muuttaakseni käytettävää conf-tiedostoa minun täytyi uudelleenkäynnistää apache2-palvelimeni:

    sudo systemctl reload apache2
    
Sitten oli aika kokeilla koodin toimivuutta:

![kuva](https://user-images.githubusercontent.com/105205141/222507641-9ae4baea-75f6-4354-9208-b258789c42f8.png)

Voidaan todeta, että testi vahvisti että syntaksissa ei ole virheitä.
    
Kokeilin curlata static kansiota:

![kuva](https://user-images.githubusercontent.com/105205141/222508193-96902e15-fc5d-417e-a9fa-b93c93838880.png)

Sivua ei näyttänyt löytyvän.

## a) klo 19.40 Django + Virtualenv

Siirryin publicwsgi/ hakemistooni, jonne loin uuden virtualenvin.

![kuva](https://user-images.githubusercontent.com/105205141/222508913-ca6dbe94-5055-43d6-90dc-1eba899effb1.png)

Siirryin enviin ja tein itselleni requirements.txt tiedoston:

    source env/bin/activate
    micro requirements.txt
    
![kuva](https://user-images.githubusercontent.com/105205141/222509310-b3295209-4312-4851-9a8d-b108eb7135bf.png)

Tämän jälkeen pystyin lataamaan djangon, luomaan projektin ja muokkasin conf-tiedostoani:

![kuva](https://user-images.githubusercontent.com/105205141/222509522-c47d719e-00cb-49bf-81d4-456a03df6e1f.png)

    django-admin startproject testi2
    sudoedit /etc/apache2/sites-available/seikku.conf
    
Muokkasin Tero Karvisen valmiista pohjasta hakemistopolut:

    Define TDIR /home/seikku/publicwsgi/testi2
    Define TWSGI /home/seikku/publicwsgi/testi2/testi2/wsgi.py
    Define TUSER seikkutest
    Define TVENV /home/seikku/publicwsgi/env/lib/python3.9/site-packages
    # See https://terokarvinen.com/2022/deploy-django/

    <VirtualHost *:80>
            Alias /static/ ${TDIR}/static/
            <Directory ${TDIR}/static/>
                Require all granted
            </Directory>

            WSGIDaemonProcess ${TUSER} user=${TUSER} group=${TUSER} threads=5 python-path="${TDIR}:${TVENV}"
            WSGIScriptAlias / ${TWSGI}
            <Directory ${TDIR}>
                WSGIProcessGroup ${TUSER}
                WSGIApplicationGroup %{GLOBAL}
                WSGIScriptReloading On
                <Files wsgi.py>
                    Require all granted
                </Files>
            </Directory>

    </VirtualHost>

    Undefine TDIR
    Undefine TWSGI
    Undefine TUSER
    Undefine TVENV
    
TUSER - eli testikäyttäjä on aiemmissa tehtävissäni luoma seikkutest -niminen käyttäjä, jolla ei ole sudo oikeuksia.

Uuden conf-tiedoston testasin komennolla:

    curl -sI localhost | grep Server

![kuva](https://user-images.githubusercontent.com/105205141/222513878-63771140-16eb-4793-b67e-bd84b9e8b4fe.png)

avasin testi2 projektini settings.py -tiedoston

    micro testi2/settings.py
    
![kuva](https://user-images.githubusercontent.com/105205141/222515490-1a80606a-f04d-4b1a-a296-1ae347a43040.png)

Päivitin muutokset palvelimelle:

![kuva](https://user-images.githubusercontent.com/105205141/222515726-e08820c1-fba6-43c4-bc80-31315e9f1859.png)

Lopetin tehtävän klo 20.13

Lähteet

https://terokarvinen.com/2022/deploy-django/

