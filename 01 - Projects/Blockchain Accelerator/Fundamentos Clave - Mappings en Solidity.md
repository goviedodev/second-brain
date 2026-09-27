---
created: 2026-09-27
tags: [project, blockchain, solidity, fundamentos]
source: Blockchain Accelerator - Skool (Jose)
---

# Fundamentos Clave - Mappings en Solidity

En el desarrollo de contratos inteligentes para la EVM (Ethereum Virtual Machine), el `mapping` es una de las estructuras de datos más esenciales y utilizadas, especialmente para la gestión contable y el control de accesos.

---

## 1. Definición y Analogía

Un `mapping` funciona como una tabla tipo **clave-valor** (equivalente conceptual a una tabla hash, un diccionario en Python o un `HashMap` en Java/JavaScript):

```solidity
// mapping(TipoClave => TipoValor) visibilidad nombre;
mapping(address => uint256) public balances;
```

- **Clave (`KeyType`):** Puede ser cualquier tipo de dato elemental de Solidity (como `address`, `uint256`, `bytes32`), excepto tipos dinámicos o complejos como structs u otros mappings. En finanzas descentralizadas (DeFi), suele ser una dirección de cuenta o contrato (`address`, ej. `msg.sender`).
- **Valor (`ValueType`):** Puede ser cualquier tipo, incluidos tipos complejos (structs, arrays u otros mappings anidados). En un banco o token, suele ser un número entero (`uint256`) que representa el saldo de la cuenta en wei.

### Rol en Smart Contracts: El Libro Mayor (*Ledger*)
La blockchain no registra de forma nativa qué porción del saldo del contrato le corresponde a cada usuario individual; sólo custodia el saldo total agregado (`address(this).balance`). Por ello, el desarrollador utiliza un `mapping` para construir y mantener el **libro mayor contable interno** de la aplicación.

---

## 2. Peculiaridades Críticas en la EVM

Los mappings en Solidity tienen comportamientos únicos derivados del diseño de almacenamiento (*storage*) de la máquina virtual de Ethereum:

### A. No existe el concepto de "clave no encontrada" (Valor por defecto)
- En la mayoría de lenguajes de programación tradicionales, consultar una clave inexistente arroja un error (`KeyError`, `null` o `undefined`).

> [!important] Principio Clave de la EVM
> En Solidity, ==**todas las claves posibles existen virtualmente desde el momento del despliegue** y devuelven el valor por defecto de su tipo (`0` para enteros, `false` para booleanos, `0x00...00` para direcciones)==.

- Si una wallet jamás ha interactuado con el contrato y se ejecuta `balances[0xABC...]`, la EVM simplemente retorna `0`.

### B. No son iterables y carecen de tamaño (`length`)
- En un `mapping` no se almacenan las claves de forma secuencial ni existe una lista de claves guardadas; cada valor se almacena directamente en una ranura criptográfica calculada mediante la función de hashing Keccak-256 (`keccak256(h(k) . p)`).
- Por este motivo:
  - No existe una propiedad `.length` o `.size`.
  - No es posible recorrer un mapping con un bucle `for` o `while` de forma directa.
  - Si un caso de uso requiere iterar sobre todos los titulares de cuentas, se debe mantener una estructura auxiliar en paralelo (por ejemplo, un array dinámico `address[] public accounts`).

---

## 3. Operaciones Esenciales

### Lectura de datos
```solidity
uint256 miSaldo = balances[msg.sender];
```

### Acreditación (Depósitos y Aumentos)
```solidity
// ✅ Acumulación correcta al saldo preexistente
balances[msg.sender] += msg.value;
```
> [!warning] Riesgo Contable: Asignación destructiva
> Nunca usar asignación simple (`=`). Hacer `balances[msg.sender] = msg.value;` sobrescribe el saldo, borrando los depósitos previos del usuario.

### Débito (Retiros y Deducciones)
```solidity
// ✅ Deducción correcta tras verificar fondos suficientes
balances[msg.sender] -= amount;
```

---

## 4. Mappings Anidados (Nested Mappings)

Los mappings pueden anidarse para representar relaciones bidimensionales o autorizaciones delegadas (patrón estándar en tokens ERC-20 para aprobaciones y límites de gasto):

```solidity
// propietario => (delegado => monto permitido)
mapping(address => mapping(address => uint256)) public allowance;
```

---

## Notas Relacionadas
- [[Blockchain Accelerator - Skool (Jose)]]
- [[Anexo - Vulnerabilidades Críticas en Smart Contracts]]
