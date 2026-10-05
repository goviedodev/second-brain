---
created: 2026-09-27
tags: [project, blockchain, solidity, fundamentos]
source: Blockchain Accelerator - Skool (Jose)
---

# Fundamentos Clave - Modificadores y el Operador `_;` en Solidity

En Solidity, los **`modifier`** (modificadores de función) permiten alterar, condicionar o extender el comportamiento de las funciones de forma declarativa y reutilizable. El símbolo **`_;`** (*merge point* o *placeholder*) es la pieza central de esta mecánica.

---

## 1. ¿Qué es y qué significa `_;`?

`_;` representa **el punto exacto donde se insertará y ejecutará el cuerpo de la función original** a la que se le aplica el modificador.

Funciona como un marcador de posición que le indica al compilador de Solidity:
> *"Inserta aquí el código de la función modificada cuando las condiciones previas se cumplan"*.

Si un modificador no incluye la sentencia `_;`, el cuerpo de la función llamada **nunca llegará a ejecutarse**.

---

## 2. Caso de Uso Clásico: Control de Acceso (`onlyOwner`)

El patrón más extendido es verificar autorizaciones antes de dar paso a la función:

```solidity
address public owner;

modifier onlyOwner() {
    require(msg.sender == owner, "No autorizado: solo el dueno");
    _; // 👈 Si el require pasa, aquí se ejecuta la función
}

// La función adopta la regla usando el identificador del modificador:
function setMaxBalance(uint256 _newMax) external onlyOwner {
    maxBalance = _newMax; // Este código corre exactamente en el lugar de `_;`
}
```

### Flujo de Ejecución Paso a Paso:
1. Una cuenta externa invoca `setMaxBalance(10 ether)`.
2. El flujo salta primero al cuerpo de `onlyOwner()`.
3. Se evalúa `require(msg.sender == owner, ...)`:
   - **Falso:** La transacción se revierte de inmediato consumiendo el gas correspondiente; la función no se ejecuta.
   - **Verdadero:** La ejecución continúa y llega a la línea **`_;`**.
4. En **`_;`**, se delega el control al cuerpo de `setMaxBalance` y se actualiza `maxBalance`.

---

## 3. La Posición de `_;` Determina el Flujo Temporal

La ubicación física de `_;` dentro del modificador define el momento exacto en el que corre el código respecto a la función:

### A. Pre-condición (Validación previa - Más común)
El código antes de `_;` corre **antes** de la función:
```solidity
modifier preCondicion() {
    require(...); // 1. Valida primero
    _;            // 2. Ejecuta la función después
}
```

### B. Post-ejecución (Limpieza, auditoría o eventos)
El código después de `_;` corre **al finalizar** la función original:
```solidity
modifier postEjecucion() {
    _;                        // 1. Ejecuta primero la función
    emit AccionCompletada();  // 2. Ejecuta este código al terminar
}
```

### C. Envolviendo el Código (*Sandwich Pattern* / Mutex para Reentrancia)
Colocar código antes y después de `_;` permite crear bloqueos (*locks*), base de mecanismos de defensa como `ReentrancyGuard`:
```solidity
bool private locked;

modifier nonReentrant() {
    require(!locked, "Reentrancy bloqueada");
    locked = true;  // 1. Cierra el candado antes

    _;              // 2. Ejecuta la función (retiros, transferencias)

    locked = false; // 3. Abre el candado al terminar
}
```

---

## 4. Resumen Mental

- Piensa en `_;` como el botón de **"continúa con el cuerpo de la función"**.
- Permite reutilizar lógica de validación sin duplicar `require` en múltiples funciones.
- Si hay múltiples modificadores en una función, se ejecutan en el orden en que fueron declarados en la firma.

---

## Notas Relacionadas
- [[Blockchain Accelerator - Skool (Jose)]]
- [[Anexo - Vulnerabilidades Críticas en Smart Contracts]]
- [[Fundamentos Clave - Mappings en Solidity]]
