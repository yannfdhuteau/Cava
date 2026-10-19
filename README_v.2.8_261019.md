# 🍷 Cava

Inventario personal de vinos para el celular. Lleva qué botellas hay, dónde están guardadas, cuáles se descorcharon y qué tal estuvieron. Dos personas pueden usarla al mismo tiempo desde sus teléfonos y ver el mismo stock.

Es un único archivo HTML, sin build ni dependencias. Se publica con GitHub Pages y guarda los datos en [Supabase](https://supabase.com).

---

## Funcionalidades

**Stock**
- Vinos agrupados en secciones que se pueden colapsar. Se ordenan por color, tipo (varietal / blend / espumante), año, país o región.
- Indicador visual de botellas por vino.
- *Descorchar* (−1, con opción de deshacer) y *+ Agregar* (+1) con un toque.
- Buscador que ignora acentos: busca por bodega, línea, variedad, región, país y ubicación.
- Filtros por color, tipo, variedad, país, región y año. Los vinos sin dato aparecen como "Sin especificar".

**Ficha de cada vino**
- Bodega, línea, año, color, variedades con porcentaje, región y notas.
- País en texto libre: reconoce nombres y alias (EEUU, UK, Holanda…) y muestra la bandera.
- Ubicación física (por ejemplo, "Caja 3").
- Foto de la etiqueta, desde la cámara o la galería. Se comprime en el teléfono antes de subirla y se puede ver a pantalla completa.

**Historial**
- *Catálogo*: todos los vinos cargados alguna vez, incluso los agotados, para reponer sin volver a cargarlos.
- *Registro*: línea de tiempo de cada botella agregada o descorchada. Cada descorche puede tener una calificación del 1 al 10 y una nota libre, que se pueden editar más adelante.

**App**
- Instalable en la pantalla de inicio de iPhone (PWA), con ícono propio.
- Se sincroniza entre dispositivos cada 25 segundos y al volver a abrir la app.
- El número de versión se ve en la propia app.

---

## Cómo funciona

| Pieza | Detalle |
|---|---|
| Frontend | `cava.html`: HTML + CSS + JavaScript en un solo archivo, sin frameworks |
| Backend | Supabase (PostgreSQL), accedido por `fetch` directo a la API REST, sin `supabase-js` |
| Hosting | GitHub Pages, rama `main` |
| Sincronización | Polling cada 25 s mientras la app está visible, con escrituras optimistas |
| Keepalive | GitHub Action que consulta la base cada 3 días para que el plan gratuito de Supabase no la pause |

### Seguridad: tenelo en cuenta

La app **no tiene login**. Usa la clave pública (*anon*) de Supabase, que queda embebida en el HTML, con políticas RLS abiertas. Cualquiera que tenga el link de la app puede ver y modificar los datos. Para un uso personal o familiar está bien; no la uses para nada sensible.

---

## Instalación desde cero

### 1. Base de datos en Supabase

Creá un proyecto en Supabase y corré esto en **SQL Editor → New query**:

```sql
create table public.vinos (
  id text primary key,
  bodega text not null,
  linea text,
  anio integer,
  color text not null,
  variedades jsonb not null default '[]'::jsonb,
  region text,
  pais text,
  ubicacion text,
  notas text,
  foto text,
  stock integer not null default 0,
  created_at timestamptz not null default now()
);

alter table public.vinos enable row level security;
create policy "vinos abierto" on public.vinos for all using (true) with check (true);

create table public.eventos (
  id text primary key,
  wine_id text not null references public.vinos(id) on delete cascade,
  tipo text not null,
  cantidad integer not null,
  fecha timestamptz not null default now(),
  snapshot jsonb not null default '{}'::jsonb,
  rating integer check (rating is null or (rating between 1 and 10)),
  nota text
);

alter table public.eventos enable row level security;
create policy "eventos abierto" on public.eventos for all using (true) with check (true);
```

### 2. Configurar la app

En `cava.html`, cerca del inicio del `<script>`, reemplazá estos dos valores por los de tu proyecto (**Project Settings → API**):

```js
var SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
var SUPABASE_KEY = 'TU-CLAVE-ANON';   // la anon/publishable, nunca la service_role
```

### 3. Publicar

1. Subí `cava.html` a este repositorio.
2. **Settings → Pages**: *Deploy from a branch*, rama `main`, carpeta `/ (root)`.
3. La app queda en `https://USUARIO.github.io/Cava/cava.html`.
4. En el iPhone: abrila en Safari → Compartir → **Agregar a pantalla de inicio**.

### 4. Keepalive (opcional, recomendado en el plan gratuito)

Supabase pausa los proyectos gratuitos después de 7 días sin actividad. El workflow `.github/workflows/supabase-keepalive.yml` lo evita:

1. **Settings → Secrets and variables → Actions → New repository secret**, con nombre `SUPABASE_ANON_KEY` y como valor la clave anon.
2. **Actions → Supabase keepalive → Run workflow** para probarlo. Tiene que terminar en verde.

GitHub desactiva los workflows programados después de 60 días sin commits en el repo. Se reactivan desde la pestaña Actions.

---

## Actualizar la app

1. **Add file → Upload files** y subí el `cava.html` nuevo, con el mismo nombre que el actual.
2. **Commit changes**.
3. Esperá uno o dos minutos y abrí la app: el número de versión tiene que coincidir.

Si una versión nueva agrega columnas, corré primero el SQL correspondiente en Supabase y después subí el HTML.

---

## Estructura del repositorio

```
├── cava.html                              # la app completa
├── README.md
└── .github/workflows/
    └── supabase-keepalive.yml             # ping cada 3 días a Supabase
```

---

## Versiones

| Versión | Cambios principales |
|---|---|
| v1.0 | Versión original, con datos guardados solo en el dispositivo |
| v2.0 | Supabase: datos compartidos entre teléfonos y migración automática de los datos locales |
| v2.1 | Ícono propio |
| v2.2 | Número de versión visible |
| v2.3 | Ubicación física, y calificación y nota por descorche |
| v2.4 | Foto de etiqueta a pantalla completa |
| v2.5 | País y filtros |
| v2.6 | Espumante en el filtro Tipo y foto desde la galería |
| v2.7 | "Ordenar por" con secciones colapsables |
| v2.8 | País en texto libre con reconocimiento y bandera |
