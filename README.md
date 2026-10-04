# 🎣 Investigação de Phishing e Resposta a Incidentes (NIST CSF)

## 📖 1. Introdução e Cenário Fictício
*   **Empresa Afetada:** Almeida Compliance Solutions (Consultoria Jurídica)
*   **O Alerta:** O gateway de segurança de e-mail identificou uma falha grave de autenticação em uma mensagem urgente direcionada ao Diretor Financeiro.
*   **O Objetivo:** Analisar o cabeçalho bruto da mensagem para comprovar a fraude (Email Spoofing) e documentar as ações de mitigação com base no framework **NIST CSF**.

---

## 🔬 2. Análise Forense da Evidência Técnica
Abaixo está o cabeçalho bruto (Header) coletado do e-mail suspeito recebido:

```text
Delivered-To: diretor.financeiro@almeidacompliance.com
Received: by 20.112.44.12 with SMTP id p12csp2340578; Sun, 04 Oct 2026 07:59:12 -0700
Return-Path: <spoof-attacker@secure-microsoft-update-alert.com>
From: "Microsoft Security Team" <no-reply@microsoft.com>
Subject: ACTION REQUIRED: Critical Security Alert - Reset Your Corporate Password Immediately
Authentication-Results: ://google.com;
       spf=fail (google.com: domain of no-reply@microsoft.com does not designate 192.0.2.55 as authorized sender)
       dkim=fail (signature verification failed);
       dmarc=fail (p=reject sp=none dis=none) header.from=microsoft.com
X-Malicious-URL: http://com-security-update.net
```

### 🔍 Explicação Didática dos Mecanismos de Defesa:

1.  **Email Spoofing (Falsificação):** O campo `From:` mostra visualmente `no-reply@microsoft.com` para induzir a vítima ao erro. No entanto, o `Return-Path` (caminho real de retorno) aponta para um servidor malicioso externo: `spoof-attacker@secure-microsoft-update-alert.com`.
2.  **Falha de SPF (Sender Policy Framework):** O teste retornou `spf=fail`. Isso significa que o endereço IP do servidor remetente (`192.0.2.55`) **não consta** na lista oficial de servidores autorizados pela Microsoft para enviar e-mails em seu nome.
3.  **Falha de DKIM (DomainKeys Identified Mail):** O teste retornou `dkim=fail`. Indica que o e-mail não possui uma assinatura digital criptografada válida, provando que a integridade da mensagem foi violada ou forjada.
4.  **Falha de DMARC:** Como consequência das falhas anteriores de SPF e DKIM, o alinhamento de DMARC falhou (`dmarc=fail`), o que disparou o alerta no SOC para a equipe de segurança.
5.  **Análise da URL Oculta:** O link contido no e-mail direcionava para o domínio `://microsoft.com-security-update.net`. Através de análise reputacional estática (via ferramentas como VirusTotal), confirmou-se que se trata de uma infraestrutura maliciosa recente voltada para **roubo de credenciais (Credential Harvesting)**.

---

## 🛡️ 3. Plano de Resposta a Incidentes (Alinhado ao NIST CSF)

Seguindo as fases estruturadas do Guia de Tratamento de Incidentes de Segurança de Computadores do NIST, as seguintes ações foram documentadas:

### 1. Detecção e Análise (Detection & Analysis)
*   Isolamento e extração dos Indicadores de Comprometimento (IOCs): Domínio malicioso e IP do atacante.
*   Varredura nos logs de acesso para verificar se houve digitação de credenciais ou acessos suspeitos na conta corporativa afetada.

### 2. Contenção, Erradicação e Recuperação (Containment, Eradication & Recovery)
*   **Bloqueio de Infraestrutura:** Inserção do domínio malicioso na lista de bloqueios (Blocklist) do firewall e do gateway de e-mail corporativo.
*   **Erradicação do Vetor:** Remoção forçada do e-mail malicioso de todas as caixas de correio da empresa para evitar que outros colaboradores cliquem no link.
*   **Remediação de Identidade:** Bloqueio preventivo da sessão do usuário afetado, seguido de redefinição mandatória de senha e auditoria dos métodos de Autenticação Multi-Fator (MFA).

### 3. Atividades Pós-Incidente (Post-Incident Activity)
*   Revisão de regras de acesso condicional para bloquear tentativas de autenticação vindas de localizações geográficas incomuns.
*   Desenvolvimento de campanhas de conscientização focadas em Phishing e Engenharia Social para mitigar riscos futuros no ambiente de trabalho.
