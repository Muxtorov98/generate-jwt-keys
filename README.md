# generate-jwt-keys

- generate-jwt-keys.sh qilib saqlab, loyihaning root papkasiga qo‘yiladi

```bash
#!/usr/bin/env bash
set -euo pipefail

### Ranglar
BLUE='\033[0;34m'
GREEN='\033[0;32m'
RED='\033[0;31m'
RESET='\033[0m'

### Helper funksiyalar
progress_bar() {
  local duration=${1:-10}
  local i

  echo -ne "${BLUE}Progress: ["
  for ((i=0; i<duration; i++)); do
    echo -ne "#"
    sleep 0.05
  done
  echo -e "]${RESET}"
}

success() {
  echo -e "${GREEN}✔ $1${RESET}"
}

error() {
  echo -e "${RED}✖ $1${RESET}"
  exit 1
}

### Loyihaning root papkasi (skript turgan joyga qarab)
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$SCRIPT_DIR"

ENV_FILE=".env"

if [ ! -f "$ENV_FILE" ]; then
  error ".env fayli topilmadi. Iltimos, .env faylni loyihaning root papkasiga qo'ying."
fi

### JWT_PASSPHRASE ni .env dan olish
JWT_PASSPHRASE=$(grep -E '^JWT_PASSPHRASE=' "$ENV_FILE" | cut -d '=' -f2- | tr -d '"')

if [ -z "${JWT_PASSPHRASE:-}" ]; then
  error "JWT_PASSPHRASE .env faylida topilmadi."
fi

KEY_DIR="api/config/jwt"
PRIVATE_KEY="$KEY_DIR/private.pem"
PUBLIC_KEY="$KEY_DIR/public.pem"

echo -e "${BLUE}Generating JWT keys...${RESET}"
progress_bar 20

if [ ! -f "$PRIVATE_KEY" ]; then
    mkdir -p "$KEY_DIR"

    # Private key
    openssl genpkey \
        -algorithm RSA \
        -pkeyopt rsa_keygen_bits:4096 \
        -aes256 \
        -out "$PRIVATE_KEY" \
        -pass pass:"$JWT_PASSPHRASE"

    # Public key
    openssl rsa \
        -pubout \
        -in "$PRIVATE_KEY" \
        -out "$PUBLIC_KEY" \
        -passin pass:"$JWT_PASSPHRASE"

    success "JWT keys were successfully generated."
else
    success "JWT keys already exist."
fi

```

### Ishga tushirish

```bash
chmod +x generate-jwt-keys.sh
./generate-jwt-keys.sh
```

## Ahamiyatli joylari:
	- .env ichida shunday qatorday bo‘lishi kerak:

```env
JWT_PASSPHRASE=nimadur_maxfiy_parol
```

