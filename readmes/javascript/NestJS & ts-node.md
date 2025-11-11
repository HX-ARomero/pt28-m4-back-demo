# NestJS & ts-node

- En NestJS, cuando se ejecuta el comando: `npm run start:dev`
- …lo que realmente hace es ejecutar: `nest start --watch`
- Esto no ejecuta nest build explícitamente, pero sí compila el código TypeScript a JavaScript en la carpeta dist automáticamente cada vez que:
  - el servidor se inicia, o
  - detecta un cambio en los archivos .ts del proyecto.

## 🔍 Cómo funciona internamente

- El modo `--watch usa tsc-watch` (a través del CLI de Nest) para:
  - Compilar los archivos TypeScript a la carpeta dist usando tu configuración de tsconfig.json.
  - Volver a compilar automáticamente cada vez que detecta un cambio en src/\*\*.
- Por eso la carpeta dist:
  - Se crea la primera vez que levantás el servidor, y
  - Se actualiza automáticamente cada vez que modificás código fuente.
- Nest no ejecuta un nest build completo cada vez, sino que recompila solo los archivos que cambian, gracias a la opción "incremental": true en tu tsconfig.json.

## ⚙️ El rol de "deleteOutDir": true

- En el archivo nest-cli.json tenemos: `"deleteOutDir": true`
  - Esto significa que NestJS borra la carpeta dist antes de cada compilación completa, pero solo cuando se ejecuta nest build, no en modo --watch.
  - Así que con `start:dev`, la carpeta dist no se borra completa cada vez, sino que se van sobrescribiendo los archivos modificados.

# 🧠 En resumen

| Acción              | ¿Compila a `dist`? | ¿Borra `dist` antes?   | ¿Recompila en cambios?   |
| ------------------- | ------------------ | ---------------------- | ------------------------ |
| `npm run build`     | ✅ Sí              | ✅ Sí (`deleteOutDir`) | ❌ No                    |
| `npm run start`     | ✅ Sí              | ❌ No                  | ❌ No                    |
| `npm run start:dev` | ✅ Sí              | ❌ No                  | ✅ Sí (solo los cambios) |

# ⚙️ ¿NestJS usa dist o ts-node al ejecutar npm run start:dev?

- Por defecto, NestJS compila tu código TypeScript a JavaScript dentro de la carpeta dist (usando tsc) y ejecuta ese código compilado con Node.js.
- 👉 O sea, no usa ts-node.
- Entonces:
  - En modo start o start:dev, Nest:
  - Compila los .ts a .js en /dist
  - Ejecuta node dist/main.js
- Solo si se configura explícitamente lo contrario, Nest puede usar ts-node para ejecutar directamente los archivos .ts sin generar dist.

# Cómo y cuándo cambió 👇

- En las versiones antiguas de NestJS, el comportamiento por defecto era ejecutar el código directamente con ts-node, sin generar la carpeta dist.

## 🕰️ Historia del cambio

### 🧩 Antes (NestJS v6 y v7 — hasta ~2020)

- El comando: `npm run start:dev`
- Ejecutaba internamente:
  - ts-node -r tsconfig-paths/register src/main.ts
- Es decir: Nest usaba ts-node directamente para levantar el proyecto en modo desarrollo.
- Por eso, no se creaba la carpeta dist durante start:dev.
- En esa época, el CLI usaba internamente ts-node y tsconfig-paths para manejar imports y decoradores.
- 📦 Ejemplo típico de package.json (NestJS 6.x o 7.x):

```bash
"scripts": {
"start": "ts-node -r tsconfig-paths/register src/main.ts",
"start:dev": "nodemon --watch 'src/\*_/_.ts' --exec 'ts-node -r tsconfig-paths/register src/main.ts'",
"start:prod": "node dist/main.js"
}
```

### ⚙️ Después (NestJS v8 en adelante — desde ~2021)

- A partir de NestJS v8, el CLI fue reescrito parcialmente y adoptó un nuevo comportamiento:
- Se introdujo una pipeline de compilación interna con tsc.
- Ahora nest start y nest start:dev usan el compilador de TypeScript (no ts-node) para generar los archivos en dist.
- Esto permitió:
  - Mejor rendimiento en proyectos grandes.
  - Soporte consistente para --watch, --debug, y --webpack.
  - Integración más limpia con herramientas como @nestjs/cli y @nestjs/schematics.
- Así que desde NestJS 8.0.0,
  - 👉 start:dev compila automáticamente a /dist y ejecuta node dist/main.js, dejando atrás el uso de ts-node por defecto.

## 🧭 En resumen

- Versión de NestJS Herramienta usada en start:dev Generaba /dist Ejemplo de ejecución
  - v6 – v7 ts-node ❌ ts-node src/main.ts
  - v8 – v11 (actual) tsc (CLI interno) ✅ node dist/main.js
- 📘 Extra: ¿por qué lo cambiaron?
  - El cambio fue intencional:
  - ts-node es más cómodo, pero más lento en grandes proyectos.
  - Compilar con tsc permite usar los archivos ya compilados para debugging, testing y despliegues rápidos.
  - Además, reduce inconsistencias con los decoradores y los metadatos de TypeScript (muy usados en Nest).
