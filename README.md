#!/bin/bash
# Script para configurar Copyparty + Git con commits automáticos

# --- Variables ---
BASE="$HOME/gitserver"
DOCS="$BASE/docs"
REPOS="$BASE/repos"
COPYPARTY="$BASE/copyparty"
LOGS="$BASE/copyparty/logs"
CONF="$COPYPARTY/the.conf"
USER="juan"
PASS="1234"
PORT=3923
AUTO_COMMIT_INTERVAL=300  # segundos (5 minutos)

# --- Crear estructura de carpetas ---
mkdir -p "$DOCS" "$REPOS" "$COPYPARTY" "$LOGS"

# --- Inicializar Git en docs si no existe ---
if [ ! -d "$DOCS/.git" ]; then
  cd "$DOCS"
  git init
  echo "Servidor Git+Copyparty listo" > README.txt
  git add .
  git commit -m "Primer commit: estructura inicial"
fi

# --- Crear archivo de configuración the.conf ---
cat > "$CONF" <<EOL
[global]
  p: $PORT
  e2dsa
  e2ts
  z, qr
  dedup
  df: 1
  no-robots
  theme: 2
  lang: spa

[accounts]
  $USER: $PASS

[/]
  .
  accs:
    rwmda: $USER

[/docs]
  $DOCS
  accs:
    rwmda: $USER

[/repos]
  $REPOS
  accs:
    rwmda: $USER
EOL

# --- Función para commits automáticos ---
auto_commit() {
  while true; do
    cd "$DOCS"
    if [ -n "$(git status --porcelain)" ]; then
      git add .
      git commit -m "Auto-commit desde Copyparty $(date +"%Y-%m-%d %H:%M:%S")"
    fi
    sleep $AUTO_COMMIT_INTERVAL
  done
}

# --- Arrancar Copyparty en segundo plano con log ---
cd "$COPYPARTY"
python3 copyparty-sfx.py -c "$CONF" > "$LOGS/copyparty.log" 2>&1 &

# --- Arrancar auto_commit en segundo plano ---
auto_commit &

echo "Copyparty y Git auto-commit corriendo!"
echo "Logs en $LOGS/copyparty.log"
echo "Usuario: $USER | Contraseña: $PASS"
echo "Carpeta docs: $DOCS"
echo "Carpeta repos: $REPOS"
