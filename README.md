<div align="center">

# check-CVE-2026-109619

**Descubra em segundos se um site é candidato à vulnerabilidade mais comentada do ano no PHP.**

🪶 Zero dependências · ⚡ Um comando · 🛡️ Seguro por padrão

[![Python](https://img.shields.io/badge/python-3.8%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Dependencies](https://img.shields.io/badge/dependencies-zero-success?style=flat-square)](#-instalação)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](#-contribuindo)

[O que faz](#-o-que-faz) ·
[Instalação](#-instalação) ·
[Uso](#-uso-em-30-segundos) ·
[Autor](#-autor) ·
[Disclaimer](#%EF%B8%8F-disclaimer)

</div>

---

## 🎯 O que faz

Você aponta para qualquer site PHP. A ferramenta responde, em segundos:

> **"Esse alvo é candidato explorável — ou é apenas mais um site que não vai a lugar nenhum?"**

Sem instalar nada. Sem configurar nada. Sem depender de SDK. **Um comando, um veredicto.**

Feita para quem faz pentest de verdade: você tem 40 alvos no escopo e 3 dias para priorizar. Essa ferramenta te dá o mapa em minutos, não em horas.

---

## ✨ Por que ela é diferente

| | |
|---|---|
| 🪶 **Zero dependências** | Um arquivo `.py`. Copia, cola, roda. |
| 🛡️ **Seguro por padrão** | Nenhum payload agressivo. Nenhum crash. |
| 💥 **Modo `--aggressive` sob demanda** | Quando você tem autorização e quer ir mais fundo. |
| 📊 **Saída JSON** | Integra em CI/CD, relatórios e dashboards. |
| 🌐 **Segue redirects** | Pega o endpoint real mesmo quando o alvo redireciona. |
| 🧠 **Veredicto claro** | `SAFE`, `POSSIBLY_VULNERABLE`, `LIKELY_VULNERABLE` — sem caixa-preta. |
| 🚫 **Sem falso positivo** | Se o alvo não é PHP, ele diz `NOT_PHP`. Simples assim. |

---

## 📦 Instalação

```bash
git clone https://github.com/fellipgomes/check-CVE-2026-109619.git
cd check-CVE-2026-109619
python3 check-CVE-2026-109619.py --help
```

**Requisitos:** Python 3.8+. Nada mais. Sem `pip install`, sem venv, sem `requirements.txt`.

---

## 🚀 Uso em 30 segundos

### Modo padrão — seguro, sem risco

```bash
python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php
```

Roda a triagem completa sem derrubar nada. Você recebe um relatório com o veredicto e as notas do que foi encontrado.

### Modo agressivo — quando você tem autorização

```bash
python3 check-CVE-2026-109619.py -u https://www.site-alvo.com/index.php --aggressive
```

Vai um passo além e revela mais detalhes do ambiente do alvo. Recomendado apenas em `staging` ou com escopo assinado.

### Saída JSON — para automatizar

```bash
python3 check-CVE-2026-109619.py -u https://staging.sua-empresa.com/login.php --json
```

Perfeito para pipelines de segurança, gates de CI/CD e inventário contínuo de exposição.

### Alvo autenticado

```bash
python3 check-CVE-2026-109619.py -u https://app.alvo.com.br/dashboard.php \
  --cookie "PHPSESSID=abc123" -v
```

---

## 🎛️ Flags

| Flag | O que faz | Padrão |
|---|---|---|
| `-u, --url` | URL do alvo (obrigatório) | — |
| `--aggressive` | Ativa a sondagem mais profunda | desligado |
| `--oom-mb` | Ajusta a profundidade do modo agressivo | `10` |
| `--cookie` | Envia cookies (alvos autenticados) | — |
| `--json` | Relatório em JSON para automação | desligado |
| `-v, --verbose` | Mostra cada passo da análise | desligado |
| `--timeout` | Tempo máximo por requisição | `15s` |

---

## 📊 Os veredictos

| Veredicto | O que significa |
|---|---|
| 🟢 **`SAFE`** | O alvo não apresenta o perfil de risco. Pode seguir para o próximo. |
| 🟡 **`POSSIBLY_VULNERABLE`** | Vale a pena investigar mais. Rode `--aggressive -v` para confirmar. |
| 🔴 **`LIKELY_VULNERABLE`** | Forte candidato. É hora de escalar a análise. |
| ⚪ **`NOT_PHP`** | Endpoint errado ou alvo atrás de WAF. Tente outra URL. |

> ⚠️ **Nenhum veredicto substitui um pentest completo.** Esta ferramenta é triagem — o primeiro filtro que economiza horas de trabalho manual.

---

## 🖥️ Exemplo de saída

<details>
<summary><b>Clique para expandir</b></summary>

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

**Fellip Melo** — AppSec & Pentester · 9 anos de experiência

*Engenheiro de segurança de aplicações apaixonado por CyberSecurity, com o objetivo de contribuir para a proteção e a inovação dos sistemas e das aplicações.*

</div>

Atuação:

- **Secure development** — shift-left, design-by-security, SAST, DAST, IAST e SCA em pipelines DevSecOps.
- **Pagamentos e e-commerce** — LGPD, GDPR, PCI-DSS, ISO 27001.
- **Pentest** — Web, Mobile (Android/iOS) e Cloud.
- **Operação** — Akamai CDN e Veracode.

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Fellip%20Melo-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/fellipmelo/)
[![GitHub](https://img.shields.io/badge/GitHub-fellipgomes-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/fellipgomes)

</div>

---

## 🤝 Contribuindo

PRs são bem-vindos. Abra uma issue descrevendo o caso de uso antes de mandar código — evita trabalho duplicado.

```bash
git checkout -b feature/minha-feature
git commit -m "feat: descrição"
git push origin feature/minha-feature
```

---

## ⚖️ Disclaimer

Publicada para **pesquisa em segurança, defesa e testes autorizados**.

- ✅ Ambientes que você controla, pentests com escopo assinado, programas de bug bounty que autorizem esse tipo de teste.
- ❌ Sistemas de terceiros sem autorização por escrito e verificável.
- 📜 O autor não se responsabiliza por uso indevido, danos colaterais ou violações legais.

> O `--aggressive` pode causar um `500` transitório no alvo. Em produção, prefira o modo padrão ou rode contra `staging`.

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja [`LICENSE`](LICENSE) para o texto completo.

<div align="center">

<sub>Feito com café, <code>socket.socket()</code> e a teimosia de não usar <code>requests</code>.</sub>

</div>
