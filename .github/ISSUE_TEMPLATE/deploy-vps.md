---
name: Deploy progetto su VPS con HestiaCP
about: Guida operativa passo passo per configurare un progetto e il deploy via Git su HestiaCP
title: "[Deploy VPS] "
labels: deployment
assignees: ""
---

# Deploy progetto su VPS con HestiaCP

## Riferimenti

- [Documentazione ufficiale HestiaCP](https://hestiacp.com/)
- [setup-remote-git.sh](https://github.com/Slaitech/Documentazione/blob/main/scripts/setup-remote-git.sh)
- [deploy.sh](https://github.com/Slaitech/Documentazione/blob/main/scripts/deploy.sh)

---

# Configurazione di un progetto

## 1. Crea l'utente in HestiaCP

Per prima cosa crea un **utente** cliccando su **Nuovo utente** e inserisci i dati richiesti.

- [ ] Form compilato e utente creato

## 2. Modifica l'utente appena creato

Una volta creato l'utente, modificalo cliccando sull'icona a forma di matita.

In questa schermata devi:

- abilitare l'accesso SSH
- scegliere la shell `bash`
- selezionare la versione di PHP che l'utente userà da riga di comando

Attenzione: la versione PHP CLI dell'utente **non è necessariamente la stessa** che useranno i domini gestiti da quell'utente.

- [ ] Accesso SSH abilitato
- [ ] Shell impostata su `bash`
- [ ] PHP CLI configurato

## 3. Entra con l'utente creato

Adesso entra con quell'utente usando l'icona con la freccia.

Per capire con quale utente stai lavorando, controlla l'username in alto a sinistra vicino al logo Hestia.

- [ ] Accesso effettuato con l'utente corretto

## 4. Crea le risorse del progetto

Ora puoi creare tutto ciò che serve al progetto:

- dominio
- database

Prima crea almeno il **dominio** del progetto, perché servirà per il percorso di deploy remoto.

- [ ] Dominio creato
- [ ] Database creato, se necessario

---

# Deploy con git + script

## 0. Copia la chiave SSH sul server

Per prima cosa assicurati di aver copiato la chiave SSH sul server.

```sh
ssh-copy-id user@ip
```

- [ ] Chiave SSH copiata sul server

## 1. Rendi disponibili gli script nella root del progetto locale

Assicurati che questi due file siano nella root del progetto da deployare (filavel ce li ha out of the box):

- [setup-remote-git.sh](https://github.com/Slaitech/Documentazione/blob/main/scripts/setup-remote-git.sh)
- [deploy.sh](https://github.com/Slaitech/Documentazione/blob/main/scripts/deploy.sh)

Se non sono già presenti nel progetto, recuperali e copiali nella root del repository locale.

- [ ] I file sono nella root del progetto locale

## 2. Dai i permessi di esecuzione a `deploy.sh`

Prima del setup remoto, rendi eseguibile `deploy.sh`.

```sh
chmod +x deploy.sh
```

- [ ] Permessi di esecuzione assegnati a `deploy.sh`

## 3. Lancia `setup-remote-git.sh`

Avvia lo script dalla root del progetto locale.

```sh
sh ./setup-remote-git.sh
```

Compila le richieste e, alla fine, stai attento all'output del comando: ti darà una cosa del tipo

```sh
git remote add live USERNAME@SERVER_IP:/home/USERNAME/web/DOMAIN/private/site.git
```

Copia e lancia il comando generato realmente dallo script.

- [ ] `setup-remote-git.sh` eseguito
- [ ] Remote `live` aggiunto correttamente

## 4. Allinea il branch predefinito del repository bare remoto

Adesso collegati via SSH al server, entra nella cartella `private/site.git` e lancia questo comando:

```sh
git symbolic-ref HEAD refs/heads/main
```

- [ ] `git symbolic-ref HEAD refs/heads/main` eseguito

## 5. Esegui il primo push verso il server

Ora torna in locale, sempre nella root del progetto, e lancia:

```sh
git push -u live main
```

- [ ] Primo push eseguito

## 6. Cambia la document root in Hestia

Adesso torna dentro Hestia, apri il dominio del progetto, vai in **Advanced options** e cambia la document root impostando:

```text
public
```

- [ ] Document root impostata su `public`

## 7. Configurazione Crontab (Produzione)

Per eseguire i task di Laravel in produzione (scheduler, code worker e health queue), è necessario configurare il crontab sul server.

### Accesso e configurazione

1. **Accedi via SSH al server**
2. **Apri l'editor crontab**

   ```sh
   crontab -e
   ```

3. **Aggiungi le seguenti righe** (adattale ai tuoi percorsi effettivi):

   ```cron
   * * * * * /usr/bin/php8.4 /path/cartella/sito-web/artisan schedule:run >/dev/null 2>&1
   * * * * * /usr/bin/php8.4 /path/cartella/sito-web/artisan queue:work --stop-when-empty >/dev/null 2>&1
   * * * * * /usr/bin/php8.4 /path/cartella/sito-web/artisan queue:work --queue=health --stop-when-empty >/dev/null 2>&1
   ```

4. **Salva il file** (premi `Ctrl+X` su nano, oppure `:wq` su vi)

- [ ] Crontab configurato in produzione

---

# Verifiche finali

- [ ] Dominio raggiungibile
- [ ] Applicazione caricata correttamente
- [ ] Eventuali attività residue annotate
