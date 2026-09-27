---
created: 2026-09-27
tags: [project, blockchain, smart-contracts, seguridad]
source: Blockchain Accelerator - Skool (Jose)
---

# Anexo - Vulnerabilidades Críticas en Smart Contracts

En el desarrollo y auditoría de contratos inteligentes en Solidity, ciertos errores arquitectónicos o descuidos en el flujo de ejecución generan vulnerabilidades críticas capaces de drenar fondos o destruir la integridad contable del contrato.

---

## 1. Parámetro de monto desacoplado en depósitos de ETH (`amount` vs. `msg.value`)

- **Nivel de Severidad:** Crítica / Catastrófica
- **Vector de Ataque:** Falsificación de balance e inflación artificial de contabilidad interna.
- **Descripción:**
  Al depositar Ether (o la moneda nativa de la EVM), una función jamás debe recibir el monto a acreditar mediante un parámetro de entrada (p. ej. `uint256 amount`). Si una función `payable` toma un argumento `amount` y actualiza la contabilidad interna usando ese valor en vez de la variable global `msg.value`, un atacante puede desacoplar los fondos reales ingresados de su registro en el contrato.

### Escenario de Explotación
```solidity
// ❌ VULNERABLE: Acepta parámetro del monto a depositar
function depositVulnerable(uint256 amount) external payable {
    // El atacante envía msg.value = 1 wei, pero pasa amount = 1000 ether
    balances[msg.sender] += amount;
}
```
1. El atacante llama a `depositVulnerable(1000 ether)` enviando apenas `1 wei` de ETH real en `msg.value`.
2. El contrato registra que el atacante tiene `1000 ether` en su mapping `balances`.
3. Posteriormente, el atacante invoca `withdraw(1000 ether)` y drena los fondos reales depositados legítimamente por los demás usuarios.

### Regla de Oro / Mitigación
- **Fuente de verdad única:** El monto real de ETH transferido en la transacción reside exclusivamente en `msg.value`.
- **Sin parámetros de monto:** La función de depósito no debe aceptar ningún parámetro de monto (`amount`).
- **Acumulación estricta (`+=` vs. `=`):** Jamás olvidar el operador de suma acumulativa `+= msg.value`. Usar una asignación simple (`= msg.value`) es una vulnerabilidad crítica de pérdida/reinicio contable:
  - Si un usuario tiene `5 ETH` depositados y luego deposita `1 ETH`, con `balances[msg.sender] = msg.value;` su balance se sobrescribe a `1 ETH`, destruyendo sus `5 ETH` previos de la contabilidad interna y dejándolos bloqueados para siempre en el contrato.
  - De igual forma en el retiro, debe descontarse con resta acumulativa (`-= amount`), nunca sobreescribir.

```solidity
// ❌ VULNERABLE: Asignación destructiva (= en lugar de +=)
// Sobrescribe el historial y destruye los depósitos anteriores del usuario
balances[msg.sender] = msg.value;

// ✅ SEGURO: Acumulación contable correcta y exclusiva desde msg.value
function deposit() external payable {
    require(msg.value > 0, "Debe depositar mas de 0 ETH");
    balances[msg.sender] += msg.value;
}
```


---

## 2. Ataque de Reentrancia (*Reentrancy Attack*)

- **Nivel de Severidad:** Crítica (responsable del hack histórico de *The DAO* en 2016).
- **Vector de Ataque:** Secuestro del flujo de ejecución mediante transferencias externas previas a la actualización de estado.
- **Descripción:**
  Ocurre cuando un contrato inteligente transfiere Ether a una dirección externa antes de actualizar su registro contable interno. Si el receptor es un contrato malicioso, la transferencia de ETH activa automáticamente su función `receive()` o `fallback()`. Desde allí, el atacante vuelve a invocar `withdraw` de forma recursiva antes de que su balance original haya sido descontado.

### Escenario de Explotación
```solidity
// ❌ VULNERABLE: Interacción externa antes de actualizar el balance
function withdrawVulnerable(uint256 amount) external {
    require(balances[msg.sender] >= amount, "Saldo insuficiente");

    // 1. Interacción externa primero: el atacante retoma el control del hilo de ejecución
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Fallo al enviar ETH");

    // 2. Efecto tardío: jamás se alcanza hasta vaciar el contrato
    balances[msg.sender] -= amount;
}
```
1. El atacante deposita `1 ETH`.
2. Llama a `withdraw(1 ETH)`.
3. El contrato víctima envía `1 ETH` llamando al contrato del atacante.
4. El `receive()` del atacante toma el control y, como `balances[msg.sender]` aún no fue reducido, vuelve a llamar a `withdraw(1 ETH)`.
5. El ciclo recursivo vacía la totalidad de fondos custodiados por el contrato víctima.

### Regla de Oro / Mitigación
1. **Patrón Checks-Effects-Interactions (CEI):** Actualizar siempre las variables de estado internas (`balances[msg.sender] -= amount;`) **antes** de cualquier llamada o transferencia externa (`call`).
2. **Mutex / Bloqueo de Reentrancia:** Utilizar modificadores como `nonReentrant` de librerías auditadas como OpenZeppelin.

---

## Notas Relacionadas
- [[Blockchain Accelerator - Skool (Jose)]]
- [[Fundamentos Clave - Mappings en Solidity]]


