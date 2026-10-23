<div align="center">

# check-CVE-2026-109619

Detector de exposição à PHP2SHELL CVE-2026-109619 em aplicações PHP.

[![Python](https://img.shields.io/badge/python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Dependencies](https://img.shields.io/badge/dependencies-zero-success?style=flat-square)](#-instalação)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

</div>

---

## Sobre

Ferramenta de triagem para identificar se um endpoint PHP é candidato explorável à CVE-2026-109619. Verifica os pré-requisitos necessários para a cadeia de exploração, sem tentar executar o exploit.

Feita para priorizar alvos rapidamente em pentests e inventários de exposição.

---

## Instalação

```bash
git clone https://github.com/fellipgomes/check-CVE-2026-109619.git
cd check-CVE-2026-109619
python3 check-CVE-2026-109619.py --help
```

**Requisitos:** Python 3.8+. Sem dependências externas.

---

## Uso

**Triagem padrão (segura):**
```bash
python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php
```

**Modo agressivo (requer autorização):**
```bash
python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php --aggressive
```

**Saída JSON para automação:**
```bash
python3 check-CVE-2026-109619.py -u https://site.com/login.php --json
```

**Alvo autenticado:**
```bash
python3 check-CVE-2026-109619.py -u https://app.site.com/dashboard.php --cookie "PHPSESSID=abc123" -v
```

### Flags

| Flag | Descrição | Padrão |
|---|---|---|
| `-u, --url` | URL do alvo | — |
| `--aggressive` | Sondagem mais profunda | desligado |
| `--oom-mb` | Profundidade do modo agressivo | `10` |
| `--cookie` | Cookies (alvos autenticados) | — |
| `--json` | Relatório em JSON | desligado |
| `-v, --verbose` | Log detalhado | desligado |
| `--timeout` | Timeout por requisição | `15s` |

---

## Veredictos

| Veredicto | Significado |
|---|---|
| `SAFE` | Não apresenta o perfil de risco. |
| `POSSIBLY_VULNERABLE` | Vale investigar mais. Rode `--aggressive -v`. |
| `LIKELY_VULNERABLE` | Candidato forte. Escalar a análise. |
| `NOT_PHP` | Endpoint errado ou alvo atrás de WAF. |

Nenhum veredicto substitui um pentest completo. A ferramenta é triagem.

---

## Exemplo de saída

<details>
<summary>Clique para expandir</summary>

```
[*] Target: https://staging.example.com/index.php
[*] Mode: aggressive
[*] Passive fingerprint...
    [.] X-Powered-By reveals PHP: PHP/8.2.18
[*] Active check...
    [.] session.upload_progress likely ON
    [.] display_errors: ON
[*] Aggressive check...
    [.] memory_limit = 128M

==============================================================
REPORT
==============================================================
  PHP detected           : True
  PHP version            : 8.2.18
  session.upload_progress: on
  display_errors         : on
  Score                  : 10
  VERDICT                : LIKELY_VULNERABLE
==============================================================
```
</details>

---

## 👤 Autor

<div align="center">

### **Fellip Melo**

**AppSec & Pentester · 9 anos de experiência**

*"Segurança não é um produto, é um processo. Meu trabalho é fazer esse processo funcionar desde a primeira linha de código até o último deploy."*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Fellip%20Melo-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/fellipmelo/)
[![GitHub](https://img.shields.io/badge/GitHub-fellipgomes-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fellipgomes)

</div>

Sou Fellip Melo, engenheiro de segurança de aplicações com nove anos de experiência. Sou apaixonado por CyberSecurity e tenho como objetivo contribuir para a proteção e a inovação dos sistemas e das aplicações.

**O que eu faço:**

- 🔐 **Secure development** — shift-left, design by security, SAST, DAST, IAST e SCA em pipelines DevSecOps
- 💳 **Pagamentos e e-commerce** — LGPD, GDPR, PCI-DSS, ISO 27001
- 🎯 **Pentest** — Web, Mobile (Android/iOS) e Cloud
- ⚙️ **Operação** — Akamai CDN e Veracode

**Vamos conversar?** Me chama no [LinkedIn](https://www.linkedin.com/in/fellipmelo/) — sempre aberto a trocar ideia sobre AppSec, pentest e segurança de aplicações.

---

## ⚖️ Disclaimer

Publicada para pesquisa em segurança, defesa e testes autorizados.

- ✅ Ambientes que você controla, pentests com escopo assinado, bug bounty autorizado
- ❌ Sistemas de terceiros sem autorização por escrito
- 📜 O autor não se responsabiliza por uso indevido ou violações legais

O `--aggressive` pode causar `500` transitório. Em produção, prefira o modo padrão ou rode contra `staging`.

---

## 📄 Licença

MIT. Veja [`LICENSE`](LICENSE) para o texto completo.
