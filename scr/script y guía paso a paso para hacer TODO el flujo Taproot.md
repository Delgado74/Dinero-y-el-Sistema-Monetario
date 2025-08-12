¡Claro! Aquí tienes un script y guía paso a paso para hacer TODO el flujo Taproot key-path spend en **regtest**: desde generar la clave y dirección, crear funding, construir, firmar y enviar la tx de gasto.

---

# Guía y script todo-en-uno para Taproot regtest

### Requisitos previos:

* Bitcoin Core corriendo en regtest, wallet creada y fondos listos.
* Python 3 y librerías: `btclib` y `python-bitcoinlib`.

---

## Paso 0: Instala librerías Python (una sola vez)

```bash
pip install btclib python-bitcoinlib
```

---

## Paso 1: Script completo `taproot_regtest_flow.py`

```python
#!/usr/bin/env python3

import os
from hashlib import sha256
from btclib.ecc import ssa
from btclib import schnorr
from btclib.bech32 import encode as bech32_encode, convertbits
from bitcoin.core import (
    b2x, lx, COIN, COutPoint, CTxIn, CTxOut, CTransaction,
)
from bitcoin.wallet import CBitcoinAddress
from bitcoin.rpc import Proxy

def tagged_hash(tag: bytes, msg: bytes) -> bytes:
    tag_hash = sha256(tag).digest()
    return sha256(tag_hash + tag_hash + msg).digest()

def p2tr_address(pubkey_x: int, hrp="bcrt") -> str:
    witver = 1
    witprog = pubkey_x.to_bytes(32, "big")
    data = [witver] + list(convertbits(witprog, 8, 5, True))
    return bech32_encode(hrp, data, bech32_variant="bech32m")

# --- 1. Generar clave privada y pública x-only ---
d = int.from_bytes(os.urandom(32), "big") % ssa.n
if d == 0:
    raise Exception("Clave privada cero, reintentar")
P = ssa._pubkey(d)  # (x,y)
if P[1] % 2 != 0:
    d = ssa.n - d
    P = ssa._pubkey(d)
Px = P[0]

# --- 2. Calcular tweak (key-path spend sin scripts) ---
tweak_bytes = tagged_hash(b"TapTweak", Px.to_bytes(32, "big"))
tweak = int.from_bytes(tweak_bytes, "big") % ssa.n

# --- 3. Clave tweakeada Q = P + tweak*G ---
Q = ssa._add_pubkeys(Px, tweak)
Qx = Q[0]

print(f"Clave privada (hex): {d:064x}")
print(f"Tweak (hex): {tweak:064x}")
print(f"Clave pública tweakeada Qx (hex): {Qx:064x}")

# --- 4. Dirección Taproot (regtest) ---
address = p2tr_address(Qx)
print(f"Dirección P2TR (regtest): {address}")

# --- 5. Conectar con bitcoin core regtest ---
rpc = Proxy(service_url="http://usuario:contraseña@localhost:18443")  # cambia usuario/contraseña

# --- 6. Enviar 1 BTC a la dirección ---
txid = rpc.sendtoaddress(address, 1)
print(f"Funding TXID: {txid}")

# --- 7. Confirmar (minar 1 bloque) ---
rpc.generatetoaddress(1, rpc.getnewaddress())

# --- 8. Obtener funding UTXO ---
tx = rpc.gettransaction(txid)
raw_tx = rpc.getrawtransaction(txid)
decoded_tx = rpc.decoderawtransaction(raw_tx)
vout = None
for o in decoded_tx['vout']:
    if o['scriptPubKey']['addresses'][0] == address:
        vout = o['n']
        break
if vout is None:
    raise Exception("No se encontró la salida con la dirección P2TR")

print(f"Funding output index: {vout}")

# --- 9. Construir tx gasto (spend) ---
funding_outpoint = COutPoint(lx(txid), vout)
txin = CTxIn(funding_outpoint)

change_addr = rpc.getnewaddress()
change_amount = int(1 * COIN - 1000)  # 1000 sats fee aprox

txout = CTxOut(change_amount, CBitcoinAddress(change_addr).to_scriptPubKey())
spend_tx = CTransaction([txin], [txout])
spend_tx.nVersion = 2
spend_tx.nLockTime = 0

# --- 10. Calcular sighash BIP-341 ---
from btclib.tx.taproot import TaprootSighash

script_pubkey = b'\x51' + Qx.to_bytes(32, 'big')  # OP_1 + 32 bytes key tweakeada

sighash_obj = TaprootSighash(spend_tx)
sighash = sighash_obj.compute_sighash(0, script_pubkey, int(1 * COIN))

# --- 11. Firmar sighash con clave tweakeada ---
d_tweaked = (d + tweak) % ssa.n
sig = schnorr.sign(d_tweaked, sighash)

# --- 12. Crear witness con la firma ---
from bitcoin.core import CTxInWitness
witness = CTxInWitness()
witness.scriptWitness.stack = [sig]
spend_tx.wit.vtxinwit.append(witness)

# --- 13. Serializar y enviar la tx gasto ---
raw_spend_tx = b2x(spend_tx.serialize_with_witness())
print(f"Raw tx gasto:\n{raw_spend_tx}")

txid_spend = rpc.sendrawtransaction(spend_tx)
print(f"TXID gasto: {txid_spend}")

# --- 14. Minar bloque para confirmar gasto ---
rpc.generatetoaddress(1, rpc.getnewaddress())
print("Gasto confirmado.")
```

---

## Paso 2: Ejecutar el script

* Cambia `"usuario:contraseña"` por tu usuario y contraseña RPC en `bitcoin.conf`.
* Dale permisos y ejecútalo:

```bash
chmod +x taproot_regtest_flow.py
./taproot_regtest_flow.py
```

---

## Qué hace el script:

* Genera clave privada y pública Tweakeada (Taproot key-path)
* Crea dirección P2TR en regtest
* Envía 1 BTC a esa dirección y mina bloque para confirmar
* Construye tx gasto, calcula sighash BIP-341
* Firma con Schnorr (BIP-340)
* Envía la tx gasto y mina bloque para confirmar

---

Si quieres, te puedo ayudar a dividir cada paso con explicaciones detalladas o preparar versiones separadas para que entiendas cada parte por separado.

¿Quieres?
