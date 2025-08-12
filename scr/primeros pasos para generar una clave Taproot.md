¡Perfecto! Vamos a avanzar con los primeros pasos para generar una clave Taproot, crear la dirección P2TR, y preparar todo para recibir fondos en regtest.

---

# Paso 1: Generar clave Taproot (clave privada y pública x-only Schnorr)

Usaremos Python con la biblioteca `btclib` para crear una clave privada, derivar la clave pública x-only, y calcular la dirección Taproot (bech32m P2TR).

```python
# taproot_key_and_address.py
from os import urandom
from btclib.ecc import ssa
from btclib import schnorr
from hashlib import sha256

def tagged_hash(tag: bytes, msg: bytes) -> bytes:
    tag_hash = sha256(tag).digest()
    return sha256(tag_hash + tag_hash + msg).digest()

# 1. Generar clave privada
d = int.from_bytes(urandom(32), "big") % ssa.n
if d == 0:
    raise Exception("Clave privada cero, reintentar")

# 2. Obtener clave pública (punto)
P = ssa._pubkey(d)  # (x, y)
Px = P[0]

# Ajustar paridad de Y: si Y es impar, usar d' = n - d
if P[1] % 2 != 0:
    d = ssa.n - d
    Px = ssa._pubkey(d)[0]

print(f"Clave privada: {d:064x}")
print(f"Clave pública x-only: {Px:064x}")

# 3. Calcular tweak para Taproot: Q = P + tweak*G
# Para key-path spends sin scripts el tweak = hash("TapTweak" || Px)
tweak_bytes = tagged_hash(b"TapTweak", Px.to_bytes(32, "big"))
tweak_int = int.from_bytes(tweak_bytes, "big") % ssa.n
print(f"Tweak int: {tweak_int:064x}")

# 4. Q = P + tweak*G
Q = ssa._add_pubkeys(Px, tweak_int)
Qx = Q[0]
print(f"Clave pública Tweakeada (Qx): {Qx:064x}")

# 5. Generar dirección bech32m P2TR
from btclib.bech32 import encode as bech32_encode

def p2tr_address(pubkey_x: int, hrp="bcrt") -> str:
    # 0x01 version para P2TR + 32 bytes pubkey_x
    witver = 1
    witprog = pubkey_x.to_bytes(32, "big")
    # convert bits de 8 a 5 (bech32m)
    from btclib.bech32 import convertbits
    data = [witver] + list(convertbits(witprog, 8, 5, True))
    return bech32_encode(hrp, data, bech32_variant="bech32m")

address = p2tr_address(Qx)
print(f"Dirección P2TR (regtest): {address}")
```

Guarda esto como `taproot_key_and_address.py` y ejecútalo:

```bash
python3 taproot_key_and_address.py
```

Esto te mostrará:

* Clave privada
* Clave pública x-only
* Tweak
* Clave pública tweakeada (Q)
* Dirección P2TR regtest (`bcrt1p...`)

---

# Paso 2: Generar una transacción funding a esa dirección en regtest

Ahora, usando `bitcoin-cli`:

```bash
# Guarda la dirección que salió en la variable
P2TR_ADDR="bcrt1p..."  # reemplaza con tu dirección generada

# Envía 1 BTC (100000000 sats) a la dirección Taproot
TXID=$(bitcoin-cli sendtoaddress "$P2TR_ADDR" 1)
echo "Funding TXID: $TXID"

# Genera 1 bloque para confirmar (puedes generar más para madurar coinbase si usaste fondos minados)
bitcoin-cli generatetoaddress 1 $(bitcoin-cli getnewaddress)
```

---

Cuando confirmes que tienes el funding en esa dirección, avanzamos a:

* Extraer ese UTXO (txid, vout)
* Construir transacción de gasto (spending) con sighash BIP-341
* Firmar con Schnorr usando la clave privada `d` y tweak
* Completar y enviar la tx

---

¿Quieres que te prepare la parte de construir y firmar la transacción de gasto en Python ahora?
