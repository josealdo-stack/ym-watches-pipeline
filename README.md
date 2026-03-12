# YM Watches — Sistema de Pipeline via WhatsApp

Pipeline com Claude Code para gerenciar um negócio de venda de relógios de luxo pelo WhatsApp. Varre o histórico de conversas, classifica oportunidades usando o framework de infraestrutura simbólica de Michel Alcoforado e gera dashboards visuais para acompanhamento.

---

## O Que Faz

**Varredura de Oportunidades** (`workflows/opportunity-scan.md`)
- Executa 3 varreduras paralelas de palavras-chave em mais de 1.000 DMs do WhatsApp (preço/negociação, sinais de adiamento, sinais de rapport/status)
- Classifica cada contato em 10 categorias de oportunidade (Tier 1 Quente → Tier 3 Ângulos Alcoforado)
- Gera um dashboard HTML no estilo WhatsApp com bolhas de conversa, ângulos de reengajamento e abordagens em português prontas para enviar

**Workflow de Reengajamento** (`workflows/reengagement-workflow.md`)
- Triagem diária rápida (30 seg) e varredura completa do pipeline (~5 min)
- Pontua cada contato (comprador anterior, estágio do negócio, decaimento por recência)
- Gera um dashboard de prioridades: NEGÓCIOS QUENTES → COMPRADORES ANTERIORES → PARADOS → MORNOS → DORMENTES → FRIOS

---

## Framework de Oportunidades

Baseado na *infraestrutura simbólica* de Michel Alcoforado — relógios como sinais de acesso social, não apenas produtos.

### Tier 1 — Quente
1. **Fechamentos Adiados** — disse "depois", mencionou um mês/evento, depois silêncio
2. **Negociações Incompletas** — preço discutido, interesse confirmado, depois silêncio
3. **Preço Esfriou** — preço foi o último assunto antes do silêncio

### Tier 2 — Morno
4. **Rapport Pessoal** — eventos em família, viagens, conquistas profissionais
5. **Menções a Concorrentes** — "vi em outra loja", "fulano tem um igual"

### Tier 3 — Ângulos Alcoforado
6. **Momentos de Passaporte** — entrada em novos contextos sociais
7. **Transições de Status** — conquista profissional, promoção, liquidez, casamento
8. **Colecionadores** — já tem peças, pronto para a próxima
9. **Contexto de Presente** — aniversário, formatura, casamento, Dia dos Pais
10. **Insegurança do Novo Rico** — aprendendo os códigos → ângulo de consultor

---

## Templates de Dashboard

Ambos os dashboards usam dados fictícios de exemplo. Abra em qualquer navegador.

| Arquivo | Descrição |
|---------|-----------|
| `dashboards/ym-opportunities-template.html` | Varredura de oportunidades no estilo WhatsApp (5 seções recolhíveis, bolhas de chat, links WA) |
| `dashboards/watch-reengagement-template.html` | Rastreador de reengajamento do pipeline (6 níveis de urgência, pontuação, botões de cópia) |

---

## Requisitos

- Bridge [whatsapp-mcp](https://github.com/lharries/whatsapp-mcp) rodando localmente (`messages.db` com seu histórico do WhatsApp)
- [Claude Code](https://claude.ai/claude-code) com o WhatsApp MCP conectado
- Execute os workflows colando os arquivos `.md` como contexto ou salvando-os no seu projeto do Claude Code

---

## Como Usar

1. Inicie a bridge do WhatsApp MCP
2. Abra o Claude Code
3. Diga: `run the opportunity scan` ou `watch re-engagement full scan`
4. Dashboard salvo na sua pasta de saída

---

## Palavras-chave de Sinal (Português)

| Tipo | Palavras-chave |
|------|----------------|
| Comprador | fechado, pago, enviado, rastreio, PIX, entrega, comprei, recebi |
| Preço | quanto, preço, valor, R$, "k " (com espaços), mil, parcela |
| Interesse | tenho interesse, me manda foto, disponível, qual o valor, quero ver |
| Pessoal | viagem, férias, casamento, aniversário, presente, formatura |
| Relógios | Rolex, Omega, AP, Patek, JLC, Cartier, Tudor, Seiko, IWC, Hublot, Richard Mille |

---

## Observações

- **Contatos LID** (JIDs `@lid`) não têm link direto para o WhatsApp Web — marcados com badge cinza
- **Palavras-chave de mês** (jan–dez) geram falsos positivos de mensagens logísticas — filtre pelo contexto de intenção
- A varredura funciona melhor com `limit=50, include_context=true, context_before=3, context_after=3`
- Tipo de oportunidade mais recorrente: contatos que mencionaram um mês e nunca retornaram (fechamentos adiados)
