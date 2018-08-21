# wpcm
Web Password &amp; Credential Management

Provee metodos para el manejo del password de un cliente web desde que la misma se introduce en el navegador hasta que se almacena en la base de datos. Tambien provee metodos para el almacenmiento de credenciales tales como Tokens o Secrets.

## Manejo del Password
* Passoword Hashing
* Salt per hashing (mitigar rainbow tables)
* Almacenamiento: Hashing mediante funcion diseñada para passwords KDF (key derivation function, evitar SHA-*) 

## Multiples layers de seguridad

Layer externo (cliente web - user form).

* SHA3-512(password + domain)

Network Layer

* https

API (server side)

* Argon2d(salt, hash)
* AES256-GCM

## Layer Externo (cliente web - user form)

Comienza cuando el usuario ingresa su password en un website e inicia el login.
Antes de enviar la informacion al servidor se encriptan mediante una pasada por SHA3-512 el password y un nombre unico para nuestro servicio (por ejemplo el nombre de dominio) que funcionara como salt.

Pros:

* El hash se vuelve el verdadero "password" del usuario.
* Asegura que nunca llegue al servidor la informacion cruda introducida por el usuario, evitando que esta pueda ser accidentalmente almacenada en un log u otro medio.
* Aliviana un ataque main-in-the-middle aunque la fuerza del hash SHA3-512 depende directamente del input del usuario que puede ser flojo en la mayoria de los casos.
* Agregar el nombre unico de nuestro servicio asegura que los passwords no pueden ser inferidos mediante password farming a la vez que complica el uso de rainbow-tables.
* Normalizacion: el hash SHA3-512 produce un output de 64 bytes fijos. Dicha longitud contibuye a validar del lado del servidor que se este manejando una representacion SHA3-512 valida. Tambien evita el truncamiento en N bytes de algunas funciones hash (bcrypt por ejemplo trunca despues de 72 bytes o ante un byte NUL). Por otro lado evita ataques DDoS a funciones de hashing (como PBKDF2) que permiten cadenas arbitrariamente largas.

Cons:

* No se puede verificar la fuerza del password del lado del servidor
* En caso de cambiar el nombre unico del servicio (la direccion web) se debe mantener el dominio anterior para validacion o cambiar el algoritmo de validacion de manera transparente.

## API (server side)

Como primer paso se utilizan el hash provisto por el cliente junto con un salt asociado a su cuenta para generar la KDF. Una vez obtenida el password es [implausible](https://crypto.stackexchange.com/questions/46718/impossibility-vs-implaussiblity/46861#46861) de ser inferido.

En segundo lugar se encripta el hash usando un algoritmo de encriptacion simetrico como AES256-GCM o ChaCha20-Poly1305 para que un dump de base de datos sea totalmente inutil para ataques brute-force

Dependencies:

* [crypto-js](https://www.npmjs.com/package/crypto-js): Implementation en javascript para generar SHA3-512 en el cliente.
* Express


# TODO:
* https implementacion
* almacenamiento de API/Secret

## Ref:
[Password and credential management](https://medium.com/@harwoeck/password-and-credential-management-in-2018-56f43669d588)

[Password storage cheatsheet](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet)
