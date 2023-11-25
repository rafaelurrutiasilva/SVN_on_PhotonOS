# SVN i PhotonOS
<img width="220" alt="SVN_on_PhotonOS" src="https://github.com/rafaelurrutiasilva/images/blob/main/SVN_on_PhotonOS.png" align=left> <br> 
Beskriver hur du kan få SVN att fungera på PhotonOS. <br>
<br>
<br>

## Bakgrund
Efter mycket letande, test av olika bibliotek samt försökt till kompilering av SVN på Photon OS så blev lösningen att köra det som en Container applikation.

## Miljö
* VMware Photon OS v4.0
* Docker version 24.0.5, build ced0996

## Uppdatering och baskonfiguration av Photon OS
Uppdate OSet, starta och enabla docker
```
tdnf update 
systemctl start docker 
systemctl enable docker 
```
 
## Arbeta med Docker 
### Skapa en docker volym
Skapa en container volym som kommer att användas för att spara allt lokalt
```
docker volume create svn_root 
```

### Skapa Container SVN applikation
Skapa container med namnet svn-server med container imagen `elleflorio/svn-server`
Här mappas docker volymen in i container. Alla förändringar och tillägg som sker inne i container kommer att sparas i docker volymen som finns utanför  containern
```
docker run --detach --name  svn-server -p 80:80 -p 3690:3690 --volume svn_root:/home/svn --workdir /home/svn elleflorio/svn-server 
```

### Konto och passwd för webbkonton
Skapa konto och passwd för access med http som protokoll
```
docker exec -t svn-server htpasswd -b /etc/subversion/passwd svnadmin Admin_1 
```
Detta konto och lösenord kommer inte att vara persistenta och måste skapas på nytt varje gång container skapas. Lösningen är att mappa den mot en lokal area.
```
mkdir -p /etc/subversion/
docker run --detach --name  svn-server -p 80:80 -p 3690:3690 --volume svn_root:/home/svn --volume /etc/subversion:/etc/subversion --workdir /home/svn elleflorio/svn-server
```

## Testa
### Surfa in
Testa via en browser byt IP till det du har
```
http://192.168.157.143/svn/
```

### Testa med ny repo
Skapa en repon RepoTest 
```
docker exec -it svn-server svnadmin create RepoTest
```

### Se vad som skapades
```
docker exec -it svn-server ls -al RepoTest
```
### Testa via en browser igen (byt IP till det du har)
```
http://192.168.157.143/svn/
```
Nu kan du börja ladda det du vill ha. Går in i container med exec -it enligt:
```
docker exec -it svn-server sh 
```

 
