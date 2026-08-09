# ⏰ GUÍA DEFINITIVA — El bot 24 HORAS SIN PARAR, GRATIS (paso a paso)

**Tu situación actual (verificada):**
- ✅ Bot funcionando en tu PC (Windows, PowerShell) en **MODO REAL**
- ✅ Credenciales OK (clave privada + wallet address), saldo $36.55 pUSD
- ✅ SDK V2 (`py-clob-client-v2`) y `create_and_post_order` corregidos
- ❌ Ahora solo se ejecuta cuando tú escribes el comando

**Objetivo de esta guía:** que el bot vigile solo, **cada 15 minutos, 24/7, sin pagar nada**.

---

## 🆚 Las 3 vías (elige una)

| Vía | ¿24/7 real? | Coste | Complejidad | Ideal si… |
|---|---|---|---|---|
| **A · GitHub Actions** ⭐ | Sí (cron cada 15 min) | **0 €** | Media | No quieres dejar el PC encendido |
| **B · Tu propio PC** (Tarea programada) | Sí (si el PC está encendido) | 0 € | **Baja** | Tu PC ya está siempre encendido |
| **C · Oracle Cloud Always Free** | Sí (servidor en la nube) | 0 € | Media-Alta | Quieres un servidor de verdad |

> **Recomendación rápida**: si tu PC suele estar encendido → **Vía B** (5 minutos, sin GitHub).
> Si no quieres depender del PC → **Vía A** (GitHub Actions, 15-20 minutos).

---

# VÍA A · GitHub Actions (recomendada si no quieres dejar el PC) ⭐

GitHub ejecuta gratis una pasada del bot cada 15 minutos. El bot **guarda su estado haciendo commit** en cada pasada → los datos viven en el repo y nunca se pierden.

## A.1 · Crea una cuenta GitHub (si no tienes)
1. Ve a **github.com** → **Sign up** → email + contraseña + usuario.
2. Confirma el email.

## A.2 · Crea el repositorio (PÚBLICO)
1. GitHub → botón **"+"** (arriba a la derecha) → **New repository**.
2. Nombre: `bot-polymarket-elon`
3. **Visibilidad: PÚBLICO** ⚠️ Importante:
   - Público → **minutos ilimitados** gratis (y el cron nunca se pausa).
   - Privado → solo 2.000 min/mes (~15-30 pasadas/día) — no llega para cada 15 min.
   - **No pasa nada por ser público**: tus claves NO se suben (van en "secrets", ocultos), y `config_real.json` está excluido por `.gitignore`.
4. **NO** marques "Add a README" (para subir limpio).
5. **Create repository**.

## A.3 · Sube la carpeta del bot (desde PowerShell)

En tu PC, en la carpeta del bot:

```powershell
cd C:\USERS\LAMEG\BITUNIX-BOT\estrategia_elon_tweets
git init
git add .
git commit -m "bot v1"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/bot-polymarket-elon.git
git push -u origin main
```

> ⚠️ Sustituye `TU_USUARIO` por tu nombre de GitHub.
> 💡 `git add .` **NO** incluirá `config_real.json` (lo impide `.gitignore`). Puedes comprobarlo con `git status` — no debe aparecer ese archivo. Tus claves se quedan solo en tu PC.

## A.4 · Crea el token de acceso (PAT) para que el bot guarde estado

1. GitHub → avatar (arriba a la derecha) → **Settings**.
2. Barra izquierda (al final) → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
3. **Repository access**: *Only select repositories* → marca `bot-polymarket-elon`.
4. **Permissions** → **Contents** → **Read and write** (solo esto).
5. **Generate token** → **copia el token** (empieza por `github_pat_...`) — solo se muestra una vez.

## A.5 · Guarda tus secretos en el repo

En GitHub → tu repo → **Settings → Secrets and variables → Actions → New repository secret**. Crea estos 4 (¡y solo estos!):

| Nombre del secret | Valor que pones |
|---|---|
| `PAT_BOT` | el token del paso A.4 (`github_pat_...`) |
| `POLY_PRIVATE_KEY` | tu clave privada del firmante (`0x5a14...`) |
| `POLY_WALLET_ADDRESS` | tu dirección de Polymarket (`0xb0E1...`) |
| `REAL_CONFIRMADO` | `1` (para que opere de verdad) |

*Opcionales (solo si los tienes):* `POLY_RELAYER_API_KEY` y `POLY_RELAYER_API_KEY_ADDRESS`.

> 🔐 Los secrets GitHub los muestra como `***` — nadie puede verlos, ni tú (si los pierdes, bórralos y vuelve a crearlos).

## A.6 · Lanza una pasada de prueba

1. En el repo → pestaña **Actions**.
2. Verás el workflow **"Bot Polymarket Elon (real)"**.
3. **Run workflow** (botón) → espera ~1-2 minutos.
4. Comprueba el resultado:
   - Click en el run → **bot** → paso **"Ejecutar una pasada del bot (MODO REAL)"** → verás el log completo del bot.
   - Debe acabar con `Pasada completada en X s`.
   - Después, en el repo debe aparecer un **commit nuevo** "estado bot ..." (el paso "Guardar estado").

## A.7 · ¡Ya está! Así funciona a partir de ahora

```
cada 15 min → GitHub lanza el bot → recoge tweets → actualiza precios
→ evalúa señal 48h → (si cruza el umbral) envía orden REAL → commit del estado
```

- 📱 Las **notificaciones al móvil (ntfy)** funcionan igual (el tema está en `config.json`, que sí se sube — no es secreto).
- 📊 El **Excel** se regenera en cada pasada desde los CSV (que sí se versionan).
- 💰 Tu **dinero está en Polymarket**, no en GitHub: GitHub solo ejecuta el bot; las órdenes las firma tu clave (secret) y van directas a Polymarket.

## A.8 · Verificación rápida diaria (opcional, 30 segundos)

```powershell
# En GitHub: repo → Actions → ver que los runs son verdes (✓)
# Y en tu móvil: la app ntfy te llegan los avisos del bot
```

## A.9 · Límites y trucos

| Tema | Detalle |
|---|---|
| Coste | 0 €. Repo público = minutos ilimitados. |
| Retraso del cron | 1-10 min de retraso posible (irrelevante: nuestras señales duran horas) |
| Pausa por inactividad | GitHub pausa los cron si el repo está 60 días sin actividad → **nuestro bot hace commit cada 15 min → nunca se pausa** 😉 |
| Si falla una pasada | El bot reintenta en la siguiente; los errores van a `bot.log` (no se versiona) y, si son graves, te llega 🚨 al móvil |
| Cambiar a modo papel | Pon el secret `REAL_CONFIRMADO` con cualquier valor que no sea `1` (o bórralo) |

---

# VÍA B · Tu propio PC (la más simple si está siempre encendido)

## B.1 · Crea una Tarea programada en Windows (cada 15 min)

Abre PowerShell **como administrador** y pega (una sola línea):

```powershell
schtasks /create /tn "BotPoly" /tr "cmd /c cd /d C:\USERS\LAMEG\BITUNIX-BOT\estrategia_elon_tweets && python -u bot.py --modo real --excel >> bot.log 2>&1" /sc minute /mo 15 /f
```

Con esto, Windows ejecuta el bot cada 15 minutos. Para que funcione **aunque no estés mirando**:
- Deja el PC encendido y con **sesión iniciada** (o configúralo en *ejecutar tanto si el usuario inició sesión como si no*: Panel de control → Tareas programadas → BotPoly → Propiedades → marcar la opción y poner tu contraseña).
- Evita que el PC **se duerma**: Ajustes → Sistema → Energía → *Nunca*.

## B.2 · Comprueba que funciona

```powershell
Get-Content C:\USERS\LAMEG\BITUNIX-BOT\estrategia_elon_tweets\bot.log -Tail 30
```

Debe mostrar las pasadas con sus horas (cada 15 min).

## B.3 · Para pararlo

```powershell
schtasks /delete /tn "BotPoly" /f
```

---

# VÍA C · Oracle Cloud Always Free (servidor 24/7 de verdad)

Oracle te regala **para siempre** una máquina virtual Linux en la nube (Always Free). Pasos resumidos (detalle en `GUIA_24H_GRATIS.md`):

1. **oracle.com/cloud/free** → **Start for free** (te piden tarjeta SOLO para verificar; no cobran si te quedas en Always Free).
2. Consola → **Compute → Instances → Create** → Ubuntu 24.04 → shape *Always Free eligible* → SSH keys (descarga la clave).
3. Conéctate: `ssh -i tu_clave.key ubuntu@IP`
4. Instala y sube el bot:
   ```bash
   sudo apt update && sudo apt install -y python3-pip curl git
   pip install openpyxl py-clob-client-v2
   # sube la carpeta con scp, o clona tu repo de GitHub (Vía A)
   ```
5. Ejecuta 24/7 (sobrevive a cerrar la SSH):
   ```bash
   cd ~/estrategia_elon_tweets
   tmux new -s bot
   python3 -u bot.py --modo real --excel --loop --intervalo 15
   # Ctrl+B, D para despegar
   ```
6. Si usas modo real en Oracle, crea allí `config_real.json` (o usa variables de entorno).

---

## ✅ RECOMENDACIÓN FINAL

| Tu caso | Elige |
|---|---|
| El PC está casi siempre encendido | **Vía B** (5 min, sin GitHub) |
| No quieres depender del PC / máxima robustez gratis | **Vía A** (GitHub Actions, repo público) |
| Quieres un servidor dedicado 24/7 con la mejor latencia | **Vía C** (Oracle) |

**Plan sugerido**: empieza por la **Vía B** hoy mismo (5 minutos) para que el bot vigile sin que tengas que escribirlo a mano; cuando puedas, muévelo a la **Vía A** para que corra aunque apagues el PC.

---

## ❓ FAQ

**¿Y mis claves? ¿Están seguras en GitHub?**
Sí: van en *Secrets* (enmascarados, cifrados por GitHub) y `config_real.json` nunca se sube (`.gitignore`). El repo puede ser público sin riesgo: las claves no están en el código.

**¿El bot puede "perder" el estado entre ejecuciones de GitHub?**
No: cada pasada termina haciendo **commit** de `datos_elon.csv`, `estado_tweets.json`, `mercado_activo.json`, `resultados_*.csv`, etc. La siguiente ejecución hace *checkout* y continúa exactamente donde estaba.

**¿Cuánto tarda en aparecer la primera orden real con esto?**
Igual que en tu PC: cuando un bin de 48h cruce cuota ≥ 3.00 y p_modelo ≥ 60%. Puede tardar horas o días — es el diseño.

**¿Puedo ver el estado desde el móvil?**
Sí: (1) las notificaciones ntfy te avisan de cada evento, y (2) la app GitHub te avisa si un run falla (opcional, en Settings del repo → Notifications).

**¿Qué hago si un run falla?**
Mira el log del run (Actions → run fallido → paso rojo). Los fallos típicos: timeout de red (reintenta solo en la siguiente pasada) o un error de código (pégamelo y lo corrijo).

**¿Puedo dejar el bot en modo papel (sin dinero) en GitHub?**
Sí: simplemente no pongas el secret `REAL_CONFIRMADO` (o ponlo a `0`) y no pongas `POLY_PRIVATE_KEY`. El bot correrá en papel (el YAML usa `--modo real` pero sin `REAL_CONFIRMADO=1` el bot se bloquea y no envía nada… mejor: para papel, cambia en el YAML `--modo real` por `--modo papel`).

---

*Documento de gestión personal. No constituye asesoramiento financiero. Juega solo con dinero que puedas perder.*
