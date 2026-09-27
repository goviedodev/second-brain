---
created: 2026-09-25
deadline: 2026-12-25
tags: [project, blockchain, educacion]
source: Skool - Jose
---

# Blockchain Accelerator

> [!note] Contexto del Proyecto
> Programa de formación y construcción técnica **Blockchain Accelerator** en Skool, impartido por Jose.
> - **Fecha límite / Entrega:** 25 de diciembre de 2026 (Navidad).
> - **Estado:** Activo.

## Ideas y conceptos clave

## Módulos y temas

### Módulo: Banco Crypto Básico (Depósito y Retiro de ETH)

- **Objetivo:** Construir un contrato inteligente elemental que actúe como una bóveda/banco descentralizado, permitiendo a los usuarios depositar Ether (ETH), llevar el registro de sus saldos y retirar sus fondos cuando lo deseen.

#### Conceptos y Mecánicas Clave
1. **Recepción de Fondos (`payable`):**
   - Para que una función acepte ETH nativo en Solidity, debe marcarse explícitamente con la palabra clave `payable`.
   - El monto enviado con la transacción se lee mediante la variable global `msg.value`.
   - La dirección que envía los fondos es `msg.sender`.

2. **Registro Contable (Ledger interno):**
   - La blockchain no sabe automáticamente qué parte del saldo del contrato pertenece a quién, por lo que el contrato debe gestionar su propia contabilidad interna.
   - Se utiliza una estructura de datos tipo clave-valor: `mapping(address => uint256) public balances;`.

3. **Retiro Seguro y Patrón Checks-Effects-Interactions (CEI):**
   - **Checks (Comprobaciones):** Verificar que el usuario tenga saldo suficiente (`require(balances[msg.sender] >= amount, "Saldo insuficiente");`).
   - **Effects (Efectos internos):** Descontar el balance en el mapping **antes** de transferir (`balances[msg.sender] -= amount;`).
   - **Interactions (Interacciones externas):** Enviar el ETH al usuario usando la sintaxis recomendada: `(bool success, ) = msg.sender.call{value: amount}(""); require(success, "Fallo en la transferencia");`.
   - > [!warning] Seguridad: Prevención de Reentrancy
   - > Si se transfieren los fondos antes de actualizar el balance, una cuenta de contrato maliciosa podría llamar recursivamente a `withdraw` y drenar todo el banco (ataque de reentrancy).

4. **Eventos (Logs en Blockchain):**
   - Emisión de eventos para que las dApps/frontends indexen depósitos y retiros en tiempo real (`emit Deposit(...)`, `emit Withdraw(...)`).

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BasicCryptoBank {
    mapping(address => uint256) public balances;

    event Deposit(address indexed account, uint256 amount);
    event Withdraw(address indexed account, uint256 amount);

    // Depositar ETH en el banco
    function deposit() external payable {
        require(msg.value > 0, "Debe depositar mas de 0 ETH");
        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }

    // Retirar ETH del banco
    function withdraw(uint256 amount) external {
        require(amount > 0, "Monto invalido");
        require(balances[msg.sender] >= amount, "Saldo insuficiente");

        // 1. Effects: descontar saldo antes de transferir
        balances[msg.sender] -= amount;

        // 2. Interactions: enviar ETH de forma segura
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Fallo al enviar ETH");

        emit Withdraw(msg.sender, amount);
    }

    // Consultar el saldo total custodiado por el contrato
    function getContractBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
```

#### Caso Práctico del Ejercicio: Saldo Agregado vs. Aislamiento Contable

El contrato opera como un **pool o fondo común** a nivel de fondos reales (`address(this).balance`), pero mantiene **estricta separación contable individual**:

- **Fondos globales agregados:** El balance real del contrato es la suma de todos los depósitos menos los retiros.
- **Aislamiento por usuario:** Cada usuario (`msg.sender`) sólo tiene potestad para retirar lo que él mismo depositó; los balances jamás se mezclan.

##### Ejemplo con Usuario A y Usuario B:
1. **Estado Inicial:**
   - Balance del Contrato: `0 ETH`.
2. **Depósito Usuario A (`0xAAA...`):**
   - Ingresa: `7 ETH`.
   - `balances[0xAAA...]` = `7 ETH`.
   - Saldo global del banco: `7 ETH`.
3. **Depósito Usuario B (`0xBBB...`):**
   - Ingresa: `5 ETH`.
   - `balances[0xBBB...]` = `5 ETH`.
   - Saldo global del banco (agregado): `12 ETH`.
4. **Regla de Retiro (Límites individuales):**
   - El Usuario A sólo puede retirar como máximo `7 ETH`.
   - El Usuario B sólo puede retirar como máximo `5 ETH`.
   - Si el Usuario B intenta retirar `6 ETH`, la transacción revierte con `"Saldo insuficiente"`, aunque el banco tenga `12 ETH` en total en su bóveda común.
   - Si el Usuario A retira sus `7 ETH`:
     - Su saldo contable pasa a `0 ETH`.
     - El banco retiene `5 ETH` (que pertenecen íntegramente al Usuario B).

#### Flujo de Prueba y Validación en Remix IDE

- **Entorno:** Remix VM (Cancun / Shanghai / London).
- **Paso 1: Deploy:**
  - Desplegar `BasicCryptoBank` con cualquier cuenta de prueba en Remix.
- **Paso 2: Depósito Usuario A:**
  - En el selector **ACCOUNT**, seleccionar la primera cuenta (Usuario A).
  - En el campo **VALUE**, ingresar `7` y seleccionar la unidad `Ether`.
  - Presionar el botón naranja `deposit`.
- **Paso 3: Depósito Usuario B:**
  - Cambiar en **ACCOUNT** a la segunda cuenta (Usuario B).
  - En el campo **VALUE**, ingresar `5` y unidad `Ether`.
  - Presionar `deposit`.
- **Paso 4: Verificación Contable:**
  - Consultar `balances` pegando la dirección de A $\rightarrow$ muestra `7000000000000000000` wei ($7 \text{ ETH}$).
  - Consultar `balances` pegando la dirección de B $\rightarrow$ muestra `5000000000000000000` wei ($5 \text{ ETH}$).
  - Llamar a `getContractBalance` $\rightarrow$ muestra `12000000000000000000` wei ($12 \text{ ETH}$).
- **Paso 5: Intento de Retiro Ilegal:**
  - Manteniendo la cuenta de B activa, llamar a `withdraw` con `6000000000000000000` ($6 \text{ ETH}$).
  - Verificar en la consola de Remix que la transacción es revertida: `"Saldo insuficiente"`.

#### Extensión: Límite Máximo del Banco (`MAX_BALANCE`) y Control de Dueño (`owner`)

Para controlar la exposición al riesgo y el tamaño del pool, se incorpora un tope global de fondos (`maxBalance` / `MAX_BALANCE`) junto con un esquema de gobernanza básica (Owner pattern).

##### Principios de Diseño:
1. **Patrón de Propietario (`owner`):**
   - Variable `address public owner;` establecida una sola vez al desplegar en el `constructor()`.
   - Modificador `modifier onlyOwner()`: encapsula la regla de autorización para evitar duplicar código.
2. **Capacidad Máxima Dinámica (`maxBalance`):**
   - Límite editable únicamente por el `owner` mediante `setMaxBalance(uint256 _newMaxBalance)`.
3. **Control en Tiempo de Depósito:**
   - En una función `payable`, `address(this).balance` ya incluye el monto recién enviado (`msg.value`).
   - Por tanto, la condición de parada es: `require(address(this).balance <= maxBalance, "Excede el balance maximo permitido");`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CryptoBankWithLimit {
    address public owner;
    uint256 public maxBalance; // Tope total que puede custodiar el banco

    mapping(address => uint256) public balances;

    event Deposit(address indexed account, uint256 amount);
    event Withdraw(address indexed account, uint256 amount);
    event MaxBalanceUpdated(uint256 oldLimit, uint256 newLimit);

    modifier onlyOwner() {
        require(msg.sender == owner, "No autorizado: solo el dueno");
        _;
    }

    constructor(uint256 _initialMaxBalance) {
        owner = msg.sender;
        maxBalance = _initialMaxBalance;
    }

    // Solo el dueno puede modificar el limite maximo
    function setMaxBalance(uint256 _newMaxBalance) external onlyOwner {
        emit MaxBalanceUpdated(maxBalance, _newMaxBalance);
        maxBalance = _newMaxBalance;
    }

    function deposit() external payable {
        require(msg.value > 0, "Debe depositar mas de 0 ETH");
        // address(this).balance ya contiene msg.value en este punto
        require(address(this).balance <= maxBalance, "Deposito excede el balance maximo del banco");

        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) external {
        require(amount > 0, "Monto invalido");
        require(balances[msg.sender] >= amount, "Saldo insuficiente");

        balances[msg.sender] -= amount;

        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Fallo al enviar ETH");

        emit Withdraw(msg.sender, amount);
    }

    function getContractBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
```

##### Flujo de Prueba de Límites y Permisos en Remix:
1. **Deploy con límite:** Cuenta 1 (Deployer / Owner) despliega pasando `_initialMaxBalance = 10000000000000000000` (10 ETH).
2. **Depósito válido:** Cuenta 2 (Usuario A) deposita `7 ETH` $\rightarrow$ Aprobado (balance = 7 ETH $\le$ 10 ETH).
3. **Depósito que excede el tope:** Cuenta 3 (Usuario B) intenta depositar `5 ETH` $\rightarrow$ Revertido con `"Deposito excede el balance maximo del banco"` (7 + 5 = 12 ETH > 10 ETH).
4. **Intento de alteración no autorizada:** Cuenta 2 (Usuario A) intenta llamar a `setMaxBalance(20 ETH)` $\rightarrow$ Revertido con `"No autorizado: solo el dueno"`.
5. **Ajuste legítimo:** Cuenta 1 (Owner) llama a `setMaxBalance(15 ETH)` $\rightarrow$ Transacción confirmada.
6. **Reintento de depósito:** Cuenta 3 (Usuario B) deposita sus `5 ETH` $\rightarrow$ Ahora se ejecuta exitosamente.

## Apuntes y reflexiones
- **Diferencia entre saldo real y contabilidad lógica:** La EVM no distingue a quién pertenece el ETH dentro de la cuenta del contrato; es responsabilidad del programador asegurar que la lógica del mapping refleje fielmente la propiedad de los fondos.
- **Herramienta activa:** Se utiliza **Remix IDE** exclusivamente para compilación, despliegue y simulación multi-cuenta en local.
- **Modificadores (`modifier`):** Permiten interceptar la ejecución de una función antes de su cuerpo (con `_;`), esencial para el control de acceso (`onlyOwner`).
- **Timing del balance en `payable`:** En Solidity, al entrar a una función `payable`, el ETH de la transacción ya se acreditó al contrato; por eso la validación es `address(this).balance <= maxBalance`.

## Recursos y enlaces
- [Remix Ethereum IDE](https://remix.ethereum.org/)

## Anexo: Vulnerabilidades Críticas en Smart Contracts

- Documento de referencia y detalle técnico: [[Anexo - Vulnerabilidades Críticas en Smart Contracts]]

## Próximos pasos / Acciones





