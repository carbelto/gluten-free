[README.md](https://github.com/user-attachments/files/33420121/README.md)
# SinGluten Mapa

Web estática y 100 % gratuita para encontrar sitios sin gluten por ciudad y ordenarlos por valoraciones de usuarios.

- **Lugares**: OpenStreetMap (etiqueta `diet:gluten_free=yes|only`) vía Overpass API.
- **Ciudad → zona**: Nominatim.
- **Mapa**: Leaflet + teselas de OSM.
- **Valoraciones**: Supabase (plan gratuito). Sin configurarlo funciona en "modo prueba" (se guardan solo en tu navegador).

No hay servidor ni build: todo es el archivo `index.html`.

## 1. Probar en local

Abre `index.html` en el navegador (o `python3 -m http.server` en esta carpeta y ve a http://localhost:8000).

## 2. Valoraciones compartidas con Supabase (gratis)

1. Crea un proyecto en https://supabase.com (plan Free).
2. En **SQL Editor**, ejecuta:

```sql
create table public.ratings (
  id bigint generated always as identity primary key,
  osm_id text not null,                       -- p. ej. "node/123456"
  stars smallint not null check (stars between 1 and 5),
  comment text check (char_length(comment) <= 500),
  created_at timestamptz not null default now()
);

create index ratings_osm_id_idx on public.ratings (osm_id);

alter table public.ratings enable row level security;

-- Cualquiera puede leer y añadir valoraciones; nadie puede editar ni borrar desde la web.
create policy "lectura publica" on public.ratings for select using (true);
create policy "alta publica" on public.ratings for insert with check (true);
```

3. En **Project Settings → API** copia la *Project URL* y la clave **anon public**.
4. Pégalas en `CONFIG` al principio del script de `index.html`.

La clave `anon` es pública por diseño; la seguridad la dan las políticas RLS de arriba.

> Limitación del prototipo: sin cuentas de usuario, una persona puede votar varias veces. Cuando quieras evitarlo, añade Supabase Auth (también gratis) y una restricción `unique (osm_id, user_id)`.

## 3. Publicar gratis

Cualquiera de estas opciones sirve para un sitio estático:

- **GitHub Pages**: sube `index.html` a un repositorio → *Settings → Pages* → rama `main`.
- **Cloudflare Pages** o **Netlify**: conecta el repositorio o arrastra la carpeta.

## Notas sobre las APIs gratuitas

- **Nominatim**: máximo 1 petición por segundo, sin autocompletado y sin uso masivo. Aquí se hace una sola búsqueda por clic, que cumple su política.
- **Overpass**: servidor público compartido; puede responder 429/504 cuando está saturado. Si la web crece, cachea resultados o usa otra instancia.
- **OSM**: hay que mostrar la atribución "© OpenStreetMap" (ya incluida). Si una ciudad sale vacía, es que faltan etiquetas en OSM: se pueden añadir en openstreetmap.org y la web las recoge.

## Ideas siguientes

- Autenticación y un voto por usuario.
- Filtro por tipo (restaurante, panadería…) y "abierto ahora" (`opening_hours`).
- Geolocalización: "cerca de mí".
- Enlace para añadir o corregir un sitio directamente en OSM.
