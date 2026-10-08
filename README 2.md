# ship-alert

Muestra barcos AIS con coordenadas, calcula distancia/CPA/TCPA y avisa por proximidad. Funciona sin internet.

## Hardware (lo que hace que funcione offline)
- **GPS**: receptor USB/NMEA (u-blox, etc.). El GPS es satelital y no necesita internet.
- **AIS**: receptor VHF AIS (clase B/transpondedor) por NMEA 0183, o un dongle RTL-SDR + [AIS-catcher](https://github.com/jvde-github/AIS-catcher) (`AIS-catcher -u 127.0.0.1 10110`).
- Equipo con batería/UPS (Raspberry Pi + batería 12 V) para cortes de luz.

## Uso
```bash
pip install -r requirements.txt
python ship_alert.py --src serial:/dev/ttyUSB0:4800 --src udp:10110 --alert-nm 3 --cpa-nm 0.5 --tcpa-min 20
```
Genera `targets.geojson` con todos los barcos (ábrelo en QGIS u OpenCPN).

## Cartas náuticas offline y actualización
Usa **OpenCPN** (gratis) junto con este script:
1. Descarga cartas ENC (S-57) / BSB: NOAA (EE.UU., gratis), SHOA (Chile), IHM/Armada (España), SHN (Argentina), UKHO, o o-charts.org (de pago).
2. En OpenCPN: *Opciones → Cartas → Añadir directorio*, y *Actualizar* tras copiar las nuevas.
3. Actualizar: descarga la nueva edición/parches (ENC se actualizan semanal/mensualmente) mientras tengas internet y cópialos al equipo; una vez descargadas funcionan sin conexión.
4. En OpenCPN añade conexión: *Opciones → Conexiones → UDP/Serial* y recibirás AIS + GPS además de este script.

## Radares por marca
El radar no se lee directamente: se integra por red o NMEA.
| Marca | Cómo integrarlo |
|---|---|
| Furuno | NMEA 0183 (sentencias TTM/RSD) o Ethernet; actualizar firmware desde su web/SD |
| Raymarine | Ethernet/SeaTalk-NG→NMEA 2000; plugin de radar de OpenCPN |
| Simrad / B&G / Lowrance | Ethernet; plugin `radar_pi` de OpenCPN |
| Garmin | Ethernet (HD/xHD) con `radar_pi`; NMEA 2000 vía pasarela |

Actualiza el firmware de cada radar con el procedimiento del fabricante (tarjeta SD/USB). Con objetivos TTM, añade `--src` apuntando al puerto del radar.

## Limitaciones
- Solo ves barcos que transmiten AIS. Los demás (pesca pequeña, militares) solo con radar.
- Alcance AIS ≈ 20-40 millas náuticas según antena.
- **No es un sistema de seguridad certificado**: no sustituye la vigilancia ni el equipo homologado.

## Subir a GitHub
```bash
git init && git add . && git commit -m "ship-alert inicial"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/ship-alert.git
git push -u origin main
```

## Mapa web con zonas de profundidad (offline)
```bash
python ship_alert.py --src serial:/dev/ttyUSB0:4800 --src udp:10110 --http 8080
# abre http://localhost:8080 (o http://IP_DE_LA_RASPBERRY:8080 desde el móvil, por WiFi local sin internet)
```
- Sin hardware, ábrelo con `python -m http.server -d web 8080` y verás el **modo demo** (datos ficticios, NO navegar).
- Colores por profundidad mínima (DRVAL1): seco, 0-2 m, 2-5 m, 5-10 m, 10-20 m, 20-50 m, >50 m.
- Alerta de **bajo fondo**: proyecta tu rumbo 10 min y avisa si la profundidad < calado + margen. Suma la marea al margen.
- Zonas de profundidad reales: `./tools/enc2geojson.sh CARTA.000` (necesita GDAL) y reemplaza `web/data/depare.geojson`. Para actualizar, repite con la edición nueva de la carta.
- El fondo no lleva mapa de calles/costa: la costa sale de la propia carta (añade también la capa LNDARE si la quieres).

## Búsqueda, seguimiento de buques y rutas
- **Buscar**: escribe nombre o MMSI; se resaltan las coincidencias y, si hay una sola, se selecciona y centra.
- **Seleccionar**: toca un barco en el mapa o en la lista. Ves MMSI, posición, velocidad, rumbo, demora, CPA y TCPA.
- **👁 Seguir**: alerta con el doble de radio y CPA. **🚫 Ignorar**: sin alertas, sin etiqueta y fuera de la lista. **Normal** lo restablece. Se guarda en el navegador.
- **🧭 Ruta**: activa el modo ruta y toca el mapa para añadir waypoints (toca uno existente para borrarlo; ↶ deshace; 🗑 borra). Muestra distancia total, tiempo a la velocidad indicada, rumbo/distancia/tiempo al siguiente WP y desvío (XTE). Los tramos con fondo menor que calado+margen se marcan en rojo. **GPX** exporta la ruta para OpenCPN o un plotter.
- Las marcas y la ruta se guardan en el navegador (localStorage), por lo que hay que abrir la web desde el servidor (`--http`), no desde un archivo suelto.
