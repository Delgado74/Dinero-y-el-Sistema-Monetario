# Fundamentos criptográficos de Bitcoin

Bitcoin se sustenta en complejos principios criptográficos desarrollados en las últimas décadas [1]. En esencia, utiliza funciones matemáticas irreversibles (one-way) que son fáciles de calcular en un sentido e inviables de revertir. Estas herramientas permiten crear **secretos digitales** y **firmas digitales infalsificables**, pilares de la seguridad en blockchain. En concreto, Bitcoin aplica criptografía de clave pública: cada usuario tiene un par de claves matemáticamente relacionadas (una privada y una pública derivada). La clave pública se comparte libremente para recibir transacciones, mientras que la clave privada, mantenida en secreto, sirve para firmar transacciones y gastar los fondos asociados [2][3].

---

## Fundamentos de la criptografía

La **criptografía asimétrica** (clave pública) es la base de Bitcoin. Su esencia es que a partir de una clave privada muy grande (un número aleatorio de cientos de dígitos) se deriva una clave pública de manera unidireccional [1][2]. Esto se realiza mediante operaciones sobre curvas elípticas: dados un punto generador \(G\) y la clave privada \(k\), la clave pública se obtiene como \(K = k \cdot G\), operación fácil de realizar pero cuya inversa (el problema del logaritmo discreto) es computacionalmente inabordable [4]. Gracias a esta relación, quien posee la clave privada puede **firmar** mensajes y transacciones; cualquiera puede verificar estas firmas con la clave pública correspondiente sin conocer la clave privada [4][5].

**Claves públicas y privadas.** En la práctica, una **clave pública** se distribuye abiertamente (p. ej. para generar direcciones de recepción), mientras que la **clave privada** se guarda a buen recaudo. La clave privada es la «raíz del control» de los fondos: quien la conoce puede crear firmas válidas para mover los bitcoins correspondientes. Por ello, mantener la clave privada es la responsabilidad esencial del usuario: perderla implica perder acceso a los fondos; revelarla implica entregar el control a terceros. Este principio suele resumirse en la comunidad como: **“Sin tus claves privadas, no son tus monedas”** [5][6].

---

## Funciones hash y punteros hash

Las **funciones hash criptográficas** son otro pilar de Bitcoin. Una función hash toma datos de tamaño arbitrario (por ejemplo, los detalles de una transacción) y los reduce a una cadena fija de bits (el *hash*), de tal forma que cualquier mínimo cambio en los datos originales provoca un hash completamente distinto. Bitcoin emplea SHA-256 (doble SHA-256 en el caso de la cabecera de bloque) y RIPEMD-160 en algunos pasos de construcción de direcciones; estas funciones garantizan integridad y resistencia a colisiones en la práctica [7].

Bitcoin combina funciones hash con **punteros hash**: cada bloque contiene el hash del bloque anterior. Un puntero hash es un puntero que, además de la referencia, almacena el hash del objeto apuntado. Así, los bloques quedan encadenados; modificar un bloque anterior cambia su hash y rompe el puntero del bloque que le sigue, obligando a recalcular todos los bloques posteriores para ocultar la alteración. Esta dependencia en cadena hace que la estructura sea resistente a modificaciones retroactivas si la cadena está protegida por trabajo (PoW) y por la honestidad mayoritaria de la potencia de cómputo [8].

*(Sugerencia: aquí puedes incluir un diagrama simple que muestre dos o tres bloques enlazados por punteros hash para ilustrar cómo una alteración rompe la cadena.)*

---

## El papel de los libros contables (blockchain) y el sellado temporal

La **blockchain** actúa como un libro contable público y distribuido: registra transacciones en bloques que se encadenan mediante punteros hash y se distribuyen entre nodos participantes. Cada bloque lleva, además, un campo de *timestamp* (sello temporal) que permite situar aproximada y cronológicamente la creación del bloque. Aunque el sello temporal no es perfectamente exacto (los relojes de los mineros pueden variar), sirve para ordenar y hacer coherente el historial y evitar ataques de reordenamiento simples [9].

El sellado temporal y la estructura enlazada permiten a la red establecer una historia verificable de transacciones: para reescribir una transacción antigua habría que rehacer el trabajo de todos los bloques posteriores (coste computacional muy elevado si la cadena está protegida), lo que dificulta el doble gasto y la censura retrospectiva [9][12].

---

## Árboles de Merkle y la integridad de los datos

Dentro de cada bloque, Bitcoin utiliza un **árbol de Merkle** para resumir todas las transacciones en un único hash raíz (Merkle root). En un árbol de Merkle, las hojas son los hashes de las transacciones; los nodos internos contienen el hash de la concatenación de los hashes de sus hijos; y la raíz resume todo el conjunto. Si una transacción cambia, el efecto se propaga hasta la raíz, que dejará de coincidir [3][10].

Esta estructura tiene dos ventajas prácticas: (1) permite verificar la inclusión de una transacción concreta en un bloque sin descargar todo el bloque (útil para nodos SPV), y (2) reduce el trabajo criptográfico requerido para calcular y verificar la integridad del conjunto de transacciones, ya que la raíz sintetiza la integridad de todo el bloque [10].

*(Sugerencia: un gráfico del árbol de Merkle con 4 hojas y la ruta de verificación de una transacción ayuda mucho en la comprensión.)*

---

## Firmas digitales y su papel en Bitcoin

Las **firmas digitales** permiten autenticar transacciones sin revelar la clave privada. En Bitcoin, el emisor firma los datos de la transacción (o un hash de ellos) con su clave privada; cualquier verificador puede comprobar la firma con la clave pública y confirmar que la transacción fue autorizada por el poseedor de la clave privada correspondiente [4][5].

Las firmas ofrecen dos garantías: **autenticación** (la transacción proviene de quien dice ser) e **integridad** (los datos no fueron alterados). En la práctica, la validación de firmas es un paso central en la verificación de transacciones por parte de los nodos: las entradas de una transacción deben contener firmas válidas que demuestren derecho a gastar las salidas referenciadas [5][11].

*(Sugerencia: un diagrama del flujo “transacción → hash → firma con clave privada → verificación con clave pública” clarifica mucho este proceso.)*

---

## Curva elíptica ECDSA y secp256k1 — ¿y la firma Schnorr?

Bitcoin utiliza el algoritmo **ECDSA** (Elliptic Curve Digital Signature Algorithm) sobre la curva **secp256k1** para generar y verificar firmas digitales. La elección de curvas elípticas permite claves más pequeñas con un nivel de seguridad alto, y secp256k1 fue la curva adoptada por el diseño original de Bitcoin [4][14].

ECDSA genera firmas compactas (pares \(r, s\)) y su seguridad se basa en la dificultad del problema del logaritmo discreto en curvas elípticas. En 2021, con la activación de **Taproot**, Bitcoin incorporó soporte para el esquema de firmas **Schnorr** (BIP-340/BIP-341), que aporta ventajas: permite agregación de firmas (mejor privacidad y escalabilidad en transacciones multi-firma) y presenta propiedades algebraicas útiles para nuevas construcciones criptográficas [15]. Schnorr también simplifica ciertas pruebas y es considerado más elegante matemáticamente que ECDSA.

---

## Criptografía de clave pública: funcionamiento práctico

En términos prácticos:

- El usuario genera una **clave privada** (un número aleatorio seguro).  
- A partir de ella se calcula la **clave pública** mediante una operación de curva elíptica.  
- La clave pública (o una dirección derivada de ella) se comparte para recibir fondos.  
- Para gastar, el usuario firma la transacción con su clave privada; la firma y la clave pública (o dirección) viajan en la transacción para que los nodos verifiquen su validez.

Este diseño permite descentralización: la red no necesita confiar en ninguna entidad para autorizar pagos; basta con comprobar la firma contra la clave pública que declara la propiedad de las monedas [2][4][11].

---

## Principio práctico: «Sin tus claves, no son tus monedas»

Toda la seguridad lógica y matemática vista converge en una conclusión operativa sencilla: **la custodia de las claves privadas es la base de la propiedad en Bitcoin**. Aunque el libro contable registre que existen balances asociados a direcciones, ese registro solo permite *ver* los saldos; sólo quien controle la clave privada puede crear firmas válidas que transfieran esos saldos. Perder la clave equivale a perder acceso; cederla equivale a ceder los fondos. Este principio explica por qué se enfatiza tanto el respaldo y la custodia (hardware wallets, multisig, BIP32-HD wallets) como pilares de la seguridad operativa [5][16].

---

## Conclusión

Los fundamentos criptográficos de Bitcoin combinan: (a) funciones hash y punteros hash que garantizan integridad y conexión; (b) árboles de Merkle que resumen y permiten verificar transacciones eficientemente; (c) firmas digitales sobre curvas elípticas que dan autenticidad y control; y (d) sellado temporal y consenso por prueba de trabajo que hacen resistente la historia de transacciones. Juntos, estos elementos crean un sistema en el que la propiedad de valor está determinada por el control de claves privadas y la protección de la cadena por trabajo computacional. Entender esos componentes es imprescindible para cualquier usuario que quiera evaluar la seguridad, riesgos y prácticas de custodia en Bitcoin.

---

## Referencias

1. Diffie, W. & Hellman, M. (1976). *New Directions in Cryptography*. IEEE Transactions on Information Theory, 22(6), 644–654.  
2. Rivest, R., Shamir, A., & Adleman, L. (1978). *A Method for Obtaining Digital Signatures and Public-Key Cryptosystems*. Communications of the ACM, 21(2), 120–126.  
3. Merkle, R. C. (1980). *Protocols for Public Key Cryptosystems*. Proceedings of the IEEE Symposium on Security and Privacy.  
4. Chaum, D. (1983). *Blind Signatures for Untraceable Payments*. Advances in Cryptology — CRYPTO '82 Proceedings, 199–203.  
5. Nakamoto, S. (2008). *Bitcoin: A Peer-to-Peer Electronic Cash System*. https://bitcoin.org/bitcoin.pdf  
6. Consideraciones sobre custodia y “your keys, your coins” en la práctica de Bitcoin — documentación y guías de seguridad (ver recursos de wallets y buenas prácticas en sitios como bitcoin.org).  
7. NIST (FIPS 180-4). *Secure Hash Standard (SHS)* — especificación de SHA-256.  
8. Back, A. (1997). *Hashcash — A Denial of Service Counter-Measure*. https://www.hashcash.org/papers/hashcash.pdf  
9. Documentación técnica sobre timestamping y estructura de bloque en Bitcoin — Bitcoin Developer Documentation (https://developer.bitcoin.org / https://bitcoin.org).  
10. Merkle tree: descripciones técnicas y aplicaciones en blockchain — (ver Merkle, 1980 y literatura asociada).  
11. Funcionamiento de firmas y verificación en Bitcoin (ECDSA): descripción técnica y operativa — (ver whitepaper y documentación de Bitcoin Core).  
12. Sobre la seguridad por reescritura y coste de rehacer PoW: explicaciones en el whitepaper de Satoshi y análisis posteriores.  
13. Documentación técnica para estándares de curva secp256k1 — SECG, SEC2; librerías y referencias en secp256k1 (Bitcoin Core uses libsecp256k1).  
14. BIP-340 / BIP-341 (Schnorr signatures y Taproot).  
15. Recursos educativos y guías avanzadas (Bitcoin Core, libros técnicos y artículos de investigación sobre criptografía aplicada a blockchain).

---

