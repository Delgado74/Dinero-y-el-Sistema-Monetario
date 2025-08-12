Genial, vamos con el paso de **construir y firmar la transacción de gasto Taproot** usando Python y la clave privada + tweak que generamos.

---

# Paso 3: Construir y firmar la transacción de gasto P2TR (Key-path spend)

Para este ejemplo usaremos la librería `btclib` y `python-bitcoinlib` para construir la transacción. Vamos a firmar el `sighash` BIP-341 con Schnorr (BIP-340) y completar la tx.

---

## 3.1 Instalar dependencias necesarias

```bash
pip install btclib python-bitcoinlib
```

---

## 3.2 Código ejemplo completo

Guarda como `taproot_spend.py` y edita las variables para poner tu `txid`, `vout`, clave privada `d` y tweak (ambos del paso anterior):

```python
import binascii
from btclib.ecc import ssa
from btclib import schnorr
from hashlib import sha256
from bitcoin.core import COutPoint, CTxIn, CTxOut, CTransaction, lx, COIN, b2x
from bitcoin.wallet import CBitcoinAddress, P2WSHBitcoinAddress
from bitcoin.core.script import CScript
from bitcoin.core.scripteval import VerifyScript, SCRIPT_VERIFY_WITNESS, SCRIPT_VERIFY_P2SH

# Parámetros a editar según tu setup
TXID = "tu_txid_funding_aqui"  # funding txid (string hex)
VOUT = 0                      # índice de salida funding
AMOUNT = 1 * COIN             # monto en satoshis (1 BTC)
D = int("tu_clave_privada_hex", 16)   # clave privada sin tweak (o con ajuste de paridad)
TWEAK = int("tu_tweak_hex", 16)       # tweak calculado antes

# 1. Calculamos clave privada tweakeada: d' = (d + tweak) mod n
d_tweaked = (D + TWEAK) % ssa.n

# 2. Construir tx input y output (en este ejemplo enviamos a dirección bech32 de cambio)
funding_outpoint = COutPoint(lx(TXID), VOUT)
txin = CTxIn(funding_outpoint)

# Dirección cambio regtest (puedes usar tu propia)
change_addr = CBitcoinAddress("bcrt1qchangeaddressgenerated")
change_amount = AMOUNT - 1000  # menos fee de 1000 sats aprox

txout = CTxOut(change_amount, change_addr.to_scriptPubKey())

tx = CTransaction([txin], [txout])
tx.nVersion = 2
tx.nLockTime = 0

# 3. Calcular sighash BIP-341

def tagged_hash(tag: bytes, msg: bytes) -> bytes:
    tag_hash = sha256(tag).digest()
    return sha256(tag_hash + tag_hash + msg).digest()

def compute_bip341_sighash(tx, input_index, script_pubkey, value, annex=b''):
    from btclib.tx.taproot import TaprootSighash
    sighash = TaprootSighash(tx)
    return sighash.compute_sighash(input_index, script_pubkey, value, annex)

# scriptPubKey P2TR: OP_1 (0x51) + 32 bytes pubkey tweakeada x-only
pubkey_tweaked_point = ssa._add_pubkeys(ssa._pubkey(D), TWEAK)
pubkey_tweaked_x = pubkey_tweaked_point[0]
script_pubkey = b'\x51' + pubkey_tweaked_x.to_bytes(32, 'big')

sighash = compute_bip341_sighash(tx, 0, script_pubkey, AMOUNT)

# 4. Firmar con Schnorr (BIP-340)
sig = schnorr.sign(d_tweaked, sighash)

# 5. Construir witness con firma Schnorr (key-path spend)
witness = [sig]

tx.wit.vtxinwit = []
from bitcoin.core import CTxInWitness
txinwitness = CTxInWitness()
txinwitness.scriptWitness.stack = witness
tx.wit.vtxinwit.append(txinwitness)

# 6. Serializar y mostrar tx raw hex
rawtx = b2x(tx.serialize_with_witness())
print("Transacción firmada lista para enviar:")
print(rawtx)
```

---

## 3.3 Explicación breve

* Se calcula la clave privada tweakeada sumando el tweak al secreto original (mod n).
* Se construye la transacción (input, output).
* Se calcula el sighash BIP-341 (usando `btclib` o función equivalente).
* Se firma ese hash con Schnorr.
* Se crea el witness con solo la firma (key-path spend).
* Se serializa para enviar.

---

## 3.4 ¿Cómo enviar la tx?

Guarda el hex que imprimió el script y ejecútalo en el CLI:

```bash
bitcoin-cli sendrawtransaction "hex-de-tu-tx"
```

---

# ¿Quieres que prepare un script paso a paso que incluya todo (generar clave, crear dirección, crear funding, gastar y firmar) con instrucciones detalladas para que puedas copiar y pegar y hacer todo en minutos?
