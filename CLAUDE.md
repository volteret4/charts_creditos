# charts_creditos

`app.py`: servidor HTTP de la biblioteca estándar + SQLite (`CHARTS_DATA_DIR/charts_creditos.db`) + dos colas en segundo plano (créditos/Last.fm y YouTube con yt-dlp). `static/`: la web.

- Créditos de MusicBrainz (ediciones, discos y canciones) y Discogs; tabla de roles en el README.
- Los iconos de «ya lo tengo» los pinta `owned-badges.js` de ya_lo_tengo (puerto 8780) si responde.
- Configuración por entorno (`.env.example`); el panel ⚙ exige `SETTINGS_PASSWORD`.
