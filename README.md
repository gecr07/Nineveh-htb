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

## Vhosting

```


```































