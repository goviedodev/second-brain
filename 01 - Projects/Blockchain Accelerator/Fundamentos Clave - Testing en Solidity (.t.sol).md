---
created: 2026-10-05
tags: [project, blockchain, solidity, fundamentos, testing]
source: Blockchain Accelerator - Skool (Jose)
---

# Fundamentos Clave - Testing en Solidity (`.t.sol`)

Los tests de los contratos se escriben **en Solidity** y los archivos de test deben **terminar en la extensión `.t.sol`** (p. ej. `BancoCrypto.t.sol`). Es la convención de **Foundry** (`forge`): así distingue los tests de los contratos normales (`.sol`) y de los scripts de despliegue (`.s.sol`).

---

## 1. Convención de nombres de archivo

| Extensión | Qué contiene | Carpeta habitual |
|-----------|--------------|------------------|
| `.sol` | Contratos de la aplicación | `src/` |
| **`.t.sol`** | **Tests** | `test/` |
| `.s.sol` | Scripts (deploy, interacciones) | `script/` |

> [!warning] Importante
> Si el archivo de test no termina en `.t.sol`, no se reconoce como test por convención y puede no ejecutarse con `forge test`.

---

## 2. Anatomía básica de un test

```solidity
// test/BancoCrypto.t.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Test} from "forge-std/Test.sol";
import {BancoCrypto} from "../src/BancoCrypto.sol";

contract BancoCryptoTest is Test {
    BancoCrypto banco;
    address owner = address(this);
    address usuarioA = makeAddr("usuarioA");

    // Se ejecuta antes de CADA test: deja un estado limpio.
    function setUp() public {
        banco = new BancoCrypto(10 ether);
        vm.deal(usuarioA, 20 ether); // le damos ETH de prueba
    }

    // Las funciones de test deben empezar con `test`.
    function testDepositoActualizaSaldo() public {
        vm.prank(usuarioA);              // la próxima llamada la hace usuarioA
        banco.deposit{value: 7 ether}();

        assertEq(banco.balances(usuarioA), 7 ether);
    }

    // Verifica que una llamada revierta con el mensaje esperado.
    function testRevierteSiNoEsOwner() public {
        vm.prank(usuarioA);
        vm.expectRevert("No autorizado: solo el dueno");
        banco.setMaxBalance(20 ether);
    }
}
```

Puntos clave:
- El contrato de test **hereda de `Test`** (`forge-std`).
- **`setUp()`** prepara el escenario antes de cada test.
- Las funciones que **empiezan con `test`** se ejecutan automáticamente.
- Los nombres `testFail...` / `test_RevertWhen_...` son variantes de estilo; hoy se prefiere `vm.expectRevert`.

---

## 2.0 Import obligatorio: `forge-std/Test.sol`

Todo archivo de test debe **importar `Test` desde `forge-std/Test.sol`** y el contrato de test debe **heredar** de él:

```solidity
import {Test} from "forge-std/Test.sol";

contract BancoCryptoTest is Test { ... }
```

Ese import es lo que da acceso a:
- Las **aserciones**: `assertEq`, `assertTrue`, `assertGt`, etc.
- Los **cheatcodes** mediante la variable `vm` (`vm.prank`, `vm.deal`, `vm.expectRevert`).
- Utilidades como `makeAddr`, `deal`, y los logs (`console`).

Sin importar y heredar de `Test`, ninguna de esas herramientas existe y el archivo no compila como test.

> [!note] Instalación
> `forge-std` es una librería que viene con un proyecto Foundry (`forge init`) o se agrega con `forge install foundry-rs/forge-std`. Queda en `lib/forge-std/`, y por eso el import usa la ruta `forge-std/Test.sol`.

---

## 2.1 Reglas para que una función sea un test (detalle)

Para que `forge test` ejecute y **muestre el resultado** de una función, debe cumplir:

1. **El nombre empieza con `test`** (p. ej. `testDeposito`, `test_DepositoActualizaSaldo`). Si se llama `deposito()` o `checkSaldo()`, Foundry la trata como función auxiliar y **no la corre como test**.
2. **Visibilidad `public` (o `external`)**. Una función `internal` o `private` nunca se ejecuta como test.
3. **Mutabilidad `view`** cuando el test solo **lee** estado y no lo modifica (según el curso: `public view`).
4. **Usa `assert*`** para verificar el resultado que se está probando (ver sección 4).
5. **Sin parámetros** → test normal. **Con parámetros** → Foundry lo trata como *fuzz test* y le inyecta valores aleatorios.

```solidity
// ✅ Se ejecuta y se reporta: empieza con `test`, es public, solo lee estado
function testOwnerEsElDeployer() public view {
    assertEq(banco.owner(), owner);
}

// ✅ Test que cambia estado (deposita): public SIN `view`
function testDepositoActualizaSaldo() public {
    vm.prank(usuarioA);
    banco.deposit{value: 1 ether}();
    assertEq(banco.balances(usuarioA), 1 ether);
}

// ❌ No es un test: no empieza con `test`
function depositoActualizaSaldo() public { /* ... */ }

// ❌ No es un test: visibilidad internal
function testAlgo() internal { /* ... */ }
```

### ¿Cuándo `view` y cuándo no?

| Tipo de test | Qué hace | Firma |
|--------------|----------|-------|
| Solo lectura | Consulta getters (`owner()`, `maxBalance()`, `balances(x)`) y compara con `assert*` | `public view` |
| Modifica estado | Deposita, retira, usa `vm.prank`, `vm.deal`, `vm.expectRevert` | `public` (sin `view`) |

> [!warning] Matiz
> `view` solo es posible si la función **no escribe estado ni llama cheatcodes que lo alteran** (`vm.prank`, `vm.deal`, `vm.expectRevert`, etc.) ni transacciones (`deposit{value: ...}`). Si el compilador marca error de mutabilidad, quita el `view`: el test se ejecutará y mostrará su resultado igual. Lo imprescindible para que aparezca en el reporte es `test` al inicio + `public`/`external`.

### Cómo se ven los resultados

```bash
forge test -vv
# [PASS] testOwnerEsElDeployer() (gas: 8712)
# [PASS] testDepositoActualizaSaldo() (gas: 52340)
# [FAIL: assertion failed] testRevierteSiNoEsOwner() (gas: 10245)
```

- Cada función `test*` aparece como `[PASS]` o `[FAIL]` con su consumo de gas.
- Más `v` (`-vv`, `-vvv`, `-vvvv`) = más detalle (logs, trazas de llamadas, *stack traces* de los fallos).

---

## 3. Cheatcodes más usados (`vm.*`)

| Cheatcode | Para qué sirve |
|-----------|----------------|
| `vm.prank(addr)` | La **siguiente** llamada se hace como `addr` (`msg.sender`) |
| `vm.startPrank(addr)` / `vm.stopPrank()` | Igual, pero para varias llamadas seguidas |
| `vm.deal(addr, monto)` | Asigna saldo de ETH a una cuenta |
| `vm.expectRevert(...)` | Espera que la siguiente llamada **revierta** |
| `makeAddr("nombre")` | Crea una dirección determinista con etiqueta |

Esto reemplaza el flujo manual de Remix (cambiar de cuenta, poner el valor, pulsar el botón) por pruebas **automáticas y repetibles** ligadas a los escenarios del [[Blockchain Accelerator - Skool (Jose)|Banco Crypto]]: depósito válido, depósito que excede `MAX_BALANCE`, intento no autorizado de `setMaxBalance`, etc.

---

## 3.1 Probar reverts con `vm.expectRevert()`

Para probar que una llamada **debe fallar** (revertir), se usa el cheatcode **`vm.expectRevert()`**. Se escribe **justo antes** de la llamada que se espera que revierta. Si la llamada **no** revierte, el test falla.

```solidity
// Revert sin verificar el motivo
function testRevierteSiNoEsOwner() public {
    vm.prank(usuarioA);
    vm.expectRevert();
    banco.setMaxBalance(20 ether);
}

// Revert verificando el mensaje exacto del require
function testRevierteConMensaje() public {
    vm.prank(usuarioA);
    vm.expectRevert("No autorizado: solo el dueno");
    banco.setMaxBalance(20 ether);
}
```

Variantes según cómo revierte el contrato:

| Caso | Uso |
|------|-----|
| Cualquier revert | `vm.expectRevert();` |
| `require(..., "mensaje")` | `vm.expectRevert("mensaje");` |
| `error MiError();` (custom error) | `vm.expectRevert(MiError.selector);` |
| Custom error con parámetros | `vm.expectRevert(abi.encodeWithSelector(MiError.selector, arg));` |

Reglas importantes:
- **Solo afecta a la siguiente llamada externa** al contrato. Si antes se ponen otras líneas que llaman al contrato, el `expectRevert` se consumirá en la primera.
- Si se usa `vm.prank`, ponerlo **antes** de `vm.expectRevert` (como en el ejemplo) para que la llamada revertida se haga como el usuario deseado.
- Pasar el mensaje o selector es preferible a `vm.expectRevert()` vacío: confirma que revierte **por la razón correcta** y no por otro error.
- Este es el equivalente automático de los pasos de Remix donde el intento no autorizado o el depósito que excede `MAX_BALANCE` salía como "revertido".

---

## 4. Aserciones (`assert`): verificar el resultado de cada test

**Cada función de test debe usar una aserción (`assert*`)** para verificar el resultado que está probando. Sin `assert`, el test solo comprueba que el código **no revierta**: aparecerá como `[PASS]` aunque el resultado sea incorrecto, dándonos una falsa sensación de seguridad.

Patrón: **Preparar → Ejecutar → Verificar (`assert`)**.

```solidity
function testRetiroDejaSaldoEnCero() public {
    // 1. Preparar
    vm.startPrank(usuarioA);
    banco.deposit{value: 5 ether}();

    // 2. Ejecutar
    banco.withdraw(5 ether);
    vm.stopPrank();

    // 3. Verificar: compara el resultado real con el esperado
    assertEq(banco.balances(usuarioA), 0);
}
```

Si la condición no se cumple, el test falla y `forge` lo reporta como `[FAIL]` indicando el valor esperado vs. el obtenido.

| Aserción | Verifica |
|----------|----------|
| `assertEq(a, b)` | `a == b` (el más usado) |
| `assertNotEq(a, b)` | `a != b` |
| `assertTrue(cond)` / `assertFalse(cond)` | La condición es verdadera / falsa |
| `assertGt(a, b)` / `assertGe(a, b)` | `a > b` / `a >= b` |
| `assertLt(a, b)` / `assertLe(a, b)` | `a < b` / `a <= b` |

> [!tip] Convención
> En `assertEq(actual, esperado)` se suele poner primero el valor real (lo que devuelve el contrato) y luego el esperado. Se puede añadir un mensaje opcional: `assertEq(banco.balances(usuarioA), 0, "El saldo debe quedar en cero")`.

Para comprobar que algo **debe fallar**, la "aserción" es `vm.expectRevert(...)` justo antes de la llamada (ver sección 3.1).

---

## 5. Ejecutar los tests

```bash
forge test                          # todos los tests
forge test --match-path test/BancoCrypto.t.sol
forge test --match-test testDeposito
forge test -vvv                     # más detalle (trazas de fallos)
forge test --match-test <nombre-funcion-test> -vvvv   # un solo test con traza completa
forge coverage                      # cobertura
```

### Probar un test específico con traza completa (`-vvvv`)

Para depurar una función concreta, se filtra por su nombre y se sube la verbosidad al máximo:

```bash
forge test --match-test testDepositoActualizaSaldo -vvvv
```

- **`--match-test <nombre-funcion-test>`**: ejecuta solo los tests cuyo nombre coincide (acepta nombre exacto o parte de él, p. ej. `testDeposito` corre todos los que lo contengan).
- **`-vvvv`**: muestra la **traza completa de ejecución**, incluso en tests que pasan.

La traza incluye cada llamada entre contratos (`[Call]`), los argumentos y valores enviados, los valores de retorno, los eventos emitidos (`emit`), los `revert` con su motivo y el gas de cada paso. Sirve para ver paso a paso qué ocurre dentro de `deposit()` o `withdraw()` y localizar dónde falla una aserción o un `revert`.

Niveles de verbosidad:

| Flag | Muestra |
|------|---------|
| (ninguno) | Solo `[PASS]` / `[FAIL]` |
| `-vv` | Logs (`console.log`) |
| `-vvv` | Trazas de los tests que **fallan** |
| `-vvvv` | **Trazas de todos** los tests (incluidos los que pasan) y `setUp` |

> [!tip] Recomendación
> Usa `--match-test` junto con `-vvvv`: con muchos tests, la traza completa de todos a la vez es demasiado ruido.

---

## Relacionado

- [[Fundamentos Clave - Modificadores y el Operador _; en Solidity]] — lógica de `onlyOwner` que conviene testear.
- [[Fundamentos Clave - Mappings en Solidity]] — saldos que se verifican con `assertEq`.
- [[Anexo - Vulnerabilidades Críticas en Smart Contracts]] — cada vulnerabilidad debería tener su test.
