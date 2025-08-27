# SegWit: Seguridad, Peso y Lightning Network

Este tutorial explica cómo SegWit mejoró Bitcoin en tres aspectos: **seguridad**, **peso de transacciones** y habilitación de **Lightning Network**, con ejemplos prácticos y diagramas.

---

## 1. Seguridad: txid y maleabilidad

Antes de SegWit, las transacciones podían sufrir **maleabilidad**:

- El **ID de la transacción (txid)** se calcula con todos los datos:
  - Inputs (de dónde vienen los BTC)
  - Outputs (a dónde van los BTC)
  - Firmas digitales (scriptSig)

- Problema: la firma puede escribirse de formas distintas, pero sigue siendo válida.
  - Esto cambia el **txid** sin invalidar la transacción.
  - Ejemplo intuitivo: como cambiar el número de seguimiento de un paquete sin cambiar el contenido.

### Con SegWit

- Las firmas se separan en un apartado llamado **witness**.
- El **txid ya no depende de las firmas**, eliminando la maleabilidad.
- Las firmas siguen existiendo y se validan dentro del bloque.

---

## 2. Peso de las transacciones

Antes de SegWit:

- Bloque limitado a 1 MB
- Todo se cuenta igual: inputs, outputs y firmas.

Con SegWit:

- Se introduce **block weight (peso del bloque)**:

$$
\text{peso} = \text{tamaño base} \times 3 + \text{tamaño total}
$$

- Tamaño base = inputs + outputs (sin firmas)  
- Tamaño total = incluye firmas (witness)  
- Consecuencia: las firmas pesan menos, aumentando la **capacidad efectiva** del bloque.

Ejemplo intuitivo:  
> Es como permitir 10 kg de equipaje, pero el laptop no cuenta dentro de esos 10 kg.

---

## 3. Lightning Network

- Lightning depende de **txid inmutables** para abrir canales de pago.  
- SegWit solucionó la maleabilidad → se puede construir Lightning de manera segura.

---

## 4. Esquema visual de bloques

### 4.1 Legacy (antes de SegWit)

```
+------------------------------------------------------------+
|                     BLOCK HEADER                           |
+------------------------------------------------------------+
|                     TRANSACTIONS                            |
|  Tx 1                                                      |
|    Inputs  --------------------+                            |
|    Outputs --------------------+----> txid = HASH(inputs + outputs + signatures)
|    Signatures <----------------+                            |
|                                                            |
|  Tx 2                                                      |
|    Inputs  --------------------+                            |
|    Outputs --------------------+----> txid = HASH(inputs + outputs + signatures)
|    Signatures <----------------+                            |
+------------------------------------------------------------+
```

- Las firmas forman parte del txid → cambiar la firma cambia el txid (maleabilidad).

---

### 4.2 SegWit

```
+------------------------------------------------------------+
|                     BLOCK HEADER                           |
+------------------------------------------------------------+
|                     TRANSACTIONS                            |
|  Tx 1                                                      |
|    Inputs  --------------------+                            |
|    Outputs --------------------+                            |
|                                                           txid = HASH(inputs + outputs)
|  Tx 2                                                      |
|    Inputs  --------------------+                            |
|    Outputs --------------------+                            |
|                                                           txid = HASH(inputs + outputs)
+------------------------------------------------------------+
|                     WITNESS DATA                            |
|  Tx 1 Witness (firma + pubkey) --------------------------> wtxid = HASH(inputs + outputs + witness)
|  Tx 2 Witness (firma + pubkey) --------------------------> wtxid = HASH(inputs + outputs + witness)
+------------------------------------------------------------+
```

- Las firmas se sacan del txid → txid estable y seguro.  
- Las firmas siguen existiendo → seguridad intacta.

---

## 5. Ejemplo de txid y wtxid

### 5.1 Legacy

```
Tx Inputs:
   prev_out = [hash_entrada, índice]
   scriptSig = firma_Alice + pubkey_Alice

Tx Outputs:
   0.01 BTC → Bob
   0.009 BTC → cambio Alice

txid = HASH(inputs + outputs + scriptSig)

Ejemplo ficticio:
txid = abcd1234ef567890

Si cambia la firma (firmaA → firmaA′):
txid = 9f8e7d6c5b4a3210  <-- Cambió el txid
```

### 5.2 SegWit

```
Tx Inputs:
   prev_out = [hash_entrada, índice]
   scriptSig = (vacío)

Tx Outputs:
   0.01 BTC → Bob
   0.009 BTC → cambio Alice

Witness Data:
   firma_Alice
   pubkey_Alice

txid = HASH(inputs + outputs)
     = 55aa77cc99dd0011  <-- txid estable

wtxid = HASH(inputs + outputs + witness)
      = eeff112233445566
```

- Cambiar la firma:  
  - **wtxid cambia** → eeff… → ddcc…  
  - **txid permanece igual** → 55aa77cc99dd0011 ✅

---

## 6. Flujo de cálculo en SegWit

```
Transacción SegWit
+--------------------+
| Inputs + Outputs   |
+--------------------+
          |
          v
       txid (estable, sin firmas)
          |
          v
    Contratos, Lightning Network
          
Witness Data (firma + pubkey)
          |
          v
      wtxid (incluye firma)
          |
          v
    Validación dentro del bloque
```

---

## 7. Resumen Comparativo

| Aspecto             | Legacy                 | SegWit                     |
|---------------------|----------------------|----------------------------|
| txid                 | Incluye firma        | No incluye firma (txid estable) |
| Maleabilidad         | Sí                   | No                         |
| Peso de bloque       | 1 MB fijo            | Mayor capacidad, firmas pesan menos |
| Lightning Network    | Inseguro             | Seguro                     |

---

**Conclusión:**  
SegWit separa las firmas del txid, aumentando la seguridad y la capacidad de los bloques, y habilitando Lightning Network de manera confiable.

