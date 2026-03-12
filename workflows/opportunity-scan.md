# YM Watches — Workflow de Varredura de Oportunidades no WhatsApp

**Criado em:** 2026-03-11
**Propósito:** Varredura completa de todas as conversas de DM do YM Watches para identificar oportunidades de venda dormentes, usando o framework de infraestrutura simbólica de Michel Alcoforado.

---

## Quando Executar

- Início de um novo trimestre de vendas
- Após um intervalo de 2+ semanas sem revisão ativa do pipeline
- Após eventos importantes (Shoptalk, eTail, NRF, Carnaval, Natal, etc.)
- Sempre que Bruno quiser um refresh completo do pipeline do YM

---

## Teoria do Trabalho

Framework de Michel Alcoforado aplicado à venda de relógios de luxo para a elite brasileira:
> Relógios não são bens de luxo — são **infraestrutura simbólica**. Eles sinalizam acesso social, pertencimento a grupos e transição de status.

Isso significa que a varredura deve identificar não apenas negociações de preço explícitas, mas também:
- **Momentos de passaporte** — entrada em novos contextos sociais (clube, viagem, evento, novo endereço)
- **Transições de status** — vitória nos negócios, promoção, cargo de liderança, evento de liquidez, casamento
- **Colecionadores** — já tem uma peça, pronto para a próxima
- **Contextos de presente** — aniversário, formatura, casamento, Dia dos Pais
- **Insegurança do novo rico** — aprendendo os códigos, fazendo muitas perguntas → ângulo de consultor

---

## Fonte de Dados

- **DB:** `messages.db` — 65k+ mensagens, 1.146 chats
- **Apenas DMs:** Filtrar JIDs com `@s.whatsapp.net` (excluir grupos `@g.us`)
- **Contatos LID:** JIDs com `@lid` = sem link direto pelo WhatsApp
- **Ferramentas MCP:** `mcp__whatsapp__list_messages`, `mcp__whatsapp__list_chats`, `mcp__whatsapp__get_message_context`

---

## Categorias de Oportunidade

### Nível 1 — Quente (acionável agora)
1. **Fechamentos com atraso** — disse "depois", citou um mês/evento, disse que voltaria
2. **Negociações incompletas** — preço discutido, interesse confirmado, depois silêncio
3. **Conversas travadas no preço** — preço foi o último assunto antes do silêncio

### Nível 2 — Morno (capital de relacionamento)
4. **Rapport pessoal** — eventos de família, viagens, conquistas nos negócios, detalhes do estilo de vida
5. **Menções a concorrentes ou comparações** — "vi em outra loja", "fulano tem um igual"

### Nível 3 — Ângulos Alcoforado (reengajamento estratégico)
6. **Momentos de passaporte** — entrada em novos contextos sociais
7. **Transições de status** — vitória nos negócios, promoção, evento de liquidez, casamento
8. **Colecionadores** — já tem uma peça, pronto para a próxima
9. **Contexto de presente** — aniversário, formatura, casamento, Dia dos Pais
10. **Insegurança do novo rico** — aprendendo os códigos → ângulo de consultor

---

## Plano de Execução

### Fase 1 — Varredura por Palavras-chave (3 subagentes em paralelo)

Cada subagente usa: `limit=50, include_context=true, context_before=3, context_after=3`
Filtro: apenas chats de DM (`@s.whatsapp.net` ou `@lid`, NÃO `@g.us`)

**Subagente A — Preço e Negociação:**
```
preço, valor, quanto, " k " (spaces), interesse, proposta, negociação, desconto, oferta, aceito
```

**Subagente B — Adiamentos e Follow-up:**
```
mês que vem, semana que vem, carnaval, férias, viagem, volto, depois, espera, aguarda,
ano que vem, final do ano, janeiro, fevereiro, março, abril, maio, junho, julho,
agosto, setembro, outubro, novembro, dezembro
```

**Subagente C — Rapport e Ângulos Alcoforado:**
```
aniversário, casamento, formatura, presente, filho, negócio fechado, comprei, mudei,
vi em outra, outra loja, rolex, patek, ap royal, coleção, clube
```

Coletar todos os valores únicos de `chat_jid` encontrados em todas as buscas.

### Fase 2 — Leitura Profunda por Chat (lotes em paralelo)

Mesclar todos os chat_jids únicos da Fase 1. Priorizar por:
1. Mensagem mais recente (threads mais ativas primeiro)
2. Palavras-chave de alto valor (menções de preço, nomes de modelos)
3. Compromissos de tempo explícitos

Para cada chat de DM: `list_messages(chat_jid=X, limit=100)`

Classificar conforme as 10 categorias de oportunidade. Extrair:
- Nome do contato, data da última mensagem
- Relógio discutido, preço se mencionado
- Detalhes pessoais, sinais de status
- Categoria principal, ângulo de reengajamento
- Linha de abertura sugerida em português

Executar em lotes de 5 chats por subagente, máximo de 10 subagentes em paralelo.

### Fase 3 — Síntese

Um subagente de síntese lê todas as saídas dos lotes e produz o relatório final.

---

## Formato de Saída

**Arquivo:** `~/nexus/dashboards/ym-opportunities-YYYY-MM-DD.html`

**Estilo:** Estética do WhatsApp Web
- Fundo: `#ECE5DD`
- Cabeçalho: `#075E54`
- Balões enviados: `#DCF8C6`
- Balões recebidos: `#FFFFFF`

**Cada card contém:**
- Nome do contato + data da última atividade + badge de categoria
- 3–5 balões de conversa mostrando a troca principal
- Rótulo de oportunidade com explicação
- Pills de relógio + preço + metadados de tempo
- Ângulo de reengajamento (1 frase)
- **Linha de abertura em português pronta para envio** em balão verde de envio
- 💬 Link para o WhatsApp Web (`https://web.whatsapp.com/send?phone=NUMBER`)

**Seções (recolhíveis):**
- 🔴 Fechamentos com Atraso
- 🟡 Negociações Incompletas
- 💰 Preço Esfriou
- 💬 Capital de Rapport
- 🎯 Ângulos Alcoforado

---

## Referência de CSS

CSS base de `~/nexus/dashboards/watch-reengagement.html`.
Links do WhatsApp: `https://web.whatsapp.com/send?phone=PHONENUMBER` (sem `+`, sem espaços).
Contatos LID não têm link direto — marcar com badge cinza "LID — sem link direto".

---

## Lições Aprendidas (execução de 2026-03-11)

- **JIDs LID** (`@lid`) não têm número de telefone — não é possível gerar links do WhatsApp Web
- **Palavras-chave de mês** (janeiro, fevereiro, etc.) geram alto volume, mas com muitos falsos positivos vindos de mensagens de logística — filtrar pelo contexto de intenção
- **Tamanho do prompt do Subagente C** pode exceder os limites se houver muitas palavras-chave num só prompt — dividir em dois se necessário
- **"k " com espaços** captura menções de preço como "50k", "100k" em texto informal
- **Sinais Alcoforado muitas vezes não são sobre o relógio** — são sobre o contexto da pessoa. Não ignorar mensagens sobre eventos sociais ou de vida
- **Tipo de oportunidade mais recorrente:** contatos que nomearam explicitamente um mês/evento e nunca voltaram → fechamentos com atraso são a categoria de maior retorno

---

## Checklist de Verificação

- [ ] Arquivo de saída existe com todas as 5 seções preenchidas
- [ ] Pelo menos uma entrada por categoria (se existirem conversas)
- [ ] Toda entrada tem uma linha de abertura em português
- [ ] Toda entrada com JID de telefone tem um link para o WhatsApp Web
- [ ] Nenhum conteúdo do InsiderOne/SDR misturado (REGRA 7)
- [ ] Dashboard commitado em `~/nexus/` e enviado (push)
