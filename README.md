<div align="center">

# check-CVE-2026-109619

**Detector não-destrutivo para o Use-After-Free em `session.upload_progress` do PHP.**

Zero dependências · Zero SDKs · Apenas a stdlib do Python

[![Python](https://img.shields.io/badge/python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Dependencies](https://img.shields.io/badge/dependencies-zero-success?style=flat-square)](#-instalação)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-research-orange?style=flat-square)](#)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](#-contribuindo)

[Visão geral](#-visão-geral) ·
[Uso](#-uso) ·
[Como funciona](#-como-funciona) ·
[Veredictos](#-como-ler-o-veredicto) ·
[Autor](#-autor) ·
[Disclaimer](#%EF%B8%8F-disclaimer)

</div>

---

## 📖 Visão geral

`check-CVE-2026-109619.py` responde a uma única pergunta:

> **Esse endpoint PHP é um candidato explorável para a cadeia de UAF do `session.upload_progress` — ou é apenas mais uma página que faz `session_start()` e devolve `404`?**

Diferente de PoCs de exploit — que dependem de offsets exatos do `libphp.so`, de um único worker `prefork` e de `display_errors=On` — esta ferramenta **verifica os pré-requisitos** sem precisar conhecer nada do servidor, sem derrubar o alvo e sem atirar no escuro.

É a ferramenta que eu queria ter nas mãos quando comecei a investigar essa classe de bug em pentests reais, onde o alvo é um domínio qualquer atrás de CDN e você não tem acesso ao `.so` do PHP.

### Por que não uma PoC de exploit?

A cadeia completa de exploração depende de **quatro condições simultâneas** que raramente se alinham em alvos desconhecidos:

| Condição | Por que importa |
|---|---|
| `display_errors = On` | Sem isso, o endereço não vaza no `Set-Cookie`. Estágio 1 morre. |
| `memory_limit` baixo | Se não for estourável pelo `--huge-mb` do exploit, não há OOM e o ponteiro nunca fica pendente. |
| Um worker `prefork` (mod_php) | O estado corrompido vive **naquele** processo. Múltiplos workers = heap se perde. |
| Offsets do `libphp.so` | `zif_system` / `zval_ptr_dtor` mudam por versão, arquitetura, distro e flags de compilação. |

Esta ferramenta checa as **três primeiras**. A quarta exige acesso ao binário do alvo.

---

## ✨ Destaques

- 🪶 **Zero dependências** — só `socket`, `ssl`, `re`, `json`, `argparse`.
- 🛡️ **Seguro por padrão** — nenhum payload de OOM, nenhum crash. Modo passivo + semi-ativo leve.
- 💥 **`--aggressive` sob demanda** — sonda de 10 MB para inferir `post_max_size` e `memory_limit` via mensagens de erro (só com autorização).
- 📊 **Saída JSON** — integra em pipelines DevSecOps / SAST-DAST gates.
- 🌐 **Segue redirects** (até 3 hops, mesma origem) — captura o falso negativo #1 de scanners que não seguem `Location`.
- 🧠 **Heurística auditável** — sem caixa-preta. Toda pontuação é explicada em `notes`.
- 🔍 **Diagnóstico em `-v`** — headers, redirect chain, bytes de resposta.
- 🚫 **Sem CVE-fantasma** — `NOT_PHP` é um resultado válido, não um falso positivo para "ter entregável".

---

## 📦 Instalação

```bash
git clone https://github.com/fellipgomes/check-CVE-2026-109619.git
cd check-CVE-2026-109619
python3 check-CVE-2026-109619.py --help

Requisitos: Python 3.8+. Nada mais. Sem pip install, sem venv, sem requirements.txt.
🚀 Uso
1. Triagem segura (modo padrão)
bash

python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php

Executa:

    GET passivo (redirects seguidos, mesma origem)

    POST multipart pequeno com PHP_SESSION_UPLOAD_PROGRESS

    POST com 1200 campos (dispara max_input_vars sem OOM)

Nenhum worker é derrubado.
2. Sondagem agressiva (com autorização)
bash

python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php --aggressive

Adiciona probe de 10 MB. Se display_errors=On, revela post_max_size e memory_limit no relatório — dados que te permitem calibrar --huge-mb do exploit.
3. Saída JSON para CI/CD
bash

python3 check-CVE-2026-109619.py -u https://staging.sua-empresa.com/login.php --json

Exemplo de gate em GitHub Actions:
yaml

- name: PHP session UAF check
  run: |
    python3 check-CVE-2026-109619.py -u "${{ vars.STAGING_URL }}" --json > out.json
    grep -q '"verdict": "LIKELY_VULNERABLE"' out.json && exit 1 || exit 0

4. Alvo autenticado
bash

python3 check-CVE-2026-109619.py -u https://app.alvo.com.br/dashboard.php \
  --cookie "PHPSESSID=abc123; other=xyz" -v

Flags
Flag	Descrição	Default
-u, --url	URL do endpoint PHP (obrigatório)	—
--timeout	Timeout por requisição	15s
--aggressive	Executa probe de OOM	false
--oom-mb	Tamanho do campo no modo agressivo	10 MB
--cookie	Header Cookie para alvos autenticados	—
--json	Emite relatório em JSON	false
-v, --verbose	Log detalhado (headers, chain, decisões)	false
🔬 Como funciona
<details> <summary><b>1. Fingerprint passivo</b></summary>

GET com redirect seguido (máx. 3 hops, mesma origem). Coleta:

    Server, X-Powered-By

    Set-Cookie: PHPSESSID=...

    Marcadores de PHP no corpo (Fatal error:, PHP Warning:, PHP Notice:, <b>Warning</b>)

Detecta também WAF/challenge (Cloudflare, Akamai, Sucuri, captcha, JS challenge).
</details><details> <summary><b>2. Probe de <code>session.upload_progress</code></b></summary>

POST multipart pequeno com PHP_SESSION_UPLOAD_PROGRESS + arquivo. Observa se PHPSESSID é emitido em resposta ao POST (não ao GET). Se sim, o progress report está ativo.

⚠️ Ressalva honesta: frameworks com ErrorDocument 404 /erro.php ou session_start() global em todo request emitem PHPSESSID mesmo com session.upload_progress=Off. Por isso o score não confirma sozinho.
</details><details> <summary><b>3. Probe de <code>display_errors</code></b></summary>

POST com 1200 campos (acima do max_input_vars=1000 padrão). Se display_errors=On, a mensagem "Input variables exceeded 1000" aparece no corpo — o mesmo mecanismo que faz o leak da PoC funcionar.
</details><details> <summary><b>4. Probe agressivo (opcional)</b></summary>

POST de 10 MB. Dispara post_max_size (padrão 8M) e, se display_errors=On, revela memory_limit na mensagem de OOM (Allowed memory size of X bytes exhausted).
</details>
📊 Como ler o veredicto
Veredicto	Score	Significado	Próximo passo
NOT_PHP	—	Nenhum marcador PHP encontrado.	Endpoint errado ou alvo atrás de WAF.
SAFE	0–1	PHP presente, mas session.upload_progress e display_errors desligados.	Cadeia inviável. Documentar e seguir.
UNLIKELY	2–4	PHP + algum pré-requisito, mas falta o essencial.	Investigar log de erro (se tiver acesso).
POSSIBLY_VULNERABLE	5–6	Dois pré-requisitos batem. Ainda não é confirmação.	Rodar --aggressive -v para afinar.
LIKELY_VULNERABLE	7+	PHP + session.upload_progress + display_errors=On + apache2handler (ou memória baixa).	Extrair libphp.so, calcular offsets, tentar PoC de exploit.

    ⚠️ Nenhum veredicto é prova de RCE. É triage, não exploit. LIKELY_VULNERABLE significa "vale investigar mais", não "alvo comprometido".

🖥️ Exemplo de saída
<details> <summary><b>Clique para expandir</b></summary>
text

[*] Target: https://staging.example.com/index.php
[*] Mode: aggressive
[*] Passive fingerprint...
    [.] PHPSESSID cookie present in initial response
    [.] X-Powered-By reveals PHP: PHP/8.2.18
[*] Active check: session.upload_progress...
    [.] POST+PHP_SESSION_UPLOAD_PROGRESS -> PHPSESSID set (session.upload_progress likely ON)
[*] Active check: display_errors (input_vars overflow)...
    [.] display_errors: ON (max_input_vars=1000 reported in response)
[*] Aggressive check: 10MB field (infer post_max_size / memory_limit)...
    [.] post_max_size = 8M (from error text)
    [.] memory_limit = 128M (from error text)

==============================================================
REPORT
==============================================================
  URL                    : https://staging.example.com/index.php
  Status                 : 200
  Server                 : Apache/2.4.58 (Debian)
  X-Powered-By           : PHP/8.2.18
  PHP detected           : True
  PHP version            : 8.2.18
  SAPI (guess)           : apache2handler (likely mod_php or proxy_fcgi)
  PHPSESSID cookie       : True
  session.upload_progress: on
  display_errors         : on
  max_input_vars hint    : 1000
  post_max_size hint     : 8M
  memory_limit hint      : 128M
  Score                  : 10
  VERDICT                : LIKELY_VULNERABLE
==============================================================

</details>
👤 Autor
<div align="center">

Fellip Melo — AppSec & Pentester · 9 anos de experiência

Engenheiro de segurança de aplicações apaixonado por CyberSecurity, com o objetivo de contribuir para a proteção e a inovação dos sistemas e das aplicações.
</div>

Atuação:

    Secure development — shift-left, design-by-security, SAST, DAST, IAST e SCA em pipelines DevSecOps.

    Pagamentos e e-commerce — LGPD, GDPR, PCI-DSS, ISO 27001.

    Pentest — Web, Mobile (Android/iOS) e Cloud.

    Operação — Akamai CDN e Veracode.

<div align="center">

https://img.shields.io/badge/LinkedIn-Fellip%20Melo-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white
https://img.shields.io/badge/GitHub-fellipgomes-181717?style=for-the-badge&logo=github&logoColor=white
</div>
🤝 Contribuindo

PRs são bem-vindos. Se quiser adicionar:

    Detecção de SAPI via probe mais específico (ex.: distinguir mod_php de php-fpm pelo header Set-Cookie path)

    Detector de phpinfo.php exposto

    Suporte a proxies autenticados

    Melhor heurística para session.upload_progress (probe de leitura de sessão em dois endpoints)

Abra uma issue descrevendo o caso de uso antes do PR — evita trabalho duplicado.
bash

git checkout -b feature/minha-feature
git commit -m "feat: descrição"
git push origin feature/minha-feature

⚖️ Disclaimer

Esta ferramenta é publicada para pesquisa em segurança, defesa e testes autorizados.

    ✅ Ambientes que você controla, pentests com escopo assinado, programas de bug bounty que autorizem esse tipo de teste.

    ❌ Sistemas de terceiros sem autorização por escrito e verificável.

    📜 O autor não se responsabiliza por uso indevido, danos colaterais ou violações legais.

    O --aggressive pode causar um 500 transitório no alvo (o worker prefork se recupera, mas o log fica marcado). Em produção, prefira o modo padrão ou rode contra staging.

📄 Licença

Distribuído sob a licença MIT. Veja LICENSE para o texto completo.
<div align="center">

<sub>Feito com café, <code>socket.socket()</code> e a teimosia de não usar <code>requests</code>.</sub>
</div> ```
