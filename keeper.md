agregar el host de la pagina en 
```bash
echo "10.10.14.239 codify.htb" | sudo tee -a /etc/hosts
```
Scaneamos los puertos
```bash
nmap -sV -A 10.10.11.227
```
![[Pasted image 20231228112632.png]]
## la pagina nos deriva a este login 


![[Pasted image 20231228111022.png]]

## hacemos una busqueda en bard.google.com
```bash
4.4.4+dfsg-2ubuntu1 default credentials
```

![[Pasted image 20231228111253.png]]
## nos lleva a la pagina principal
![[Pasted image 20231228111541.png]]
## Nos digimos a admin y user

![[Pasted image 20231228111642.png]]
## Los entrega el nombre de usuario y la password inicial
![[Pasted image 20231228102311.png]]
![[Pasted image 20231228102229.png]]

## Accedemos via ssh y tenemos la bandera de usuario
![[Pasted image 20231228110711.png]]
El archivo RT3000.zip tenia dentro dos archivos el KeepPassDumpFull.dmp y el passcodes.kdbx

 Haciendo una busqueda encontramos el proximo [CVE-2023-3278
 ](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-32784)



