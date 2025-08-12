Perfecto, entonces vamos a continuar siguiendo el orden que te propuse.

Ya hemos cubierto la explicación matemática y el mapa visual; ahora vamos con **ejemplos prácticos ejecutables** y una **demostración numérica de cómo se rompe ECDSA si se reutiliza `k`**.

---

## 5) Ejemplo práctico en Python — ECDSA con `coincurve` (rápido y seguro)

```python
# pip install coincurve
from coincurve import PrivateKey, PublicKey
from hashlib import sha256

# 1. Crear clave privada y pública
priv = PrivateKey()
pub = priv.public_key
print("Privada (hex):", priv.to_hex())
print("Pública (hex, comprimida):", pub.format(compressed=True).hex())

# 2. Mensaje y hash
msg = b"Hola Bitcoin!"
h = sha256(msg).digest()

# 3. Firmar
sig = priv.sign(h)  # Firma ECDSA (DER por defecto)
print("Firma (DER hex):", sig.hex())

# 4. Verificar
is_valid = pub.verify(sig, h)
print("¿Firma válida?:", is_valid)
```

📌 Este código usa `secp256k1` (misma curva que Bitcoin) y hace todo el ciclo: genera claves, firma y verifica.

---

## 6) Ejemplo práctico en Python — Schnorr (BIP-340) con `btclib`

```python
# pip install btclib
from btclib import schnorr
from btclib.ecc import ssa
from os import urandom
from hashlib import sha256

# 1. Clave privada y pública
x = int.from_bytes(urandom(32), "big") % ssa.n
P = ssa._pubkey(x)
print("Privada:", hex(x))
print("Pública X:", hex(P[0]))

# 2. Mensaje y hash
msg = b"Hola Taproot!"
h = sha256(msg).digest()

# 3. Firmar (BIP-340)
sig = schnorr.sign(x, msg)
print("Firma Schnorr (64 bytes hex):", sig.hex())

# 4. Verificar
ok = schnorr.verify(P[0].to_bytes(32, "big"), msg, sig)
print("¿Firma válida?:", ok)
```

📌 `btclib` ya aplica el *tagged hash* de BIP-340, maneja la paridad de Y y normaliza coordenadas, así que no hay que reinventar nada.

---

## 7) Ejemplo numérico simplificado — romper ECDSA si `k` se repite

Para hacerlo entendible vamos a trabajar con números muy pequeños y una curva ficticia:

```
Parámetros:
n = 23  (orden del generador, número pequeño para ejemplo)
Clave privada: d = 7
Mensajes: m1 = "hola", m2 = "adios"
Hash(m1) = e1 = 5
Hash(m2) = e2 = 14
```

Supongamos que **por error** usamos el mismo `k = 10` en ambas firmas.

Firma 1:

```
r = (k·G)_x mod n = 3   # (valor ficticio para ejemplo)
s1 = k^{-1}·(e1 + d·r) mod n
   = 10^{-1}·(5 + 7·3) mod 23
   = 7 · (5 + 21) mod 23   # 10^{-1} mod 23 = 7
   = 7 · 26 mod 23
   = 7 · 3 mod 23 = 21
```

Firma 2:

```
s2 = k^{-1}·(e2 + d·r) mod n
   = 7 · (14 + 21) mod 23
   = 7 · 35 mod 23
   = 7 · 12 mod 23 = 15
```

Atacante ve:

```
r = 3, s1 = 21, e1 = 5
r = 3, s2 = 15, e2 = 14
```

Recuperar `k`:

```
k = (e1 - e2) · (s1 - s2)^{-1} mod n
  = (5 - 14) · (21 - 15)^{-1} mod 23
  = (-9 mod 23) · (6^{-1} mod 23)
  = 14 · 4 mod 23         # 6^{-1} mod 23 = 4
  = 10  <-- ¡es el k!
```

Recuperar `d`:

```
d = (s1·k - e1) · r^{-1} mod n
  = (21·10 - 5) · 3^{-1} mod 23
  = (210 - 5) · 8 mod 23   # 3^{-1} mod 23 = 8
  = 205 · 8 mod 23
  = (21)·8 mod 23 = 168 mod 23 = 7  <-- ¡clave privada!
```

🔍 Moral de la historia: **reutilizar `k` = pérdida instantánea de la clave privada**.

---

Yo te sugiero que el siguiente paso sea que te prepare **un ejemplo Taproot real con firma Schnorr BIP-340** usando `libsecp256k1` para que veas cómo sería en Bitcoin.
Esto conectaría todo lo que vimos con la aplicación directa en transacciones modernas.

¿Quieres que lo haga?
