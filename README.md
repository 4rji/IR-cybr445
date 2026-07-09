# IR-cybr445

<p align="center">
  <img src="logo.png" alt="Logo del proyecto" width="180">
</p>

Repositorio de evidencias y guía de trabajo para un informe de respuesta a incidentes. El material documenta el análisis de una intrusión en un equipo Windows, con foco en memoria, disco, artefactos de navegador, Prefetch, registro de Windows, eventos y posibles indicadores de compromiso.

## Resumen del Caso

La evidencia apunta a una infección por **Xtreme RAT** después de que el usuario visitara un sitio web comprometido. El análisis describe una cadena de ataque con posible Exploit Kit, descarga y ejecución de malware, uso de herramientas de reconocimiento y credenciales, escaneo de red interna y posible transferencia de datos mediante herramientas SSH/SCP.

El documento principal reconstruye la actividad observada alrededor del **16 de agosto de 2016**, usando artefactos forenses como Volatility, ClamAV, historial/caché de Firefox, Prefetch, Windows Event Logs, RegRipper y Windows Registry Recovery.

## Contenido del Repositorio

| Ruta | Descripción |
|------|-------------|
| [`Evidence-for-first-incident-response-report.md`](Evidence-for-first-incident-response-report.md) | Evidencia transcrita del PDF original, con explicación del análisis y referencias a capturas. |
| [`ir_report_instructions.md`](ir_report_instructions.md) | Instrucciones para generar un reporte formal de respuesta a incidentes en Markdown y LaTeX. |
| [`capturas/`](capturas/) | Carpeta con 43 imágenes usadas como evidencia visual del análisis. |
| [`logo.png`](logo.png) | Logo previsto para la portada del reporte. |
| [`.gitattributes`](.gitattributes) | Configuración básica de normalización de finales de línea para Git. |

## Evidencia Incluida

El repositorio incluye capturas y texto relacionado con:

- Análisis de memoria con Volatility.
- Detección de procesos sospechosos asociados a Xtreme RAT.
- Revisión de conexiones de red.
- Análisis de imagen de disco y sistema de archivos.
- Hallazgos en caché e historial de Firefox.
- Análisis de ejecutables `3568226350.exe` y `54948tp.exe`.
- Evidencia de ejecución en Prefetch.
- Eventos de Windows relacionados con `hydra.exe`.
- Persistencia y rastros en el registro de Windows.
- Línea de tiempo general del incidente.
- Tabla de indicadores de compromiso.

## Hallazgos Principales

- Malware identificado: **Xtreme RAT**.
- Procesos sospechosos: `svchost.exe`, `explorer.exe` y `update.exe`.
- Ruta relevante de persistencia/ejecución: `%APPDATA%\HostData\update.exe`.
- Posible vector inicial: visita a `blog.mycompany.ex`, redirección a `blog.mysportclub.ex` y explotación relacionada con **CVE-2012-3993**.
- Herramientas observadas: Mimikatz, BrowserPasswordDump, Nmap, THC Hydra, Plink y PSCP.
- Red interna observada en el análisis: `192.168.5.1`, `192.168.5.10` y `192.168.5.15`.
- Software vulnerable identificado: Mozilla Firefox 33.0.3 y Adobe Flash Plugin 18.0.0.194.

## Indicadores de Compromiso Destacados

| Tipo | Indicador |
|------|-----------|
| Malware | Xtreme RAT |
| Dominio | `blog.mycompany.ex` |
| Dominio | `blog.mysportclub.ex` |
| URL | `http://blog.mysportclub.ex/wp-content/uploads/hk/task/opspy/index.php` |
| URL | `http://blog.mysportclub.ex/wp-content/uploads/hk/files/data_32.bin` |
| Archivo | `3568226350.exe` / `3568226350[1].exe` |
| Archivo | `%TEMP%\54948tp.exe` |
| Ruta | `%APPDATA%\HostData\update.exe` |
| Ruta | `%APPDATA%\EpUpdate\` |
| Registro | `GhCtxq8t` |

## Uso Sugerido

1. Revisar [`Evidence-for-first-incident-response-report.md`](Evidence-for-first-incident-response-report.md) para entender la evidencia original.
2. Consultar las imágenes en [`capturas/`](capturas/) cuando el documento referencie una figura.
3. Usar [`ir_report_instructions.md`](ir_report_instructions.md) como guía para redactar el informe final de respuesta a incidentes.
4. Crear los entregables esperados:
   - `incident_report_draft.md`
   - `incident_report.tex`

## Estructura Esperada del Reporte

Según las instrucciones incluidas, el reporte final debe contener:

- Portada.
- Tabla de contenido.
- Resumen ejecutivo.
- Línea de tiempo.
- Hallazgos.
- Preguntas investigativas.
- Sistemas y personas involucradas.
- Indicadores de compromiso.
- Evidencia recolectada.
- Remediación.
- Recomendaciones.
- Lecciones aprendidas.
- Apéndices.
- Referencias.

## Nota

Este repositorio contiene evidencia transcrita y capturas asociadas para fines académicos o de práctica de análisis forense. No incluye las imágenes forenses originales de memoria o disco, por lo que cualquier reporte final debe limitarse a los hechos documentados en la evidencia disponible.
