# Operador Lógico AND (&&)

[Volver](./JavaScript.md)

## Tabla de Verdad de AND (&&)

```txt
| A     | B     | A && B | Explicación                       |
| ----- | ----- | ------ | --------------------------------- |
| true  | true  | true   | Ambos son verdaderos.             |
| true  | false | false  | Uno es falso → toda la expresión. |
| false | true  | false  | Uno es falso → toda la expresión. |
| false | false | false  | Ambos son falsos.                 |
```

📗 Ejemplos:

```js
console.log(5 && "OK"); // "OK"
```

- Descripción:
  - JS evalúa 5 → truthy.
  - Evalúa "OK" → truthy.
  - Devuelve el último valor evaluado → "OK".

```js
console.log(0 && "Hola"); // 0
```

- Descripción:
  - El primer valor (0) es falsy.
  - Esto implica que ya no es necesario evaluar el segundo valor, el operador detiene la evaluación (short-circuit) y devuelve 0.
  - El valor de verdad de toda la sentencia es "false".

[Volver](./JavaScript.md)