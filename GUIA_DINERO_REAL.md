# 💰 GUÍA DEFINITIVA — Poner el bot en funcionamiento con DINERO REAL

**Bot:** Polymarket · «Elon Musk # tweets» (ventanas de 48 h)
**Versión del bot:** `bot.py --modo real` + `operar_real.py`

> ⚠️ **LEE ESTO PRIMERO**
> - Esto **no es asesoramiento financiero**. Las predicciones con dinero real conllevan riesgo de pérdida total.
> - Usa **solo dinero que puedas permitirte perder por completo**.
> - Polymarket **no está disponible para residentes de EE. UU.**; en España es accesible, pero revisa la normativa local y tus obligaciones fiscales (los beneficios de trading en cripto se declaran en la renta).
> - El módulo real **NO ha sido probado con dinero real**: sigue los pasos en orden y empieza con cantidades mínimas ($3.30).

---

## 🗺️ Resumen del camino (10 pasos)

| Paso | Qué haces | Tiempo |
|---|---|---|
| 1 | Crear wallet y cuenta en Polymarket | 15 min |
| 2 | Depositar USDC + POL (gas) | 10 min |
| 3 | Crear API keys en Polymarket | 5 min |
| 4 | Instalar dependencias | 5 min |
| 5 | Configurar `config_real.json` | 5 min |
| 6 | **Modo seco** (--simular) | 10 min |
| 7 | Primera apuesta real mínima | 5 min |
| 8 | Verificar liquidación automática | 2 días |
| 9 | Activar el bot en automático | 5 min |
| 10 | Controlar comisiones, slippage y riesgos | continuo |

---

## PASO 1 · Wallet y cuenta en Polymarket

1. **Instala MetaMask** (extensión de navegador o app móvil). Crea una wallet nueva y **guarda la frase semilla en papel** (nunca en el ordenador).
2. **Añade la red Polygon** a MetaMask (ajustes → redes → añadir red):
   - Nombre: `Polygon Mainnet`
   - RPC: `https://polygon-rpc.com`
   - Chain ID: `137`
   - Símbolo: `POL`
3. **Crea tu cuenta en Polymarket** (polymarket.com): email + conectar wallet (firma el mensaje).
4. *(Opcional pero recomendado)*: en Polymarket usa la **wallet de custodia/smart wallet** que te asignan por defecto (más segura para el bot, porque no expones tu clave privada del navegador).

## PASO 2 · Fondos (USDC en Polygon + POL para gas)

Polymarket opera con **USDC en Polygon** (stablecoin = 1 $). Además necesitas un poco de **POL** para pagar el gas de las transacciones de aprobación/depósito.

| Concepto | Cantidad recomendada |
|---|---|
| USDC (Polygon) | bankroll inicial: **$500** mínimo según la estrategia |
| POL (Polygon) | **$2–5** (0.5–1 POL) — solo para gas, sobra de sobra |

**Cómo conseguir USDC en Polygon:**
- **Opción A (exchange)**: compra USDC en Binance/Kraken/etc. → retira por **red Polygon** → dirección de tu wallet Polymarket (o MetaMask) → llega en ~1 min con comisión de ~$0.50.
- **Opción B (swap)**: si ya tienes USDC en Ethereum, pásalo a Polygon con el puente oficial (bridge.polygon.technology).

**Depositar en Polymarket:**
1. Entra en Polymarket → **Deposit** (o Portafolio → Deposit).
2. Elige cantidad en USDC (Polygon) → confirma en MetaMask (te pedirá **aprobar** el contrato = 1 transacción de gas, y luego el **depósito** = otra).
3. En la página verás tu saldo disponible en USDC.

> 🔐 **Seguridad**: nunca compartas tu frase semilla. La API key de Polymarket (paso 3) está limitada a trading, no puede retirar fondos — así que aunque la roben, no pueden sacar tu dinero (solo el wallet dueño retira).

## PASO 3 · Crear las credenciales de la API (flujo OFICIAL verificado en docs.polymarket.com)

Para una cuenta creada por **email** (sin MetaMask), necesitas **2 cosas + 1 opcional**:

### 3.1 · Dirección de tu cuenta (wallet address)
Perfil de Polymarket (menú del avatar, arriba a la derecha) → **copia la dirección `0x...`** que se muestra (es tu *Deposit Wallet*). Es la misma que usas para depositar.

### 3.2 · Clave privada del firmante (signer) — OBLIGATORIA
Como tu cuenta se creó por email, Polymarket (Magic Link) te permite exportar la clave privada:
1. Entra en **https://reveal.magic.link/polymarket** (estando logueado en Polymarket).
2. Inicia sesión en Magic Link.
3. **Export Private Key** → se muestra la clave privada (empieza por `0x...`).
4. **Guárdala en un sitio seguro** (NO compartirla, NO subirla a GitHub — está en `.gitignore`).
5. Cierra sesión de Magic Link.

### 3.3 · Clave API del Relayer (OPCIONAL, para operar sin pagar gas)
1. En Polymarket → **Settings → API Keys → Relayer API Keys** → **Create** (esto es lo que has visto en pantalla).
2. Copia la **API Key** y el **Signer Address** que se muestran.
3. Con ella el bot puede firmar transacciones de liquidación **sin que necesites POL para gas**. Si no la pones, el bot igualmente funciona (necesitarás POL en la wallet para el gas de las liquidaciones).

> ❓ ¿Y las "API Key + Secret + Passphrase" (CLOB)?
> Son **opcionales**: el bot puede **derivarlas automáticamente** firmando con la clave privada (paso 3.2) cuando llama a `create_or_derive_api_creds()`. Si algún día las ves en Settings → API Keys (sección CLOB), puedes copiarlas también; si no, no pasa nada.

## PASO 4 · Instalar dependencias

```bash
# Python 3.9+ (comprueba: python3 --version)
pip install openpyxl py-clob-client requests
```

El bot en sí no necesita más (curl ya viene en tu sistema).

## PASO 5 · Configurar `config_real.json`

```bash
cd estrategia_elon_tweets
cp config_real.json.example config_real.json
notepad config_real.json    # en Windows
nano config_real.json       # en Linux/Mac
```

Rellena:

```json
{
  "api_key": "",
  "api_secret": "",
  "api_passphrase": "",
  "wallet_private_key": "0x_CLAVE_PRIVADA_DEL_FIRMANTE_(paso_3.2)",
  "wallet_address": "0x_DIRECCION_DE_TU_CUENTA_(paso_3.1)",
  "relayer_api_key": "",
  "relayer_api_key_address": "",
  "bankroll": 500,
  "fee_pct": 0.0,
  "confirmado": false
}
```

| Campo | Qué poner |
|---|---|
| `wallet_private_key` | 🔑 La clave privada del firmante (paso 3.2) — **obligatoria** |
| `wallet_address` | Tu dirección `0x...` de Polymarket (paso 3.1) — **obligatoria** |
| `relayer_api_key` / `relayer_api_key_address` | Opcionales (paso 3.3) — para gasless |
| `api_key` / `api_secret` / `api_passphrase` | Déjalas vacías (se derivan solas) |
| `bankroll` | Tu capital total dedicado (p. ej. 500) |
| `fee_pct` | Comisión taker del mercado (0.00–0.02) |
| `confirmado` | **`false`** hasta el paso 9 |

> Alternativa segura sin archivo en disco: variables de entorno
> ```bash
> export POLY_PRIVATE_KEY=0x...
> export POLY_WALLET_ADDRESS=0x...
> export REAL_CONFIRMADO=1
> ```

## PASO 6 · Modo seco (prueba sin arriesgar nada)

```bash
python3 bot.py --modo real --simular --excel
```

- El bot recoge datos, actualiza precios y **SIMULA** la orden que enviaría (sin enviarla).
- Debe imprimir algo como: `→ Orden SIMULADA: Elon Musk # tweets ... 40-64 YES @ 0.330 ...` cuando haya señal; si no hay señal, dirá `(sin señal 48 h → no se abre apuesta real)`.
- También puedes forzar la señal con datos de prueba:
  ```bash
  python3 operar_real.py --simular
  ```

> ✅ Cuando veas la orden simulada correcta (bin, lado, precio, stake según tu paso), pasamos al siguiente paso.

## PASO 7 · Primera apuesta real MÍNIMA

1. Asegúrate de tener **al menos $10 de USDC** en Polymarket (para el primer stake de $3.30 + margen).
2. En `config_real.json` pon `"confirmado": true`.
3. Ejecuta UNA pasada real:
   ```bash
   python3 bot.py --modo real --excel
   ```
4. El bot:
   - Comprueba el saldo USDC (si es insuficiente, **bloquea**).
   - Envía una **orden límite GTC** a tu precio (cuota ≥ 3.00).
   - Espera a que se llene; si no se llena en **60 min, la cancela** (nunca persigue precio).
   - Te avisa al móvil: `💰 Apuesta REAL abierta`.
5. **Comprueba en Polymarket** (Portafolio → Open Orders) que la orden está ahí, y luego en Positions cuando se llene.

> 💡 Si la orden no se llena y se cancela, **no es una pérdida**: solo no entraste. El bot lo repetirá en la siguiente ventana si la señal sigue viva.

## PASO 8 · Liquidación (cómo recuperas el dinero)

- Cuando el mercado se resuelve (el día D+2 a las 12:00 ET), Polymarket **canjea automáticamente** tus shares ganadoras → el USDC **vuelve solo a tu saldo** de Polymarket. No tienes que hacer nada.
- El bot detecta el ganador real, anota **GANADA/PERDIDA** en el historial y el Excel, y te notifica:
  ```
  💰 Apuesta REAL cerrada
  ✅ GANADA +$6.70 (REAL)
  Bin 40-64 · YES · ganador real 40-64
  Stake $3.30 · saldo $506.70
  ```
- Si pierdes, las shares valen 0 y pierdes el stake (más la comisión si la hay).

## PASO 9 · Activar el bot en automático

Ahora sí, déjalo vigilando 24/7. Elige el método según el documento **GUIA_24H_GRATIS.md** (GitHub Actions gratis, Oracle Cloud gratis o tu propio PC).

```bash
# En tu PC (modo continuo):
python3 bot.py --modo real --loop --intervalo 15 --excel
# O en GitHub Actions / cron: una pasada por invocación
python3 bot.py --modo real --excel
```

## PASO 10 · Comisiones, slippage y controles de riesgo

### Comisiones
| Concepto | Detalle |
|---|---|
| Taker fee (mercados de tweets) | Normalmente **0%**; algunos mercados 1–2%. Comprueba la página del mercado (campo "fees"). |
| Gas (depósito/aprobación) | ~$0.01–0.05 por tx en Polygon. Irrelevante. |
| Retiro | Polymarket no cobra; solo gas de Polygon. |

### Slippage
- El bot usa **órdenes límite a tu precio**, así que no pagas slippage: compras exactamente a la cuota que viste (o no compras).
- Riesgo real: que la orden **no se llene** (si el precio sube antes). Aceptado por diseño (cancelación a los 60 min).

### Controles de riesgo ya integrados en `operar_real.py`
1. **`confirmado: true`** obligatorio (sin ello, no envía nada).
2. **Verificación de saldo USDC** antes de cada orden.
3. **Una sola apuesta real activa** (secuencial, como en papel).
4. **Stake según tabla** 3.30 × 1.5^(paso−1) y **stop de ciclo en el paso 7** (pérdida máx. $106.18).
5. **Límite de bankroll**: ningún stake puede superar el 50% del bankroll.
6. **Cancelación automática** de órdenes no llenadas a los 60 min.
7. **Notificaciones al móvil** en cada evento.

### Tus controles (humanos)
- Nunca subas el stake inicial (3.30) ni el multiplicador (1.5).
- Si pierdes el ciclo completo (7 pasos): **pausa 24 h** y revisa.
- Revisa los KPIs del Excel cada semana: si el win rate baja del 40% en 30 días → parar y recalibrar.
- **Retira beneficios** periódicamente a tu wallet (Polymarket → Withdraw → Polygon → exchange).

---

## ✅ CHECKLIST FINAL ANTES DE DINERO REAL

- [ ] Wallet creada y frase semilla guardada en papel
- [ ] Cuenta Polymarket creada con la wallet conectada
- [ ] USDC depositado en Polymarket (≥ $10 para empezar, $500 como bankroll objetivo)
- [ ] POL disponible para gas
- [ ] API keys creadas (API Key, Secret, Passphrase) y guardadas
- [ ] `pip install openpyxl py-clob-client requests` sin errores
- [ ] `config_real.json` relleno con `"confirmado": false`
- [ ] `python3 bot.py --modo real --simular` muestra la orden simulada correctamente
- [ ] `"confirmado": true` puesto
- [ ] Primera pasada real: orden visible en Polymarket (Open Orders)
- [ ] Notificación al móvil recibida
- [ ] Bot en automático (ver GUIA_24H_GRATIS.md)
- [ ] Excel `Historial_Operaciones.xlsx` registrando las operaciones tipo REAL

---

## 🆘 Solución de problemas

| Problema | Causa probable | Solución |
|---|---|---|
| `Falta 'py-clob-client'` | No instalaste la librería | `pip install py-clob-client` |
| `[BLOQUEADO] config_real.json no tiene confirmado` | Olvidaste el paso 7 | Pon `"confirmado": true` |
| `[BLOQUEADO] saldo insuficiente` | No hay USDC en Polymarket | Deposita (paso 2) |
| `no se pudo enviar la orden: ...` | Claves mal copiadas | Revisa API Key/Secret/Passphrase; revoca y recrea si hace falta |
| La orden no se llena y se cancela | Precio de mercado > tu límite | Normal: el bot esperará otra señal. No subas el precio manualmente |
| `get_balance_allowance` da error | Versión de la librería | `pip install -U py-clob-client`; si sigue, revisa la doc en github.com/Polymarket/clob-client |

---

*Documento de gestión personal. No constituye asesoramiento financiero ni de inversión. Juega/predice solo con dinero que puedas perder.*
