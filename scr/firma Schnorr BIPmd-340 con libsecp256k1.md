Perfecto, vamos con el **Ejemplo 1: firma Schnorr BIP-340 con libsecp256k1**, paso a paso y con explicación matemática y de código.

---

## 🔹 Instalación de `libsecp256k1` con soporte Schnorr

```bash
git clone https://github.com/bitcoin-core/secp256k1.git
cd secp256k1
./autogen.sh
./configure --enable-module-schnorrsig --enable-experimental
make
sudo make install
```

📌 La opción `--enable-experimental` es necesaria porque Schnorr todavía es un módulo experimental en esta librería.

---

## 🔹 Ejemplo en Python con `secp256k1` y Schnorr

```python
# pip install secp256k1 hashlib
import os, hashlib, secp256k1

# 1. Crear clave privada y pública
priv_key_bytes = os.urandom(32)
priv = secp256k1.PrivateKey(priv_key_bytes, raw=True)
pub = priv.pubkey

print("Privada (hex):", priv_key_bytes.hex())
print("Pública X-only:", pub.serialize()[1:].hex())  # quitar byte de paridad

# 2. Mensaje y hash BIP-340 (tagged hash)
msg = b"Hola Taproot con Schnorr"
tag = b"BIP0340/challenge"
tag_hash = hashlib.sha256(tag).digest()
t = hashlib.sha256(tag_hash + tag_hash + msg).digest()
print("Tagged hash (hex):", t.hex())

# 3. Firmar (Schnorr)
sig = priv.schnorr_sign(msg, None, raw=True)  # raw=True para bytes crudos
print("Firma Schnorr (64 bytes hex):", sig.hex())

# 4. Verificar
ok = pub.schnorr_verify(msg, sig, None, raw=True)
print("¿Firma válida?:", ok)
```

---

## 🔹 Explicación matemática del proceso (BIP-340)

1. **Clave privada**:
   $d \in [1, n-1]$
   donde $n$ es el orden del generador $G$.

2. **Clave pública X-only**:
   $P = d \cdot G$
   Se usa solo la coordenada $X$ y se ajusta la paridad de $Y$ para que sea par.

3. **Generación del nonce `k`**:
   Se calcula de forma determinística:
   $k = \text{hash\_bytes}(d' \parallel m)$ mod $n$
   donde $d' = d$ o $n-d$ si $Y$ de $P$ es impar.
   Esto previene la reutilización accidental de `k`.

4. **Punto `R`**:
   $R = k \cdot G$
   Si $Y_R$ es impar, reemplazar $k \leftarrow n - k$.

5. **Desafío `e`** (tagged hash):
   $e = \text{SHA256}(\text{SHA256}(\text{"BIP0340/challenge"}) \parallel \text{SHA256}(\text{"BIP0340/challenge"}) \parallel R_x \parallel P_x \parallel m)$ mod $n$

6. **Firma**:
   $s = (k + e \cdot d) \mod n$
   Firma final: `R_x || s` (64 bytes).

7. **Verificación**:
   Verifica que:
   $s \cdot G = R + e \cdot P$
   y que la coordenada Y de $R$ sea par.

---

Si quieres, el siguiente ejemplo que te puedo preparar sería **firmar una transacción Bitcoin Taproot real con Schnorr BIP-340** usando esta misma librería, para ver cómo entra en juego dentro de Bitcoin Script y PSBT.
Ese ya sería el paso final de la conexión entre la teoría y la práctica Bitcoin.

¿Quieres que lo arme?
