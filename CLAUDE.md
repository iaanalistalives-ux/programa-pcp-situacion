# CLAUDE.md — Programa PCP · Situación de OP (programa-pcp-situacion)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
`index.html` reemplaza el Excel "Programa" de PCP (`prueba 22-10.xlsm`: pestañas DATA, Hoja2, REPORTE, STOCK POR PROCESO, LUISA, PROGRAMA TRANSFER, PROGRAMA DE INSPECCION, PRIORIDADES y REPORTE DE SEGUNDAS, con sus macros). Lee la base; no tiene pantalla de carga: los dos reportes del sistema los llena Sistemas directo.
- Barra: búsqueda (OP, grupo, estilo, P.O., color), cliente (3 letras del grupo), semana XFTY y Pendientes / Encajadas / Todas (por defecto pendientes = % encajado ≤ 100, como el filtro ENCAJADO = NO del Excel).
- KPIs: prendas por encajar (a producir − encajadas), % encajado (encajadas ÷ requeridas), minutos por cortar y por costurar, OP sin XFTY.
- Pestañas:
  - **Resumen**: flujo de corte, costura y acabado requerido por día (acumulado hasta cada fecha meta ÷ (días que faltan × eficiencia 0.85)) y stock por proceso por semana XFTY.
  - **Reporte** (antes "Situación OP"; nombre igual al del Excel desde el 22/10/2026): una fila por OP y color con subtotal por grupo (pestaña REPORTE); vista "Avance por proceso" (pestaña PRIORIDADES).
  - **Fechas XFTY y despacho**: la bandeja de trabajo de PCP. XFTY por OP (era Hoja2) y fecha de despacho por OP con excepción por color (era la columna AU). Se guardan solas, con historial; ninguna actualización del sistema las pisa. Fuera de la regla 500 (bandeja de trabajo, orden por XFTY).
  - **Corte**: cola de corte directo o paños (STOCK POR PROCESO), corte por mes de la fecha meta, % habilitado (LUISA).
  - **Costura e inspección**: costura por mes, programa de inspección, programa de transfer.
  - **Acabado y segundas**: acabado por mes y segundas por OP (rojo desde 2 %).
- Excel LIVES (bloque `lsXlsJs` v1.11 con `window.LS_XLS`): Resumen = la pestaña abierta; Detalle = cada OP y color filtrado.

## Base de datos (Supabase lives-datos, prefijo `sop_`)
- Puerta única `sop_api(p_accion, p)`: `leer`, `guardar_fecha` ({op, campo: 'xfty'|'despacho', fecha 'YYYY-MM-DD' o '' para borrar, comb opcional, quitar_colores}) e `historial` ({op}).
- Tablas que llena el sistema: `sop_situacion` (reporte Situación de Órdenes de Pedido, una fila por OP + color), `sop_pedido` (Situación de Pedido Global, una fila por OP), `sop_cargas`.
- Lo que escribe PCP: `sop_fechas`, `sop_fechas_color`, `sop_historial`. Quién registra: `sop_registran` (vtineo) o superusuario; Rodrigo solo mira.
- Reglas en tablas: `sop_excluir_po` (palabras de P.O. de muestra, distinguen mayúsculas como la macro), `sop_parametros` (días de las fechas meta 45/35/7, eficiencia 0.85, umbrales).
- Vista `sop_v_op`: todas las fórmulas de DATA (BI a DT) y los campos calculados de las tablas dinámicas. La ruta, minutos y consumo salen de la propia OP (el Excel los tomaba del primer OP del estilo).
- Registro SQL: serie `So` (So01 situacion op base). No usa Storage.

## Quién la usa
PCP (Analista de Planeamiento, vtineo). Absorbe a `despachos-pcp`, que queda como reserva sin cambios.

## Pendientes vistos al escribir esto (22/10/2026)
- Sistemas debe llenar `sop_situacion` y `sop_pedido` directo (hoy tienen los reportes del 22/10/2026).
- La columna Proveedor de Programa de Inspección no se incluyó: salía de una tercera hoja (PROVEEDORES DE COSTURA) con datos de enero 2026.
- Confirmar con PCP si las P.O. "TOP ORDER" son muestras (hoy se excluyen, igual que en el Excel).
