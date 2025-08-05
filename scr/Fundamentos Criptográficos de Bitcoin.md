# Fundamentos Criptográficos de Bitcoin

Bitcoin se basa en principios criptográficos sólidos que garantizan su seguridad, descentralización y resistencia a la censura. Esta sección introduce los conceptos matemáticos clave que todo bitcoiner debe entender para comprender cómo y por qué funciona Bitcoin.

---

## 🔐 Fundamentos de la Criptografía

La criptografía es la ciencia de proteger información mediante técnicas matemáticas. En Bitcoin, la criptografía se usa para:

- Proteger las transacciones (firmas digitales).
- Garantizar la integridad de los datos (funciones hash).
- Permitir la verificación pública sin revelar información privada (clave pública/privada).

**Tipos de criptografía usados en Bitcoin:**

| Tipo                      | Uso en Bitcoin                                  |
|---------------------------|------------------------------------------------|
| Criptografía de clave pública | Generación de direcciones y firmas           |
| Funciones hash            | Identificadores únicos, Merkle Trees, PoW      |
| Firmas digitales          | Autenticación de transacciones                  |
| Criptografía de curva elíptica | ECDSA/secp256k1: claves y firmas seguras     |

---

## 🔄 Funciones Hash y Punteros Hash

Una **función hash** toma un input de cualquier tamaño y devuelve una salida de tamaño fijo (por ejemplo, 256 bits en SHA-256). Estas funciones son:

- Deterministas: mismo input → mismo output.
- Unidireccionales: imposible derivar input a partir del output.
- Sensibles: pequeño cambio en el input → cambio total en el output.
- Colisión-resistentes: difícil encontrar dos inputs con mismo output.

### 📊 Ejemplo (SHA-256):

Input: "hola mundo"
Output: 64ec88ca00b268e5ba1a35678a1b5316d212f4f366b2477245e662b2e5c50c7e


### 📌 Punteros hash

En Bitcoin, los bloques y las transacciones se enlazan usando **punteros hash**, lo que crea una cadena verificable e inmutable.

![Gráfico de bloques enlazados por punteros hash](images/blockchain.png)

---

## 🕰 El papel de los libros contables y el sellado temporal

Bitcoin es un **libro contable público e inmutable**. Cada bloque contiene un *timestamp* (marca temporal) y un *hash* del bloque anterior. Esto actúa como un **sello temporal criptográfico** (proof-of-publication):

- Garantiza el orden de las transacciones.
- Permite probar que una información existía en cierto momento.

---

## 🌳 Árboles de Merkle y la integridad de los datos

Los árboles de Merkle permiten verificar que una transacción está en un bloque sin tener que descargar todas las transacciones.

![Gráfico de Árbol de Merkle](images/merkel.webp)

**Ventajas:**

- Eficiencia en la verificación.
- Escalabilidad (SPV wallets).
- Integridad de datos asegurada.

Cada bloque incluye una *Merkle root*, que es el hash raíz del árbol de todas las transacciones.

---

## ✍️ Firmas Digitales

Las **firmas digitales** permiten a un usuario demostrar la propiedad de una clave privada sin revelarla. En Bitcoin se usan para **autorizar transacciones**.

Proceso:

1. El usuario firma una transacción con su clave privada.
2. Cualquiera puede verificarla con su clave pública.

Esto garantiza:

- Autenticidad (solo el dueño puede gastar).
- Integridad (nadie puede modificar la transacción firmada).
- No repudio (no puede negar haber firmado).

---

## ➗ Curva Elíptica ECDSA y secp256k1

Bitcoin usa la **firma digital ECDSA** (Elliptic Curve Digital Signature Algorithm) sobre la curva **secp256k1**.

Características de secp256k1:

- Clave privada: número aleatorio de 256 bits.
- Clave pública: punto sobre la curva derivado de la clave privada.
- Seguridad basada en la dificultad de resolver el **problema del logaritmo discreto** en curvas elípticas.

![Curva Elíptica secp256k1](images/eliptica.png)

---

## 🧠 ¿Y las firmas Schnorr?

Aunque Bitcoin aún no las usa por defecto, desde la activación de **Taproot**, el protocolo admite **firmas Schnorr**, que tienen ventajas como:

- Menor tamaño y mayor eficiencia.
- Soporte para firmas agregadas (más privacidad y escalabilidad).
- Linealidad matemática (útil para protocolos avanzados como MuSig2).

---

## 🔑 Criptografía de Clave Pública

Bitcoin usa criptografía asimétrica:

| Clave privada 🕵️ | Secreta, se genera aleatoriamente. Usada para firmar. |
|------------------|--------------------------------------------------------|
| Clave pública 🌍  | Derivada de la privada. Usada para verificar firmas.  |

### Ejemplo:

```text
Clave privada:      5HueCGU8rMjxEXxiPuD5BDu...
Clave pública:      04a34b5d3... (punto en la curva elíptica)
Dirección Bitcoin:  1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
```

Nadie puede calcular la clave privada a partir de la pública.

---
📌 Principio clave: "Sin tus claves, no son tus monedas"
Bitcoin enseña soberanía digital:

🔐 Si no controlas tus claves privadas, no controlas tu dinero.

Esto subraya la importancia de la autocustodia y el uso de billeteras seguras donde tú tengas las claves.

---

✅ Conclusión
La criptografía no es solo una herramienta en Bitcoin, es el núcleo que le da:

Seguridad.

Transparencia.

Descentralización.

Confianza sin necesidad de terceros.

Comprender estos fundamentos te acerca a la esencia misma de Bitcoin.

---

📚 Lecturas recomendadas:

Bitcoin Whitepaper - Satoshi Nakamoto

Mastering Bitcoin - Andreas M. Antonopoulos

"Groking Bitcoin" - Kalle Rosenbaum

