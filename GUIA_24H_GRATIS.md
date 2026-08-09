# ⏰ GUÍA — Funcionamiento 24 HORAS SIN PARAR con métodos GRATUITOS

**Objetivo:** que el bot (papel o dinero real) corra 24/7 sin pagar servidores.

---

## 🆚 Comparativa de opciones gratuitas

| Opción | ¿24/7 real? | Coste | Complejidad | Ideal para |
|---|---|---|---|---|
| **A · GitHub Actions** ⭐ | Sí (cada 15 min) | 0 € (repo público = minutos ilimitados; privado = 2.000 min/mes) | Media | Casi todo el mundo; sin servidor que mantener |
| **B · Oracle Cloud Always Free** ⭐ | Sí (VPS real 24/7) | 0 € (para siempre) | Media-Alta | Quien quiere un servidor de verdad y control total |
| **C · Tu propio PC encendido** | Sí (si no se apaga) | 0 € (electricidad) | Baja | Pruebas y uso personal ligero |
| **D · Hugging Face Spaces + UptimeRobot** | Sí (con pings) | 0 € | Alta | Avanzados; el estado se resetea (complicado) |
| Render / PythonAnywhere gratis | No (duermen) | 0 € | Baja | ❌ No sirven para 24/7 real |

**Recomendación:** empieza con **A (GitHub Actions)** — es la que menos mantenimiento requiere. Si algún día quieres un servidor 24/7 de verdad, **B (Oracle)**.

---

# OPCIÓN A · GitHub Actions (recomendada) ⭐

GitHub ejecuta gratis tus "trabajos" en la nube cada vez que se lo pides. El truco: un **cron cada 15 minutos** lanza una pasada del bot, y el bot **guarda su estado haciendo commit** al final de cada pasada (así no "olvida" nada entre ejecuciones).

### A.1 · Crea el repositorio
1. Entra en **github.com** → **New repository**.
2. Nombre: `bot-polymarket-elon` (o el que quieras).
3. **VISIBILIDAD: PÚBLICO** (recomendado → minutos ilimitados y el cron nunca se pausa por inactividad). Si lo haces privado: tienes 2.000 min/mes (≈ 96 pasadas/día × ~1,5 min ≈ 4.300 min/mes → **no llega**; tendrías que espaciar a cada 30–60 min). Lo público no es un problema: **nunca subas `config_real.json`** (ya está en `.gitignore`) y las claves van en los *secrets*.
4. **No** inicialices con README (para subir limpio).

### A.2 · Sube la carpeta del bot
```bash
cd /ruta/de/tu/ordenador
git init
git remote add origin https://github.com/TU_USUARIO/bot-polymarket-elon.git
git add .
git commit -m "bot v1"
git branch -M main
git push -u origin main
```
> El repositorio ya incluye `.github/workflows/bot.yml` (el archivo que creamos) y `.gitignore` (que excluye `config_real.json` y los `.xlsx`).

### A.3 · Crea el token de acceso (PAT) para que el bot pueda guardar estado
1. GitHub → tu avatar → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
2. Permisos: solo **Contents → Read and write** de ese repositorio.
3. Copia el token (empieza por `github_pat_...`).

### A.4 · Guarda los secretos en el repo
En GitHub → tu repo → **Settings → Secrets and variables → Actions → New repository secret**:

| Nombre del secret | Valor |
|---|---|
| `PAT_BOT` | el token del paso A.3 |
| `POLY_API_KEY` | *(solo modo real)* tu API key de Polymarket |
| `POLY_API_SECRET` | *(solo modo real)* |
| `POLY_API_PASSPHRASE` | *(solo modo real)* |

### A.5 · Actívalo y compruébalo
1. En el repo, pestaña **Actions** → verás el workflow `Bot Polymarket Elon`.
2. Ejecútalo una vez a mano: **Actions → Bot Polymarket Elon → Run workflow** (botón a la derecha).
3. Espera ~1 minuto y comprueba que el run termina en ✅ y que hay un commit nuevo "estado bot ...".
4. A partir de ahí, GitHub lo ejecuta **cada 15 minutos** automáticamente.

### A.6 · ¿Y el dinero real?
El workflow ya pasa las claves `POLY_*` como variables de entorno al bot. Para activar el modo real edita el **final del YAML** (o usa un secret `REAL_CONFIRMADO=1`... más simple: crea en el repo el secret `REAL_CONFIRMADO` con valor `1`). El bot leerá `REAL_CONFIRMADO=1` desde el entorno → actúa como `confirmado: true`.

> ⚠️ **Antes de poner `REAL_CONFIRMADO=1`**, valida todo en modo seco (GUIA_DINERO_REAL.md pasos 6-7).

### A.7 · Límites y trucos que debes saber
- **Minutos**: público = ilimitados; privado = 2.000/mes (→ espacia el cron a 30–60 min o pasa a público).
- **Retraso**: los cron de GitHub pueden llegar con 1–10 min de retraso. Para nuestra estrategia (señales que duran horas) es irrelevante.
- **Inactividad**: GitHub pausa los cron si el repo lleva **60 días sin actividad**. Nuestro bot hace commit cada 15 min → nunca se pausa. 😉
- **Horario**: el cron usa UTC; no importa, el bot usa hora ET internamente.
- **Coste**: 0 €. Solo necesitas una cuenta GitHub (gratis).

---

# OPCIÓN B · Oracle Cloud Always Free (VPS 24/7) ⭐

Oracle te regala **para siempre** una máquina virtual en la nube (2 instancias ARM o 1 AMD). Es un servidor Linux real, encendido 24/7.

### B.1 · Crea la cuenta
1. Ve a **oracle.com/cloud/free** → **Start for free**.
2. Te pedirán tarjeta de crédito para verificar identidad. **No te cobran nada** mientras te quedes en el plan Always Free (te avisan si vas a consumir algo de pago; puedes poner alertas de coste a $0).
3. Completa la verificación (tarjeta + móvil). Listo.

### B.2 · Crea la instancia
1. Consola Oracle → **Compute → Instances → Create instance**.
2. Imagen: **Ubuntu 24.04** (o 22.04).
3. Shape: elige la opción **Always Free eligible** (AMD E2.1.Micro 1 OCPU/1 GB RAM es suficiente para nuestro bot).
4. SSH keys: **Generate a key pair** → descarga la clave privada (`.key`) y guárdala bien.
5. Create. Espera a que el estado sea *Running* (1-2 min).
6. Anota la **IP pública** de la instancia.

### B.3 · Conéctate y prepara
```bash
chmod 600 tu_clave.key
ssh -i tu_clave.key ubuntu@IP_PUBLICA
# dentro del servidor:
sudo apt update && sudo apt install -y python3-pip curl git
pip install openpyxl py-clob-client
```

### B.4 · Sube el bot y ejecútalo
```bash
# desde tu ordenador (otra terminal):
scp -i tu_clave.key -r /ruta/estrategia_elon_tweets ubuntu@IP_PUBLICA:~
# dentro del servidor:
cd ~/estrategia_elon_tweets
# (si vas a usar dinero real: crea config_real.json aquí, con nano)
tmux new -s bot            # abre una sesión persistente
python3 bot.py --loop --intervalo 15 --excel
# desconecta sin matarlo: Ctrl+B y luego D
# para volver a verlo: tmux attach -t bot
```
Ahora el bot corre **24/7** aunque cierres la SSH. Si reinicias el servidor, vuelve a abrir tmux y lanza el bot (o usa `cron` como en C.2 — más robusto).

### B.5 · Avisos importantes
- **Firewall**: la instancia solo necesita SSH (puerto 22). No abras más puertos.
- **Instancias inactivas**: Oracle puede **reclamar** (parar) una instancia Always Free si está **7 días seguidos sin uso**. Nuestro bot genera actividad continua cada 15 min → normalmente no pasa. Si algún día la ves *Stopped*, solo dale a *Start*.
- **Seguridad**: no guardes `config_real.json` con claves si no es imprescindible; mejor usa variables de entorno.
- **Copias de seguridad**: descarga de vez en cuando `datos_elon.csv`, `resultados_*.csv` y el Excel (scp) por si la instancia se pierde.

---

# OPCIÓN C · Tu propio PC encendido

Si tu ordenador está encendido 24/7 (o un mini-PC / Raspberry Pi), es lo más sencillo.

### C.1 · Windows (Tarea programada)
```bat
:: Un archivo arrancar_bot.bat:
cd /d C:\ruta\estrategia_elon_tweets
python bot.py --excel >> bot.log 2>&1
```
1. **Win+R** → `taskschd.msc` → **Crear tarea básica**.
2. Disparador: **Diariamente** a las 00:00 → y luego **editar la tarea** → pestaña *Disparadores* → **Nuevo** → *Repetir cada 15 minutos* durante *1 día* (y "Detener la tarea si dura más de... desmarcado").
3. Acción: programa `python.exe`, argumentos `bot.py --excel`, iniciar en `C:\ruta\estrategia_elon_tweets`.
4. Marca **"Ejecutar tanto si el usuario inició sesión como si no"** y dale la contraseña (o usa "Solo cuando el usuario está conectado" si no quieres).

### C.2 · Linux / Mac (cron)
```bash
crontab -e
# añade (cada 15 minutos):
*/15 * * * * cd /ruta/estrategia_elon_tweets && /usr/bin/python3 bot.py --excel >> bot.log 2>&1
```
Para modo continuo con loop y que sobreviva al cierre de terminal:
```bash
tmux new -s bot
python3 bot.py --loop --intervalo 15 --excel
# Ctrl+B, D para despegar
```

> ⚠️ El PC debe **no dormirse**: ajusta el plan de energía a "nunca dormir" (Windows) o usa `caffeinate` (Mac).

---

# OPCIÓN D · Hugging Face Spaces + UptimeRobot (avanzada)

1. Crea un **Space** en huggingface.co (tipo **Docker**, visibilidad privada, CPU free).
2. Sube el bot con un `Dockerfile` que ejecute `python3 bot.py --loop --intervalo 15 --excel`.
3. **UptimeRobot** (uptimerobot.com, gratis hasta 50 monitores) → monitor HTTP cada **5 minutos** a la URL de tu Space (ej. `https://tu-space.hf.space`) para que **no se duerma** (los Spaces free se apagan a las 48 h sin actividad).
4. ⚠️ **El estado vive en el disco del Space y se pierde al reconstruir** → para persistir necesitas que el bot haga commit a GitHub (como en opción A) o guardar en un almacenamiento externo. Por eso esta opción es la más frágil: úsala solo si sabes lo que haces.

---

## 🎯 RECOMENDACIÓN FINAL

| Tu caso | Elige |
|---|---|
| Lo más simple y robusto, sin tarjeta de crédito | **A · GitHub Actions** (repo público) |
| Quieres un servidor 24/7 de verdad (modo real con órdenes en milisegundos, sin retrasos de cron) | **B · Oracle Cloud Always Free** |
| Pruebas rápidas / ya tienes un PC siempre encendido | **C · Tu PC** |
| Curioso/avanzado | **D · HF Spaces** |

**Plan sugerido:** empieza con **C** (hoy mismo, 5 minutos) para validar el bot en modo papel; después muévelo a **A** (GitHub Actions) para que corra gratis 24/7 aunque apagues el PC; y cuando quieras pasar a dinero real con la máxima fiabilidad, súbelo a **B** (Oracle) con `--loop --intervalo 15`.

---

## ❓ FAQ

**¿GitHub Actions cobra algo?** No. Repo público = minutos ilimitados. Repo privado = 2.000 min/mes gratis.

**¿Puedo usar los secrets para el modo real?** Sí: `POLY_API_KEY`, `POLY_API_SECRET`, `POLY_API_PASSPHRASE`, `REAL_CONFIRMADO=1`.

**¿Qué pasa si falla una pasada?** El bot reintenta solo en la siguiente; los errores quedan en `bot.log` y, si es grave (fallan las fuentes), te llega 🚨 al móvil.

**¿Cuánto tarda en notarse la primera operación real?** Depende de la señal: el bot solo apuesta cuando un bin de 48 h cumple cuota ≥ 3.00 y p_modelo ≥ 60%. Pueden pasar días sin señal — eso es normal y correcto.

**¿Los datos se pierden entre ejecuciones en GitHub Actions?** No: el bot hace commit de su estado (CSV, JSON) en cada pasada. El siguiente run hace checkout y continúa donde estaba.

---

*Documento de gestión personal. No constituye asesoramiento financiero.*
