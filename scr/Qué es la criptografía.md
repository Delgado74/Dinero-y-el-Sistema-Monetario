# ¿Qué es la criptografía?

La criptografía es el conjunto de técnicas y principios que permiten proteger la información: garantizar confidencialidad, integridad, autenticidad y, en algunos casos, no repudio. Está basada en matemáticas, teoría de la información y diseño de protocolos.

# Propiedades de seguridad importantes

* **Confidencialidad**: sólo los autorizados leen el mensaje.
* **Integridad**: el mensaje no fue alterado.
* **Autenticidad**: se puede verificar quién generó el mensaje.
* **No repudio**: el autor no puede negar su autoría (relacionado con firmas digitales).
* **Frescura / prevención de replay**: evitar reenvíos/repeticiones de mensajes válidos.

# Primitivas criptográficas básicas

1. **Funciones hash**

   * Mapéan datos de longitud arbitraria a un valor fijo (ej.: SHA-256).
   * Propiedades: resistencia a preimagen, segunda preimagen, colisiones.
   * Uso: integridad de datos, prueba de trabajo, Merkle trees, derivación de claves.

2. **Cifrado simétrico**

   * Misma clave para cifrar y descifrar (ej.: AES).
   * Rápido y usado para datos a gran volumen.
   * Importante: modos de operación (GCM, CBC...) y autenticación (AEAD).

3. **Cifrado asimétrico (clave pública)**

   * Par clave pública / clave privada (ej.: RSA, curvas elípticas).
   * Usos: intercambio de claves, firma digital, cifrado de pequeñas cantidades de datos.

4. **Firmas digitales**

   * Prueba de que quien tiene la clave privada firmó un mensaje (ej.: ECDSA, Schnorr).
   * Verificables con la clave pública.
   * Requisitos: buen generador de números aleatorios para ciertos esquemas (evitar fugas).

5. **Intercambio de claves**

   * Permite establecer una clave simétrica compartida (ej.: Diffie–Hellman, ECDH).
   * Base de TLS y otros protocolos seguros.

6. **Derivación de claves y KDFs**

   * Transforman secretos en claves con propiedades deseadas (PBKDF2, HKDF, Argon2).

7. **Generadores de números pseudoaleatorios (RNG/PRNG/ CSPRNG)**

   * Cruciales: la seguridad depende de buena aleatoriedad (hardware RNG, /dev/urandom, libs OS).

8. **Primitivas avanzadas**

   * **MAC / AEAD**: autenticación + cifrado (GCM, ChaCha20-Poly1305).
   * **Zero-knowledge proofs**, **homomorphic encryption**, **firmas de grupo**, **PRFs**, etc. (temas avanzados y específicos).

# Intuición matemática (muy breve)

* **RSA**: seguridad basada en dificultad de factorizar grandes enteros.
* **Curvas elípticas (ECC)**: seguridad en la dificultad del problema del logarítmo discreto en curvas; claves mucho más cortas que RSA para el mismo nivel de seguridad.
* **Funciones hash**: diseño para ser un “compressor” no invertible y resistente a colisiones.

# Firmas y un punto habitual de confusión (ej.: ECDSA)

* En esquemas como ECDSA aparece una base `G` —es un punto fijo en la curva elíptica conocido por todos (generador).
* Para firmar se usa un número aleatorio efímero `k` (a veces llamado `r` en explicaciones) por firma; si se reutiliza `k` o es predecible, se filtra la clave privada.
* `r` y `s` son los dos valores de la firma; `H(m)` es el hash del mensaje `m` que se firma.
* Por eso se insiste en usar CSPRNG y bibliotecas probadas (ej.: secp256k1 para Bitcoin).

# Protocolos y composición

La criptografía útil es combinar primitivos correctamente en protocolos (ej.: TLS, SSH, PGP). Fallos comunes vienen por diseño incorrecto o mal uso (no por fallos matemáticos en las primitivas bien estudiadas).

# Vulnerabilidades prácticas

* **Mala aleatoriedad** → claves o noces comprometidas.
* **Errores de implementación** → timing attacks, side-channels (lectura por consumo de tiempo, consumo de energía).
* **Gestión de claves pobre** → almacenarlas en texto plano, backups inseguros.
* **Protocolos mal diseñados** → ataques replay, downgrade.
* **Cryptanalysis y avances (ej. computación cuántica)** → ciertos algoritmos (RSA, ECC) serían vulnerables a algoritmos cuánticos teóricos; ya se trabaja en criptografía post-cuántica.

# Buenas prácticas (prácticas recomendadas)

* No inventes cifrados: **usa bibliotecas probadas** (libsodium, OpenSSL, BoringSSL, libs for your platform).
* Usa **algoritmos actuales y modos seguros** (AEAD, claves largas, curvas recomendadas).
* Usa RNG del sistema o hardware TRNG; evita implementaciones caseras.
* Manejo seguro de claves: hardware wallets, HSMs, cifrado de discos, almacén de secretos seguro.
* Backups cifrados de claves privadas y frases semilla, con múltiples copias físicas separadas.
* Rotación y revocación de claves cuando sea posible.
* Revisión/ auditoría de código y uso de bibliotecas mantenidas.
* En protocolos, preferir estándares y revisiones por la comunidad (RFC, NIST, IETF).

# Aplicación práctica rápida: Bitcoin y firmas

* Bitcoin usa curvas elípticas (secp256k1) y firmas digitales.
* Riesgo clásico: reutilizar el valor aleatorio `k` en ECDSA revela la clave privada.
* Mejoras modernas: Schnorr signatures (BIP-340) ofrecen ventajas como linealidad y agregación.

# Recursos para profundizar

* Libros: “Understanding Cryptography” (Christof Paar), “Serious Cryptography” (Jean-Philippe Aumasson).
* RFCs: RFC 8446 (TLS 1.3) para ver cómo se usan primitives en protocolos.
* Practicar: experimentar con libs como libsodium, OpenSSL y herramientas como `gpg`, `age`.
* Cursos y vídeos de criptografía aplicada y teoría de números.

---

Si quieres, puedo ahora:

* Explicar **con detalle matemático** cómo funciona ECDSA/Schnorr paso a paso (incluyendo `k`, `r`, `s`, `H(m)` y por qué falla si `k` se repite).
* Hacer un **mapa visual** de cómo se combinan hashes, firmas y cifrado en TLS o en una transacción Bitcoin.
* Recomendar **bibliotecas y comandos** concretos para generar claves, firmar y verificar.

¿Qué prefieres que desarrollemos ahora?
