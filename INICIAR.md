# 🤖 BOT DEFINITIVO — Polymarket · Elon Musk # tweets

Bot de **paper trading** (prueba sin dinero) para los mercados «Elon Musk # tweets»
de Polymarket (ventanas de 48 h). Hace TODO automáticamente cada 15 minutos:

1. **Recoge tweets** de @elonmusk (jina/x.com con tiempos exactos + xcancel de respaldo)
2. **Actualiza los precios** de los mercados de Polymarket (bins + cuotas)
3. **Paper trading**: abre apuestas simuladas cuando hay señal real (cuota ≥ 3.00
   y p_modelo ≥ 60% YES / ≤ 30% NO) y las resuelve con el ganador REAL del mercado
4. **Genera el Excel** de resultados automáticamente
5. Todo queda en `bot.log`

---

## 📚 DOS GUÍAS IMPORTANTES (por separado)

| Guía | Contenido |
|---|---|
| **`GUIA_DINERO_REAL.md`** | Paso a paso para pasar el bot a **dinero real** (wallet, fondos USDC, API keys, modo seco, primera apuesta, liquidación, comisiones y riesgos) |
| **`GUIA_24H_GRATIS.md`** | Cómo hacer que el bot corra **24 horas sin parar gratis** (GitHub Actions, Oracle Cloud Always Free, tu PC, HF Spaces) |

El bot ya soporta ambos: `--modo papel` (default, sin dinero) y `--modo real`
(con `config_real.json`, ver GUIA_DINERO_REAL.md).

---

## ▶️ CÓMO ARRANCARLO

### Opción A — Modo continuo (RECOMENDADO)
```bash
cd estrategia_elon_tweets
python3 bot.py --loop --intervalo 15 --excel
```
Corre para siempre (cada 15 min). Cierra con `Ctrl+C`.

### Opción B — Una pasada manual
```bash
python3 bot.py
```

### Opción C — Ver el estado sin esperar
```bash
python3 bot.py --estado
```

### Opción D — Arranque automático (sin terminal abierta)

**Linux/Mac (cron):** `crontab -e` y añade:
```
*/15 * * * * cd /ruta/a/estrategia_elon_tweets && /usr/bin/python3 bot.py >> bot.log 2>&1
```
> ⚠ El cron no usa `--loop` (cada invocación = 1 pasada). Intervalo mínimo 15 min para no saturar las fuentes.

**Windows (Tarea programada):** Crea una tarea que ejecute cada 15 min:
`python C:\ruta\estrategia_elon_tweets\bot.py`

---

## 📦 REQUISITOS

- Python **3.9+**
- `curl` (viene con macOS/Linux/Windows 10+)
- Librerías (instalar una vez):
  ```bash
  pip install openpyxl        # obligatorio para generar Excels
  pip install numpy           # solo para reconstruir la serie (construir_serie.py)
  ```

## ⚙️ OPCIONES

| Opción | Qué hace |
|---|---|
| `--loop` | Modo continuo |
| `--intervalo N` | Pasadas cada N minutos (default 15) |
| `--excel` | Regenera `Resultados_Papel.xlsx` cuando hay cambios |
| `--sin-reposts` | Excluye reposts del conteo |
| `--estado` | Muestra saldo, paso, apuesta activa e historial |

## 🧪 OPCIONAL — fuente exacta (X API v2)

Sin API key el bot usa jina/x.com + xcancel (buenas, pero el historial profundo
es limitado). Con un token oficial de X (plan Basic, ~$100/mes) el conteo es exacto:
```bash
export X_BEARER_TOKEN=tu_token
python3 bot.py --loop --intervalo 15
```

## 📁 ARCHIVOS QUE GESTIONA

| Archivo | Contenido |
|---|---|
| `datos_elon.csv` | Serie diaria de tweets (fecha, tweets) — base de la señal |
| `estado_tweets.json` | Tweets crudos recogidos (con tiempos exactos) |
| `mercado_activo.json` | Mercados de Polymarket con bins y cuotas en vivo |
| `papel.json` | Estado del paper trading (saldo, paso, apuesta activa) |
| `resultados_papel.csv` | Historial de apuestas de papel |
| `Resultados_Papel.xlsx` | Excel de resultados (KPIs + curva de saldo) |
| `bot.log` | Registro de todas las pasadas |

## 📊 HERRAMIENTAS COMPLEMENTARIAS

```bash
python3 senal_vivo.py --actualizar     # señal detallada de todos los mercados
python3 mercado_polymarket.py          # tabla de mercados y cuotas en vivo
python3 simulador.py --csv datos_elon.csv --bin 40 64 --precio 0.33 --excel --montecarlo 300
python3 papel.py --excel               # forzar actualización del Excel de papel
```

## 📱 AVISOS AL MÓVIL (abiertas, cerradas y resultado)

El bot te avisa al móvil en cuanto **abre** una apuesta de papel y cuando la **cierra** (con su resultado). Dos canales:

### Opción 1 · ntfy.sh (RECOMENDADA — gratis, sin registro, 1 minuto)
1. Instala la app **ntfy** en tu móvil (App Store / Google Play / F-Droid).
2. Ábrela y suscríbete al tema: **`elon-poly-g2p7e8ev`**
3. ✅ Listo. Ya has recibido (o recibirás) el mensaje de prueba.

> El tema está en `config.json`. Puedes cambiarlo cuando quieras:
> `python3 notificar.py --topic tu-tema-nuevo`

### Opción 2 · Telegram (opcional)
1. En Telegram, habla con **@BotFather** → `/newbot` → copia el *token*.
2. Consigue tu **chat_id** (escríbele a @userinfobot o usa `getUpdates`).
3. Edita `config.json`:
```json
{
  "ntfy": { "topic": "elon-poly-g2p7e8ev", "token": null },
  "telegram": { "token": "123456:ABC-DEF...", "chat_id": "123456789" }
}
```

### Probar el aviso
```bash
python3 notificar.py --test        # envía prueba a todos los canales
python3 bot.py --notificar-test    # igual, desde el bot
```

**Ejemplo de aviso de cierre:**
```
🔔 Apuesta cerrada
✅ GANADA  +$6.70
Bin 40-64 · Lado YES
Ganador real del mercado: 40-64
Stake $3.30 · Paso 1
Saldo: $506.70
```

### Avisos adicionales que ya envía el bot
| Situación | Aviso |
|---|---|
| Bin **cerca** de cumplir la señal pero sin apostar (cuota justa o p_modelo justo) | 👀 **Casi señal (sin apuesta)** — informativo, máx. 1 cada 6 h por mercado |
| Fallo en la recogida de tweets (jina y xcancel) o en la actualización del mercado | 🚨 **Alerta del bot** — prioridad ALTA, máx. 1 por hora |
| **Resumen diario** (una vez al día, por defecto a las **20:00**) | 📊 Saldo, beneficio acumulado, operaciones de hoy, apuesta activa, mercados 48 h abiertos y métricas AVG7/V2/λ48 |

Con esto el móvil te cuenta todo: abiertas ✅, cerradas con resultado 🔔, señales que se quedan a las puertas 👀, problemas técnicos 🚨 y el estado del día 📊.

> Cambiar la hora del resumen diario:
> ```bash
> python3 notificar.py --resumen-hora 21   # a las 21:00
> python3 notificar.py --resumen-hora -1   # desactivar
> ```

## 📊 EXCEL ACUMULATIVO — Historial_Operaciones.xlsx

El bot mantiene un **libro de operaciones que solo crece**: cada vez que se
resuelve una apuesta de papel, se **AÑADE** una fila nueva (con su ID único,
nunca duplica ni borra las anteriores). Hojas:

| Hoja | Contenido |
|---|---|
| **Operaciones** | Cada operación: fecha, tipo (PAPEL/SIMULACIÓN), mercado, bin, lado, precio, cuota, p_modelo, paso, stake, real, resultado, beneficio, saldo |
| **Resumen** | KPIs en vivo: win rate, beneficio, ROI, saldo, total invertido, cuota media, racha y drawdown |
| **Equity** | Curva de saldo con gráfico, actualizada con todas las filas |
| **Notas** | Cómo funciona y avisos |

Regenerar/añadir manualmente:
```bash
python3 excel_historial.py                              # añade las de papel
python3 excel_historial.py --csv resultados_simulacion.csv   # añade otras
```
> El libro actual (semilla) contiene las 2 operaciones de la simulación
> con datos reales (tipo SIMULACIÓN); las de papel se irán añadiendo como
> tipo PAPEL cuando el bot resuelva apuestas.

## ⚠️ AVISOS

- **Es paper trading**: no mueve dinero real. Las señales son reales, los saldos son ficticios.
- La regla de oro del sistema: **no apostar si no hay señal con cuota ≥ 3.00**.
- Los datos de días recientes pueden ser parciales hasta que el bot acumula 24 h de cobertura continua; el guard monótono evita que los datos se degraden.
- Esto no es asesoramiento financiero. Si algún día pasas a dinero real: bankroll mínimo $500, una sola apuesta activa, stop de ciclo en el paso 7.
