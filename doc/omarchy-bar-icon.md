# Icono de waynergy en la barra de Omarchy

Este documento explica como agregar un icono en la barra de estado de
[Omarchy](https://omarchy.org/) (Hyprland + Quickshell) para iniciar,
detener o reiniciar `waynergy.service` sin usar la terminal.

Usa el tipo de modulo `"command"` que la barra de Omarchy ya soporta de
forma nativa en `~/.config/omarchy/shell.json` (no requiere un plugin QML
aparte). Ver el README de referencia en
`/usr/share/omarchy/shell/plugins/bar/README.md` para el detalle del
formato de estos modulos.

Este flujo asume que `waynergy` corre como servicio de usuario systemd
(`waynergy.service`). Si todavia no lo tienes asi, primero crea/activa
ese servicio antes de continuar.

## 1. Scripts

Crea el directorio de scripts si no existe:

```bash
mkdir -p ~/.config/omarchy/bar/scripts
```

### `~/.config/omarchy/bar/scripts/waynergy-status`

Imprime el estado en el formato JSON que espera la barra
(`{"text","tooltip","class"}`, donde `class:"active"` resalta el icono):

```bash
#!/bin/bash
# Estado de waynergy.service para el modulo "command" de la barra de Omarchy.
if systemctl --user is-active --quiet waynergy.service; then
	printf '{"text":"⌨","tooltip":"waynergy: activo\\nclic = detener · clic derecho = reiniciar","class":"active"}'
else
	printf '{"text":"⌨","tooltip":"waynergy: detenido\\nclic = iniciar · clic derecho = reiniciar","class":""}'
fi
```

### `~/.config/omarchy/bar/scripts/waynergy-toggle`

Clic izquierdo: alterna el servicio (lo detiene si esta activo, lo
arranca si no):

```bash
#!/bin/bash
# Alterna waynergy.service: lo para si esta activo, lo arranca si no.
if systemctl --user is-active --quiet waynergy.service; then
	systemctl --user stop waynergy.service
	notify-send -a waynergy -i input-keyboard "waynergy" "Detenido"
else
	systemctl --user start waynergy.service
	notify-send -a waynergy -i input-keyboard "waynergy" "Iniciado"
fi
```

### `~/.config/omarchy/bar/scripts/waynergy-restart`

Clic derecho: reinicia el servicio:

```bash
#!/bin/bash
systemctl --user restart waynergy.service
notify-send -a waynergy -i input-keyboard "waynergy" "Reiniciado"
```

Dales permiso de ejecucion a los tres:

```bash
chmod +x ~/.config/omarchy/bar/scripts/waynergy-status \
          ~/.config/omarchy/bar/scripts/waynergy-toggle \
          ~/.config/omarchy/bar/scripts/waynergy-restart
```

## 2. Entrada en `shell.json`

Edita `~/.config/omarchy/shell.json` y agrega una entrada en la seccion
de la barra donde quieras que aparezca el icono (por ejemplo
`bar.layout.right`, junto al tray):

```json
{
  "id": "waynergy",
  "type": "command",
  "exec": "~/.config/omarchy/bar/scripts/waynergy-status",
  "interval": 3,
  "tooltip": "waynergy",
  "onClick": "~/.config/omarchy/bar/scripts/waynergy-toggle",
  "onRightClick": "~/.config/omarchy/bar/scripts/waynergy-restart"
}
```

`shell.json` se recarga en caliente al guardar: el icono deberia
aparecer solo, sin reiniciar Hyprland ni la sesion.

## 3. Uso

- **Clic izquierdo**: inicia o detiene `waynergy.service`.
- **Clic derecho**: reinicia el servicio.
- El texto del icono (⌨) y su tooltip cambian segun el estado, y se
  refrescan cada 3 segundos (`interval`).

## Nota tecnica

Al guardar `shell.json`, Omarchy recarga toda la barra y `Bar.qml` puede
imprimir una advertencia benigna, una sola vez, del tipo:

```
TypeError: Cannot assign to read-only property "moduleName"
```

Esto ocurre porque el componente generico `CustomCommandModule` declara
`moduleName`/`settings` como `readonly`, pero la funcion interna
`injectProps()` de la barra intenta asignarlas para *cualquier* modulo
tipo `"command"` (no es especifico de este icono). No se repite en cada
ciclo de sondeo y no afecta el funcionamiento del widget.
