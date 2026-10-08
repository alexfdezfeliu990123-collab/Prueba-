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
