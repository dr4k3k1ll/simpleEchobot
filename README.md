# 🔒 Protezione del Token Telegram con Variabili d'Ambiente

Questa guida spiega come gestire il **token Telegram in modo sicuro**, evitando di esporlo su GitHub.

---

## 📌 Perché proteggere il Token?
Salvare direttamente il **BOT_TOKEN** nel codice è pericoloso perché:
- **Se pubblichi il codice su GitHub, chiunque può vedere e usare il token.**
- **Migliora la sicurezza del progetto.**
- **Evita di dover aggiornare il codice ogni volta che il token cambia.**

---

## ✅ 1. Impostare il Token come Variabile d'Ambiente

### 🔹 Su Windows Apri il terminale:
- PowerShell: 
```powershell
$env:BOT_TOKEN="tuo_token_telegram"
```
 - Prompt dei comandi:
```cmd
set BOT_TOKEN=tuo_token_telegram
```
### NOTA!!!⚠️ Questi comandi funzionano solo per la sessione corrente, se ```CMD``` o ```Powershell``` vengono chiusi e poi riaperti la variabile d'ambiente verrà cancellata.
### Se si vuole che il valore della variabile d'ambiente sia salvata in modo permanente su Windows:
```powershell
[System.Environment]::SetEnvironmentVariable("BOT_TOKEN", "tuo_token_telegram", "User")
```
### 🔹 Su Linux/macOS:
```bash
# Variabile d'ambiente valida solo per la sessione attuale:
export BOT_TOKEN="tuo_token_telegram"

# Per renderla permanente, aggiungila a ~/.bashrc o ~/.bash_profile:
echo 'export BOT_TOKEN="tuo_token_telegram"' >> ~/.bashrc
source ~/.bashrc
```
### Verifica se la variabile d'ambiente è stata impostata correttamente:
### 🔹Su Windows con:
- Powershell:
```powershell
echo $env:BOT_TOKEN
```
- Prompt dei comandi:
```cmd
echo %BOT_TOKEN%
```
### 🔹 Su Linux/macOS:
```bash
echo $BOT_TOKEN
```
### Se il comando non stampa nulla, significa che la variabile non è stata impostata correttamente.
### Dopo aver impostato il token, puoi eseguire lo script:
```bash
python bot.py
```
### ATTENZIONE!!!⚠️: Prima di avviare il bot su powershell o su cmd bisogna assicurarsi di essere dentro la cartella di destinazione di ```bot.py```
```bash
cd /percorso/del/progetto
python bot.py

```
---
## ✅ 2. Usare il Token nel Codice Python
### Nel tuo script ```bot.py```, usa questo codice:
```python
import os
from telegram.ext import Application
# -----------------------------
#             code
# -----------------------------
def main() -> None:
    """Start the bot."""
    TOKEN = os.getenv("BOT_TOKEN")
    if not TOKEN:
    	raise ValueError("ERRORE: BOT_TOKEN non trovato. Assicurati di averlo impostato nelle variabili d'ambiente!")
    application = Application.builder().token(TOKEN).build()
# -----------------------------
#           end code
# -----------------------------
```
### 🔹 Se la variabile non è impostata, il programma restituirà un errore.
---
## ✅ 3. Nascondere il Token su GitHub con .env
### Puoi anche usare un file ```.env``` per gestire le variabili:
### 1 Crea un file ```.env``` nella cartella del progetto:
```ini
BOT_TOKEN=tuo_token_telegram
```
### 2 Installa ```python-dotenv```
```bash
pip install python-dotenv
```
### 3 Modifica il codice Python per leggere ```.env```:
```python
from dotenv import load_dotenv, find_dotenv
import os
# Verifica la presenza del file .env prima di caricarlo
if not find_dotenv():
    raise FileNotFoundError("⚠️ ERRORE: File .env non trovato!")
# -----------------------------
#             code
# -----------------------------
def main() -> None:
    """Start the bot."""
    load_dotenv() # Carica le variabili dal file .env
    TOKEN = os.getenv("BOT_TOKEN")
    if not TOKEN:
    	raise ValueError("ERRORE: BOT_TOKEN non trovato!")
    application = Application.builder().token(TOKEN).build()
# -----------------------------
#           end code
# -----------------------------
```
### 4 Aggiungi ```.env``` a ```.gitignore``` per non caricarlo su GitHub:
```bash
# File .gitignore
.env
```
### Se il file ```.gitignore``` non esiste va creato manualmente:
```bash
echo ".env" >> .gitignore
```
---
## ✅ 4. Controllare che il Token NON sia su GitHub
### Prima di fare il commit, verifica che il token non sia già stato salvato:
```bash
git grep "BOT_TOKEN"
```
### Se trovi il token nei commit passati, rimuovilo immediatamente e rigenera un nuovo token su ```@BotFather```.
### Se hai già committato il token, rimuovilo dalla cronologia Git con:
```bash
pip install git-filter-repo  # Installa lo strumento per modificare la cronologia Git

# Rimuove ogni traccia del file .env dalla cronologia dei commit
git filter-repo --path .env --invert-paths

# Dopo la pulizia della cronologia, devi forzare il push delle modifiche:
git push origin --force --all
```
---
## ✅ 5. Usare GitHub Actions con i Secrets
### Se vuoi usare il bot su GitHub Actions, imposta il token come secret:

### 1 Vai su GitHub > Settings > Secrets and variables > Actions.
### 2 Clicca "New repository secret" e chiamalo ```BOT_TOKEN```, inserendo il valore.
### 3 Nel tuo ```.github/workflows/bot.yml```, usa:
```yamlname: Avvio Bot Telegram

on:
  push:
    branches:
      - main  # Esegui l'azione solo quando viene pushato sul branch principale

jobs:
  run-bot:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.9"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Avvia il bot
        env:
          BOT_TOKEN: ${{ secrets.BOT_TOKEN }}
        run: python bot.py
```
