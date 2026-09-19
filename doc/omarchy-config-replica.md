# Replicar la configuracion de waynergy en otra maquina Omarchy

Esta guia documenta la configuracion completa de waynergy usada en esta
maquina (ROG Zephyrus G14, Omarchy/Hyprland) como cliente de un servidor
Synergy 3 en macOS, para poder dejarla identica en otra instalacion de
Omarchy que se conecte al mismo servidor.

El motivo principal para escribir esto es el **teclado**: sin la seccion
`[raw-keymap]` de mas abajo, las teclas de texto funcionan pero los
atajos de Hyprland (Super+numero, etc.) fallan o disparan otra cosa
distinta a la esperada. Esto ya se investigo y resolvio una vez en esta
maquina — ver la explicacion tecnica dentro del propio `config.ini`.

## 0. Requisito: binario de waynergy con el fix de clipboard

Esta maquina NO usa el waynergy de los repos de Arch (`/usr/bin/waynergy`,
version 0.0.17 stock), porque esa version se desconecta (`EBAD`) al usar
el portapapeles contra un servidor Synergy 3 / Deskflow. En su lugar usa
un build propio, instalado en `~/.local/bin/waynergy` (que tiene
prioridad en el `PATH` sobre `/usr/bin/waynergy`).

Ese build sale de este mismo fork, rama `clipboard-protocol-fix` (o
`upstream-clipboard-fix`, que es la misma serie de commits sin el
material de documentacion de Omarchy). Ver tambien el PR upstream:
https://github.com/r-c-f/waynergy/pull/120

Para replicarlo en la maquina nueva:

```bash
git clone --branch clipboard-protocol-fix https://github.com/robertgarcia/waynergy.git
cd waynergy
meson setup build
ninja -C build
mkdir -p ~/.local/bin
cp build/waynergy ~/.local/bin/waynergy
```

Confirma que el `PATH` del usuario tiene `~/.local/bin` antes que
`/usr/bin` (asi viene por defecto en Omarchy). Verifica la version:

```bash
~/.local/bin/waynergy --version
# Deberia reportar algo como v0.0.17-7-ge6d6fb2, NO v0.0.17 a secas.
```

## 1. `~/.config/waynergy/config.ini`

Copia este archivo tal cual, con **dos cambios obligatorios**:

- `name`: debe ser **unico por maquina** (es como el servidor Synergy
  identifica cada pantalla). No reutilices `rogzephyrus-d658d482` en la
  maquina nueva — pon un nombre propio, por ejemplo el hostname.
- `host`: mantenlo igual solo si la maquina nueva se conecta al mismo
  Mac (`Robertos-MacBook-Pro.local`). Si es otro servidor, cambialo.

Todo lo demas — y en especial `[raw-keymap]`, que es la parte que
arregla el teclado — debe copiarse sin modificar, porque depende del
Mac origen (los keycodes crudos que manda), no de la maquina cliente.

```ini
; waynergy — cliente Synergy para Wayland.
; Servidor primario: Robertos-MacBook-Pro.local (192.168.31.78 al 2026-08-29), macOS.
;
; ENFOQUE: traducir los keycodes crudos del Mac a keycodes evdev LOCALES con
; [raw-keymap], en vez de subir un keymap xkb propio.
;
; Por qué: Hyprland envía a las APLICACIONES el keymap del teclado virtual, pero
; resuelve sus propios ATAJOS contra el keymap estándar (el teclado de waynergy
; sale como main:false en `hyprctl devices`). Con un keymap propio el texto salía
; bien pero los atajos no: Ctrl+3 mandaba keycode 28, que en evdev es 't', y
; disparaba SUPER+T (toggle flotante) en vez de SUPER+3.
; Con [raw-keymap] los keycodes que llegan YA son los estándar, así que texto y
; atajos usan el mismo mapa.
;
; El fichero xkb_keymap.mac-DESACTIVADO es el enfoque anterior, conservado por si
; hiciera falta volver. NO poner wl_keyboard_map = false: bloquea waynergy 0.0.17
; (wl_display_dispatch espera un evento que nunca llega con el escritorio quieto).

; Nombre mDNS en vez de IP: el Mac toma direccion por DHCP y va saltando
; entre .37 y .78, lo que rompia la conexion cada pocos dias. waynergy resuelve
; con getaddrinfo y /etc/nsswitch.conf tiene mdns_minimal, asi que esto sigue al
; Mac alla donde el router lo mande. Comprobar con: getent ahostsv4 Robertos-MacBook-Pro.local
host = Robertos-MacBook-Pro.local
port = 24800
name = CAMBIAR-POR-NOMBRE-UNICO-DE-ESTA-MAQUINA
backend = uinput

[tls]
enable = true
tofu = true

; Traducción: <código crudo del Mac> = <keycode evdev local>
; Generado cruzando doc/xkb/keycodes/mac de waynergy (offset 7) con
; /usr/share/X11/xkb/keycodes/evdev. Las dos últimas líneas son el intercambio
; Command<->Ctrl que pidió el usuario, con valores medidos en captura real:
;   Command -> DKDN id 61419 (Super_L),  raw 56
;   Ctrl    -> DKDN id 61411 (Control_L), raw 60
[raw-keymap]
1 = 38	; AC01
2 = 39	; AC02
3 = 40	; AC03
4 = 41	; AC04
5 = 43	; AC06
6 = 42	; AC05
7 = 52	; AB01
8 = 53	; AB02
9 = 54	; AB03
10 = 55	; AB04
12 = 56	; AB05
13 = 24	; AD01
14 = 25	; AD02
15 = 26	; AD03
16 = 27	; AD04
17 = 29	; AD06
18 = 28	; AD05
19 = 10	; AE01
20 = 11	; AE02
21 = 12	; AE03
22 = 13	; AE04
23 = 15	; AE06
24 = 14	; AE05
25 = 21	; AE12
26 = 18	; AE09
27 = 16	; AE07
28 = 20	; AE11
29 = 17	; AE08
30 = 19	; AE10
31 = 35	; AD12
32 = 32	; AD09
33 = 30	; AD07
34 = 34	; AD11
35 = 31	; AD08
36 = 33	; AD10
37 = 36	; RTRN
38 = 46	; AC09
39 = 44	; AC07
40 = 48	; AC11
41 = 45	; AC08
42 = 47	; AC10
43 = 51	; BKSL
44 = 59	; AB08
45 = 61	; AB10
46 = 57	; AB06
47 = 58	; AB07
48 = 60	; AB09
49 = 23	; TAB
50 = 65	; SPCE
51 = 49	; TLDE
52 = 22	; BKSP
54 = 9	; ESC
57 = 50	; LFSH
58 = 66	; CAPS
59 = 64	; LALT
66 = 91	; KPDL
68 = 63	; KPMU
70 = 86	; KPAD
72 = 77	; NMLK
76 = 106	; KPDV
77 = 104	; KPEN
79 = 82	; KPSU
83 = 90	; KP0
84 = 87	; KP1
85 = 88	; KP2
86 = 89	; KP3
87 = 83	; KP4
88 = 84	; KP5
89 = 85	; KP6
90 = 79	; KP7
92 = 80	; KP8
93 = 81	; KP9
97 = 71	; FK05
98 = 72	; FK06
99 = 73	; FK07
100 = 69	; FK03
101 = 74	; FK08
102 = 75	; FK09
104 = 95	; FK11
106 = 107	; PRSC
108 = 78	; SCLK
110 = 76	; FK10
112 = 96	; FK12
114 = 127	; PAUS
115 = 118	; INS
116 = 110	; HOME
117 = 112	; PGUP
118 = 119	; DELE
119 = 70	; FK04
120 = 115	; END
121 = 68	; FK02
122 = 117	; PGDN
123 = 67	; FK01
124 = 113	; LEFT
125 = 114	; RGHT
126 = 116	; DOWN
127 = 111	; UP
56 = 37	; Command del Mac -> Control_L
60 = 133	; Ctrl del Mac    -> Super_L
```

No es necesario dejar `no-clip = true` en `[raw-keymap]`: es una linea
suelta e inerte que quedo de una config anterior (el nombre de la
opcion real es `no-clip` bajo la seccion general, no dentro de
`[raw-keymap]`), y con el binario parcheado del paso 0 el clipboard ya
funciona sin necesidad de desactivarlo. No la repliques.

## 2. Unidades systemd `--user`

Crea estos cuatro archivos en `~/.config/systemd/user/`.

### `waynergy.service`

```ini
[Unit]
Description=waynergy — cliente Synergy nativo para Wayland
Documentation=https://github.com/r-c-f/waynergy
# Sustituye a synergy.service (enmascarado): Synergy 3 no puede inyectar
# entrada en Hyprland — no hay portal RemoteDesktop y XTEST vía XWayland
# no llega al compositor.
After=graphical-session.target
PartOf=graphical-session.target

[Service]
Type=simple
# Configuración completa en ~/.config/waynergy/config.ini
ExecStart=/home/rgarcia/.local/bin/waynergy
Restart=always
RestartSec=3

# waynergy no responde a SIGTERM con prontitud y deja hijos wl-paste
# sujetando el cgroup: sin esto, `systemctl stop` tardaba 90 s y acababa
# en SIGKILL con 'Failed with result timeout'.
KillMode=control-group
TimeoutStopSec=5
# El socket del portapapeles queda rancio si el proceso muere de forma brusca.
ExecStopPost=/usr/bin/rm -f /run/user/1000/waynergy-clip-sock

[Install]
WantedBy=graphical-session.target
```

> Ajusta la ruta de `ExecStart` y el UID en `ExecStopPost`
> (`/run/user/1000/...`) si el usuario en la maquina nueva tiene otro
> nombre o UID.

### `waynergy-monitor-watch.service`

```ini
[Unit]
Description=Reinicia waynergy cuando cambia la disposición de monitores
Documentation=man:hyprland(1)
After=graphical-session.target waynergy.service
PartOf=graphical-session.target

[Service]
Type=simple
ExecStart=%h/.local/bin/waynergy-monitor-watch
Restart=always
RestartSec=3

[Install]
WantedBy=graphical-session.target
```

### `waynergy-connection-watch.service`

```ini
[Unit]
Description=Vigilante de conexion de waynergy
Documentation=file:%h/.local/bin/waynergy-connection-watch
# Complementa a Restart=always de waynergy.service, que no sirve aqui: el bug
# de reconexion de waynergy 0.0.17 deja el proceso vivo pero inutil, y systemd
# solo reinicia lo que muere.
After=graphical-session.target
PartOf=graphical-session.target

[Service]
Type=oneshot
ExecStart=%h/.local/bin/waynergy-connection-watch
```

### `waynergy-connection-watch.timer`

```ini
[Unit]
Description=Comprueba cada minuto que waynergy sigue conectado
PartOf=graphical-session.target

[Timer]
OnActiveSec=1min
OnUnitActiveSec=1min
# Sin esto systemd agrupa disparos con hasta 1 min de holgura, que sobre un
# intervalo de 1 min significaria comprobar cada dos.
AccuracySec=5s
Unit=waynergy-connection-watch.service

[Install]
WantedBy=graphical-session.target
```

## 3. Scripts auxiliares en `~/.local/bin/`

### `waynergy-monitor-watch`

Reinicia waynergy cuando cambia la disposicion de monitores de
Hyprland (necesario porque waynergy cachea la lista de `wl_output` al
arrancar y se rompe si un monitor desaparece, por ejemplo al cerrar la
tapa en modo clamshell).

```bash
#!/bin/bash
# Reinicia waynergy cuando cambia la disposición de monitores de Hyprland.
#
# Por qué: waynergy cachea la lista de wl_output al arrancar y mapea contra ella
# el movimiento absoluto del puntero. Cuando un output desaparece (modo clamshell
# al cerrar la tapa, o desenchufar el HDMI) escupe "Could not find xdg output" en
# bucle y el ratón deja de moverse. El teclado sobrevive porque
# zwp_virtual_keyboard no depende de ningún output.
#
# Escucha .socket2.sock, pero NO decide por el nombre del evento: al despertar
# compara una huella de `hyprctl monitors` con la anterior y solo reinicia si de
# verdad cambió. Así da igual qué evento exacto emita Hyprland al deshabilitar un
# output, y un `hyprctl reload` que no toca monitores no provoca reinicios.

set -uo pipefail

DEBOUNCE=2          # segundos de calma tras un evento antes de evaluar
POLL_TICK=0.25      # granularidad del bucle de lectura
POLL_INTERVAL=15    # sondeo de seguridad, por si un cambio no emite evento
RECONNECT_DELAY=3   # espera antes de reintentar si el socket no está

fingerprint() {
  hyprctl monitors -j 2>/dev/null |
    jq -Sc '[.[] | {name, x, y, width, height, scale}]' 2>/dev/null
}

socket_path() {
  local runtime="${XDG_RUNTIME_DIR:-/run/user/$(id -u)}"
  local sig="${HYPRLAND_INSTANCE_SIGNATURE:-}" candidate

  if [[ -n $sig && -S $runtime/hypr/$sig/.socket2.sock ]]; then
    printf '%s\n' "$runtime/hypr/$sig/.socket2.sock"
    return 0
  fi

  # La firma cambia en cada arranque de Hyprland; si la heredada ya no vale,
  # quedarse con el socket vivo más reciente.
  candidate=$(ls -1dt "$runtime"/hypr/*/.socket2.sock 2>/dev/null | head -1)
  [[ -S $candidate ]] || return 1
  printf '%s\n' "$candidate"
}

maybe_restart() {
  local current
  current=$(fingerprint)

  # Una huella vacía significa que Hyprland no respondió, no que no haya
  # monitores: no tomarla como un cambio.
  [[ -n $current ]] || return 0
  [[ $current == "$LAST" ]] && return 0

  echo "monitores: $LAST -> $current"
  LAST=$current

  # Si el usuario lo paró a mano, no resucitarlo.
  if systemctl --user is-active --quiet waynergy.service; then
    echo "reiniciando waynergy"
    systemctl --user restart waynergy.service
  else
    echo "waynergy no está activo: no lo toco"
  fi
}

# El debounce se mide desde el evento, NO como silencio en el socket: socket2 es
# un flujo continuo de eventos de ventana (títulos, foco), así que esperar a que
# calle entero significa no evaluar nunca. Por eso el read usa un tick corto y la
# decisión la toma el reloj.
#
# Devuelve 1 cuando el flujo de eventos termina, para que el bucle exterior reconecte.
listen() {
  local line rc pending=0 deadline=0
  local next_poll=$(( EPOCHSECONDS + POLL_INTERVAL ))

  while :; do
    IFS= read -r -t "$POLL_TICK" line
    rc=$?
    if (( rc == 0 )); then
      if [[ $line == monitor* || $line == configreloaded* ]]; then
        (( pending )) || deadline=$(( EPOCHSECONDS + DEBOUNCE ))
        pending=1
      fi
    elif (( rc <= 128 )); then
      return 1  # EOF: Hyprland se fue o el socket murió (>128 es timeout del read).
    fi

    if (( pending && EPOCHSECONDS >= deadline )); then
      pending=0
      next_poll=$(( EPOCHSECONDS + POLL_INTERVAL ))
      maybe_restart
    elif (( EPOCHSECONDS >= next_poll )); then
      # Red de seguridad: comprobado que reposicionar un monitor con
      # `hyprctl eval` (que es como lo hace omarchy-hyprland-monitor-clamshell)
      # no emite ningún evento de monitor, solo open/closelayer de la barra. El
      # camino por eventos no basta por sí solo.
      next_poll=$(( EPOCHSECONDS + POLL_INTERVAL ))
      maybe_restart
    fi
  done
}

LAST=$(fingerprint)
echo "arrancado; monitores iniciales: $LAST"

while :; do
  if ! sock=$(socket_path); then
    sleep "$RECONNECT_DELAY"
    continue
  fi
  listen < <(socat -u UNIX-CONNECT:"$sock" -)
  echo "flujo de eventos cerrado; reconectando en ${RECONNECT_DELAY}s"
  sleep "$RECONNECT_DELAY"
done
```

Requiere `jq` y `socat` instalados (`omarchy pkg add jq socat` si
faltan).

### `waynergy-connection-watch`

Rescata a waynergy cuando se cuelga sin morir (bug de reconexion de la
0.0.17: si el Mac se duerme o Synergy se reinicia, el cliente se queda
vivo pero inutil, y `Restart=always` no ayuda porque el proceso no
muere).

```bash
#!/bin/bash
# Rescata a waynergy cuando se cuelga sin morir.
#
# Por que: waynergy 0.0.17 tiene un fallo de reconexion. Si el servidor Synergy
# del Mac desaparece un momento (el Mac se duerme, o Synergy se reinicia), el
# cliente NO muere: se queda girando en un bucle gastando 10-30% de CPU, sin
# reconectar y sin escribir un log mas. `Restart=always` no lo rescata porque
# systemd solo reinicia procesos que mueren, y este sigue vivo pero inutil.
#
# El sintoma que se nota primero no es Synergy, es que el puntero del raton
# deja de dibujarse en pantalla: el proceso atascado retiene su puntero virtual
# y el compositor deja de pintar el cursor. Reiniciar el servicio lo cura.
#
# Regla de actuacion: reiniciar SOLO si el puerto del servidor esta abierto y
# aun asi no hay conexion establecida. Si el Mac esta apagado o dormido no hay
# nada que arreglar y este script se calla — no tiene sentido reiniciar en bucle
# contra un servidor que no existe.

set -uo pipefail

CONFIG="${WAYNERGY_WATCH_CONFIG:-$HOME/.config/waynergy/config.ini}"
UNIT="waynergy.service"
GRACE=45            # s de cortesia tras arrancar, antes de juzgarlo colgado
MAX_RESTARTS=5      # tope de reinicios...
WINDOW=3600         # ...en esta ventana de tiempo (s)
STATE="${WAYNERGY_WATCH_STATE:-${XDG_RUNTIME_DIR:-/run/user/$(id -u)}/waynergy-watch.restarts}"

log() { echo "$*"; }

# Si el servicio no esta activo, no es asunto nuestro: o la sesion grafica no
# existe, o el usuario lo paro a proposito.
systemctl --user is-active --quiet "$UNIT" || exit 0

host=$(sed -n 's/^[[:space:]]*host[[:space:]]*=[[:space:]]*\([^[:space:];#]*\).*/\1/p' "$CONFIG" | head -1)
port=$(sed -n 's/^[[:space:]]*port[[:space:]]*=[[:space:]]*\([^[:space:];#]*\).*/\1/p' "$CONFIG" | head -1)
port=${port:-24800}
if [[ -z $host ]]; then
  log "no pude leer 'host' de $CONFIG; nada que vigilar"
  exit 0
fi

# ¿Hay conexion establecida al servidor? Si la hay, todo en orden.
if ss -tnH state established "dst $host:$port" 2>/dev/null | grep -q .; then
  exit 0
fi

# Sin conexion. Dale margen si acaba de arrancar: los primeros segundos son
# normales, y sin esto entrariamos en un bucle de reinicios.
started=$(systemctl --user show "$UNIT" -p ActiveEnterTimestampMonotonic --value 2>/dev/null)
now=$(awk '{printf "%d", $1 * 1000000}' /proc/uptime)
if [[ -n ${started:-} && $started -gt 0 ]]; then
  age=$(( (now - started) / 1000000 ))
  (( age < GRACE )) && exit 0
fi

# ¿Esta el servidor realmente ahi? Si el Mac no responde, no hay nada que
# reiniciar: waynergy tiene razon en no estar conectado.
if ! timeout 3 bash -c "exec 3<>/dev/tcp/$host/$port" 2>/dev/null; then
  exit 0
fi

# Puerto abierto y sin conexion: este es el estado patologico.
# Antes de actuar, comprueba que no llevamos ya demasiados reinicios.
cutoff=$(( $(date +%s) - WINDOW ))
recent=$(awk -v c="$cutoff" '$1 > c' "$STATE" 2>/dev/null)
count=$(printf '%s' "$recent" | grep -c . || true)
if (( count >= MAX_RESTARTS )); then
  log "$host:$port acepta conexiones pero waynergy no conecta, y ya van $count reinicios en la ultima hora: me detengo. Revisa a mano."
  exit 0
fi

log "$host:$port abierto pero sin conexion establecida: waynergy parece colgado, reiniciando (reinicio $((count + 1)) de $MAX_RESTARTS en esta hora)"
{ [[ -n $recent ]] && printf '%s\n' "$recent"; date +%s; } > "$STATE"
systemctl --user restart "$UNIT"
```

Dale permiso de ejecucion a ambos:

```bash
chmod +x ~/.local/bin/waynergy-monitor-watch ~/.local/bin/waynergy-connection-watch
```

## 4. Activar todo

```bash
systemctl --user daemon-reload
systemctl --user enable --now waynergy.service
systemctl --user enable --now waynergy-monitor-watch.service
systemctl --user enable --now waynergy-connection-watch.timer
```

Verifica:

```bash
systemctl --user status waynergy.service waynergy-monitor-watch.service waynergy-connection-watch.timer
journalctl --user -u waynergy.service -f
```

## 5. Icono opcional en la barra de Omarchy

Si tambien quieres el icono de la barra para iniciar/parar/reiniciar el
servicio desde el escritorio, sigue
[`omarchy-bar-icon.md`](omarchy-bar-icon.md) en este mismo directorio.
