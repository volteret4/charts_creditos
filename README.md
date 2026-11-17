# Charts de créditos

Visor al estilo de `charts_artistas`, pero por créditos técnicos: escribes el nombre de un productor, un ingeniero o un masterizador y muestra los discos en los que ha trabajado, con vídeos de YouTube y la información de Last.fm.

## Para qué sirve

Para explorar de un vistazo lo que ha producido, mezclado, masterizado, grabado o remezclado alguien: un mosaico de portadas agrupado por rol, guardado en tu servidor para no repetir consultas.

## Instalación individual

Necesitas Docker.

```bash
git clone https://github.com/volteret4/charts_creditos.git
```

```bash
cd charts_creditos && cp .env.example .env && mkdir -p data
```

Rellena `LASTFM_API_KEY` en `.env` (y, si quieres, `DISCOGS_TOKEN` y `LASTFM_USERNAME`). Después:

```bash
docker compose up -d --build
```

Se abre en <http://localhost:8794>.

## Instalación desde REPEAT-ONE

Este módulo forma parte de [REPEAT-ONE](https://volteret4.github.io/repeat-one-web/), que permite elegir qué módulos instalar: [ver la instalación](https://github.com/volteret4/repeat-one-web#instalación). Allí se llama `charts_creditos`.

## Cambiar a otro servicio

Con `CHARTS_SWITCH=Nombre:puerto,Nombre:puerto` en el `.env`, la barra lateral muestra un selector con los servicios de ese mismo host que estén respondiendo. Sin la variable no aparece.

## Cómo funciona

1. **Búsqueda**: se piden candidatos a MusicBrainz y Discogs. Se marca por defecto la coincidencia exacta de cada fuente, y puedes cambiarla porque hay muchos homónimos.
2. **Créditos**: al añadir a alguien eliges los roles que quieres ver. Cada rol es una sección. Un disco con varios roles sale en la sección del primero, y en su ficha constan todos.

   | Rol | MusicBrainz | Discogs |
   |---|---|---|
   | Producción | `producer` | `Producer`, `Co-producer` |
   | Mezcla | `mix` | `Mixed by` |
   | Mastering | `mastering`, `lacquer cut` | — |
   | Ingeniería | `engineer`, `recording`, `audio`, `sound`, `balance` | — |
   | Remezcla | `remixer` | `Remix` |

   MusicBrainz da créditos de ediciones, discos y canciones. Una canción cuenta para los discos que la incluyen, menos los recopilatorios, salvo que solo exista en un recopilatorio. Las ediciones y canciones se resuelven en lotes de 25 con la búsqueda de MusicBrainz, no una a una.

   Discogs mete el mastering, la grabación y las colaboraciones como músico en un mismo rol, `Appearance`, que no se usa: separarlo exigiría abrir cada edición. Por eso el mastering y la ingeniería salen solo de MusicBrainz.
3. **Discos**: se juntan los dos catálogos por artista + título. MusicBrainz manda en el tipo de disco. Lo que solo está en Discogs saca el tipo del formato de su edición principal. El filtro de tipo de la barra superior muestra solo álbumes, EP, singles…
4. **Last.fm**: oyentes, escuchas, etiquetas, portada, wiki y canciones del disco; del artista del disco, su biografía, etiquetas y similares. La cabecera muestra lo que Last.fm sepa de la persona, si la conoce. Con `LASTFM_USERNAME` también tus escuchas de cada disco. Se refresca a los 14 días.
5. **YouTube**: `yt-dlp` busca y guarda hasta 5 vídeos cuyo título o canal nombren disco y artista. Si un disco solo está subido pista a pista, valen los vídeos del artista que nombren una de sus canciones. Si aún no se ha buscado un disco, se busca al pulsarlo.

El botón **Roles** los cambia después: lo desmarcado desaparece y, al volver a marcarlo, no se repiten las consultas ya guardadas.

Hay dos colas independientes. La primera trae créditos y Last.fm; al terminar, la persona pasa a la de YouTube. Así, mientras se buscan los vídeos de una, las siguientes ya van trayendo su información.

Un masterizador puede tener miles de discos. Con `YT_WORKERS=2`, 3000 discos son cerca de una hora de búsquedas de YouTube.

## Desarrollo

Sin Docker y desde esta carpeta (lee `.env` si existe):

```bash
pip install -r requirements.txt && python app.py
```

| Variable | Para qué |
|---|---|
| `LASTFM_API_KEY` | Información de Last.fm |
| `DISCOGS_TOKEN` | Opcional: sin él solo se usa MusicBrainz |
| `LASTFM_USERNAME` | Opcional: tus escuchas de cada disco |
| `YT_WORKERS` | Búsquedas de YouTube simultáneas (2 por defecto). Cada una suma unas 1700 búsquedas/h desde la misma IP |
| `SETTINGS_PASSWORD` | Panel de configuración ⚙ |
| `CHARTS_DATA_DIR` | Carpeta de datos (por defecto `data/`) |

## Si YouTube bloquea las búsquedas

Si responde «Sign in to confirm you're not a bot», la cola de YouTube se pausa 30 minutos y reintenta sola; si sigue bloqueado, la pausa se duplica hasta un máximo de 4 horas. La cola de Last.fm sigue avanzando. Si no se resuelve, exporta las cookies en formato Netscape a `data/cookies.txt` y pulsa ↻ en cualquier elemento para reintentar ya: solo se buscan los discos que faltan.

## Limitaciones

- Dos ediciones con títulos distintos («Beat» y «Beat (Remastered)») cuentan como discos distintos.
- El índice de búsqueda de MusicBrainz puede ir unas horas por detrás de sus datos: un crédito recién añadido puede tardar en salir.
- Los iconos de «ya lo tengo» vienen del servicio `ya_lo_tengo` (puerto 8780); sin él, simplemente no salen.

## Código

- `app.py`: servidor sin dependencias web, trabajos en segundo plano y SQLite (`data/charts_creditos.db`).
- `static/app.js`: la web.
