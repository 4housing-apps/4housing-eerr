# 4housing-eerr — Contexto del proyecto

App interna **Gestión · EERR** de 4housing (estado de resultados, presupuesto y
proyectos, con datos de Tango). Parte del portal unificado 4housing.

## Stack

- **Frontend:** un solo archivo `index.html` (HTML/CSS/JS vanilla). Trae embebida una
  librería (SheetJS/xlsx) para leer Excel — ojo al editar, es un bloque minificado grande.
  Acceso a datos por REST (`fetch` a `SB_REST` + helper `sbFetchAllPublic`).
- **Hosting:** GitHub Pages, org `4housing`, repo `4housing-eerr`.
  URL: https://4housing.github.io/4housing-eerr/ (el repo se transfirió desde
  `4housing-apps`; el `REDIRECT_URL` del login apunta a este dominio y debe estar en
  las Redirect URLs de Supabase Auth).
- **Backend:** Supabase **unificado** del portal → proyecto `wcpkpwxhqdcdljfwzcmy` (wcpk).
  Login **Microsoft (Azure)**. El ref/URL del proyecto es lo que usa la app.

## Datos y acceso (migración sept 2026 desde el proyecto viejo aaqp)

- La app guarda su estado en **`eerr_estado`** (clave/valor JSON) — es su tabla propia.
- Además **lee** 4 tablas de referencia de Compras: `compras_usd_diario`,
  `compras_usd_diario_oficial`, `compras_idx_facpce`, `compras_clasif_renglon`.
- **Acceso = dirección.** EERR lo ven quienes tienen `es_direccion=true` (hoy Pablo y
  Micaela), que ven todos los sectores. `eerr_estado` está gateada con
  `tiene_sector('eerr')`; no hay filas de sector `eerr` porque lo cubre dirección.
- Hay una compuerta al iniciar: si el usuario no puede leer `eerr_estado` (no tiene el
  sector), muestra pantalla "sin acceso".

## Reglas de trabajo — NO NEGOCIABLES

1. **No romper lo que ya funciona.** Preferí agregar antes que modificar. Si tocás
   código compartido, `grep` de todos los usos primero. Probá lo que tocaste, no solo
   lo que agregaste.
2. **SQL nunca se ejecuta solo.** Se entrega como `.sql` y lo corre una persona a mano
   en el SQL Editor de Supabase. Sin escritura directa a producción salvo autorización puntual.
3. **Orden de deploy:** primero el SQL (si agrega tablas/columnas), después el HTML.
4. **RLS siempre `authenticated`, nunca `anon`.** Toda tabla nueva con compuerta de sector.
5. **Secretos nunca en `index.html`** (es público). anon/publishable es pública por
   diseño; tokens y service keys, jamás.
6. **Validá el JS con `node --check`** antes de terminar (valida sintaxis, no
   comportamiento: si tocaste algo existente, verificá que siga andando).
7. **Cambios incrementales y aditivos:** una feature por PR, chico y reversible.
8. **Decisiones estructurales se cierran antes de codear.**
9. **Git:** `git pull` antes de empezar; ramas por feature + PR o coordinar antes de
   tocar el `index.html`; commits chicos, descriptivos, en español. Tras `stash pop`/merge,
   chequeá que no queden marcadores de conflicto (`<<<<<<<`) antes de commitear.
10. **Nombres de tablas por sector:** `eerr_` acá (`eerr_estado`); el resto
    `fhcomercial_`, `labocomercial_`, `diseno_`, `planificacion_`, `compras_`, `logistica_`.
    `core_` reservado.
11. **Datos de negocio nunca al repo público** (dumps `.sql` gitignoreados).
12. **Diagnosticar con evidencia** (`grep`/`diff`), no adivinar. Reportar resultados
    con fidelidad: si algo falla o se saltó, decirlo con la salida real.

## Cómo entregar

- HTML/JS: archivo completo actualizado, validado con `node --check`.
- SQL: archivo `.sql` separado; nunca ejecutado contra producción sin que lo revise y
  corra una persona.
