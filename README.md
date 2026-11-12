# Field Shift

**Adaptar la chacra con datos de la NASA.** Demo para la sierra de Moquegua, Perú: toma 25 años de datos satelitales de [NASA POWER](https://power.larc.nasa.gov), muestra el clima de la campaña (con escenarios El Niño y La Niña), recomienda una rotación de cultivos explicable y hace el backtest de un seguro paramétrico, con una póliza opcional en Avalanche Fuji.

Es la base del workshop y del hackathon del reto **Field Shift: Adapting Farms with NASA Data** del [NASA Space Apps Challenge 2026](https://www.spaceappschallenge.org), con Team1 Avalanche.

> **Contrato educativo, no auditado; solo Fuji y wallets de prueba.** Nunca uses fondos reales.

- Guía del workshop (qué hace cada clic y qué código corre): [`doc/workshop/guia-field-shift.html`](doc/workshop/guia-field-shift.html)
- Especificación y validación (NexusAgil): [`doc/sdd/001-field-shift/`](doc/sdd/001-field-shift/)

---

## Cómo levantarlo

```bash
git clone https://github.com/ferrosasfp/field-shift.git
cd field-shift
npm ci
npm run dev
```

Abre http://localhost:3000/es/field-shift (o `/en/field-shift`). No requiere Supabase ni login: la ruta es pública.

### Requisitos

- Node.js 20 o más reciente.
- [Foundry](https://book.getfoundry.sh) (opcional, solo para el contrato).

### Sin wifi (workshop)

Los datos ya vienen en el repo (`src/features/field-shift/data/*.json`). Si NASA POWER no responde en 8 s, la app usa esa copia y lo indica en pantalla ("copia guardada").

```bash
FIELD_SHIFT_OFFLINE=true npm run dev   # no intenta la red: usa los snapshots directo
npm run field-shift:datos               # con internet: actualiza los snapshots
```

### Tests

```bash
npm run test                                              # Vitest (incluye el golden de paridad con el MVP)
npm run test:e2e -- field-shift                           # Playwright
npm run field-shift:golden -- <ruta-a-field-shift-mvp>    # regenera el golden desde el MVP original
```


### El resto de la app (opcional: login, dashboard, wallet)

Field Shift no necesita Supabase. Si quieres levantar también el login y el dashboard, necesitas Docker Desktop:

```bash
npx supabase start            # Supabase local con las migraciones de supabase/migrations
```

Copia en `.env.local` la `API URL` y la `anon key` que imprime el comando (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`), además de `SUPABASE_SERVICE_ROLE_KEY` y `NEXT_PUBLIC_SITE_URL=http://localhost:3000`. Mira `.env.example` para el resto de variables.

### Contrato (opcional, Avalanche Fuji)

> **Contrato educativo, no auditado; solo Fuji y wallets de prueba**

Ya hay uno desplegado para el workshop en Fuji: [`0x594c5D7Bb4ff20F296c1CfacFE4c5555e278961c`](https://testnet.snowtrace.io/address/0x594c5D7Bb4ff20F296c1CfacFE4c5555e278961c) (zona 0, Carumas). Para usarlo, ponlo en `NEXT_PUBLIC_FIELD_SHIFT_CONTRACT` dentro de `.env.local`. Para desplegar el tuyo:

```bash
npm run contracts:install     # forge-std + OpenZeppelin en contracts/lib
npm run contracts:test
```

1. Crea `contracts/.env` con `PRIVATE_KEY=0x...` **de una wallet de pruebas** (nunca una con fondos reales) y consigue AVAX de prueba en el [faucet de Fuji](https://core.app/tools/testnet-faucet/).
2. Despliega: `npm run contracts:deploy:poliza:fuji`.
3. Copia la dirección que imprime en `NEXT_PUBLIC_FIELD_SHIFT_CONTRACT` (`.env.local`) y reinicia `npm run dev`. Sin esa variable, el bloque de la póliza queda oculto.
4. Conecta tu wallet (Core o MetaMask) en Fuji y compra una póliza.
5. Simula al oráculo publicando el índice (humedad del suelo × 1000; 380 = 0.380):

   ```bash
   CONTRATO=0x... INDICE=380 npm run contracts:reportar:fuji
   ```

   Si el índice queda debajo del umbral (450), presiona **Reclamar pago**.

### Limitaciones del MVP

- Cada dato de NASA POWER cubre una celda de unos 50 km: en Moquegua una celda mezcla pisos ecológicos distintos y zonas cercanas pueden dar el mismo resultado.
- Los valores agronómicos de los cultivos son aproximados: valídalos con un agrónomo local.
- `T2M_MIN` es mensual: confirma en el diccionario de parámetros de NASA POWER si es la mínima absoluta del mes o el promedio de las mínimas diarias, porque cambia cómo se lee la tarjeta de heladas.
- La lluvia de NASA POWER puede diferir de la de las estaciones de Senamhi; compararlas es una buena mejora.

### Ideas para el hackathon

- Agregar vegetación (NDVI de MODIS) para confirmar daño en el cultivo.
- Usar humedad del suelo de SMAP con mejor resolución.
- Datos de suelo reales (SoilGrids) en lugar de reglas generales.
- Un mapa para marcar la parcela.
- Validar cultivos y umbrales con un agrónomo, una agencia agraria o una junta de usuarios de riego.
- Comparar con estaciones de Senamhi.
- Agregar los valles con riego usando la lluvia de la cuenca alta como indicador del agua disponible.
- Conectar el oráculo con Chainlink Functions.
- Usar el pronóstico de ENFEN antes de sembrar para elegir el escenario automáticamente.
- En el contrato: cerrar la venta de pólizas antes de que se conozca el pronóstico de El Niño (si no, la gente compra solo cuando ya sabe que viene seco) y guardar reservas para un año en que todos cobran a la vez.


---

## Stack

Next.js 16 + React 19 + TypeScript · Tailwind · next-intl (ES/EN) · Zod · Vitest + Playwright · Supabase (opcional) · Viem + Wagmi · Foundry + OpenZeppelin · Avalanche Fuji.

El código de Field Shift vive en `src/features/field-shift/` (motor en `services/`, UI en `components/`), la ruta en `src/app/[locale]/field-shift/`, la API en `src/app/api/field-shift/nasa/` y el contrato en `contracts/src/PolizaParametrica.sol`.

## Basado en NexusFactory

Este proyecto se generó desde la plantilla [NexusFactory](https://github.com/ferrosasfp/nexus-factory) (modo hybrid) y trae su metodología NexusAgil en `.claude/`. Para empezar otro proyecto, usa la plantilla, no este repo.

## Créditos

Creado por [Fernando Rosas](https://github.com/ferrosasfp). Datos: NASA POWER (NASA Langley Research Center).

MIT License
