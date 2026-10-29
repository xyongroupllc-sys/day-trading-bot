# Cierre — 3:45 PM ET / 12:45 PM PT

**Rutina:** `trig_01JfXHtBEcy8Y3Twi3CaPN1q`
**Cron:** `45 19 * * 1-5` (UTC)
**MCPs:** Robinhood-Trading, Notion

---

Cierre obligatorio de fin de día del sistema de day trading automatizado (Ciclo "V2 - Ago 2026+"). Corres a las 3:45 PM ET / 12:45 PM PT.

CUENTA ROBINHOOD: 737702357 (••••2357)
(Función Telegram y PASO 0 idénticos a los otros monitores)

PASO 1 — LEER ESTADO DE NOTION:
Busca registros de HOY (Ciclo="V2 - Ago 2026+").
Si no hay registros de hoy: terminar sin notificar.

PASO 2 — CIERRE DE POSICIONES ABIERTAS:
get_equity_positions(account_number='737702357')
get_equity_orders(account_number='737702357')

Para cada candidato de hoy con Estado="En curso":
    TICKER, FILL_PRICE, FILL_QTY, ADVANCED_ORDER_ID, STOP_ORDER_ID, TARGET_ORDER_ID ← de Notion

    Si STOP_ORDER_ID filled → Notion Estado="Cerrada - Stop"
    Si TARGET_ORDER_ID filled → Notion Estado="Target Alcanzado"
    Si posición aún abierta en Robinhood:
        a. cancel_advanced_order(ADVANCED_ORDER_ID) — o cancelar STOP + TARGET por separado (compatibilidad)
        b. review_equity_order → place_equity_order market sell
        c. VERIFICACIÓN: esperar ~5s → get_equity_orders(CIERRE_ORDER_ID)
           Si state != 'filled': reintentar una vez.
           Si segundo intento tampoco confirma: tg('⚠️ ERROR CIERRE {TICKER}: no confirmó fill — revisar manualmente')
                                               Notion Estado="Cierre Forzado - SIN CONFIRMAR"
        d. Si confirmó filled: Notion Estado="Cierre Forzado", Precio Cierre $, P&L $

Para candidatos con Estado="En espera de fill":
    Actualizar Notion: Estado="No Ejecutada"

PASO 3 — ACTUALIZAR JOURNAL:
Actualizar/crear en "📋 Day Trading Journal" (Tipo="📊 Cierre", Ciclo="V2 - Ago 2026+"):
- Señales Totales, Señales Ejecutadas, Señales Ganadas, Tasa de Éxito %, P&L Neto del Día

PASO 4 — BOLETÍN DE CIERRE (2 mensajes separados):

MENSAJE 1 — Reporte de Cierre:
Tabla: #rango | ticker | tipo | entrada | resultado | P&L $
Resultados posibles: Target Alcanzado / Cerrada - Stop / Cierre Forzado / Cierre Forzado - SIN CONFIRMAR / Cerrada - Gestión Activa / No Ejecutada.

MENSAJE 2 — Scorecard del Día:
Señales generadas vs ejecutadas vs ganadas/perdidas, P&L neto, tasa de acierto.
Lecciones del día (qué funcionó, qué no).
Outlook breve para la siguiente sesión.

TODO en español. Horas en PT (ET - 3h), nunca en ET.

MANEJO DE ERRORES: tg(f'ERROR Cierre DT [FECHA PT] paso [N]: {str(e)[:400]}')
