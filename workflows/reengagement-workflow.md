# Workflow de Reengajamento de Relógios

## Gatilhos
- Diga: "update watch threads", "watch re-engagement", "scan WhatsApp"
- Automático: durante o Morning Kickstart (triagem rápida apenas)

## Pré-requisitos
- Bridge do WhatsApp MCP rodando (`~/tools/whatsapp-mcp/start-bridge.bat`)

## Triagem Rápida (Morning Kickstart, ~30 segundos)
Usada durante o Morning Kickstart — resumo leve, sem regenerar o dashboard.

1. `list_chats(limit=30, sort_by="last_active")` → filtrar DMs
2. `get_last_interaction(jid)` para os 10 mais recentes
3. Relatório: "X contatos aguardando resposta, Y novas mensagens desde ontem"
4. Sem varredura profunda, sem atualização de dashboard

## Pipeline de Varredura Completa (~5 min)

### Fase 1 — INVENTÁRIO (1 chamada MCP)
```
list_chats(limit=250, sort_by="last_active") → filtrar apenas DMs (excluir grupos)
```
Saída: lista de JIDs com timestamps da última atividade

### Fase 2 — TRIAGEM (~N chamadas MCP, em lote)
```
get_last_interaction(jid) for each DM
```
Critérios de filtro:
- Ativo desde junho de 2025 (descartar mais antigos)
- Identificar ~50-80 contatos para varredura profunda
- Classificação rápida: "estão esperando" (mensagens não lidas deles) vs "devo responder" (minha última mensagem, sem resposta)

### Fase 3 — VARREDURA PROFUNDA (buscas principais, subagentes fazem o parse)
```
list_messages(chat_jid=X, after="2025-06-01", limit=50) per contact
```
- Salvar lotes de mensagens em arquivos temporários: `~/nexus/data/temp-watch-batch-N.json`
- Lançar ~5 subagentes em paralelo para processar os lotes
- Cada subagente extrai:
  - **Sinais**: palavras-chave de negócio, preços, interesse, eventos pessoais
  - **Ganchos pessoais**: viagens, eventos, preferências, histórias
  - **Marcas de relógio**: modelos e marcas mencionados
  - **Indicadores de negócio**: preço discutido, pagamento mencionado, rastreio de entrega
  - **Resumo**: resumo do contato em 2-3 linhas

### Fase 4 — CLASSIFICAÇÃO E PONTUAÇÃO (1 subagente)
**Fórmula de pontuação:**
- Base: comprador anterior +40, negócio ativo +50, demonstrou interesse +25, tem ganchos pessoais +15
- Decaimento: -1 por dia em silêncio (limite de -40)
- Bônus: está esperando resposta +20, evento pessoal próximo +20, múltiplas interações +10

**Categorias (ordenadas por prioridade):**

| # | Categoria | Cor | Critério |
|---|-----------|-----|----------|
| 1 | HOT DEALS | Vermelho (#e74c3c) | Negociação de preço ativa nos últimos 14 dias |
| 2 | PAST BUYERS | Roxo (#9b59b6) | Palavras-chave de negócio nas mensagens (fechado, pago, rastreio, PIX) |
| 3 | STALLED | Amarelo (#f39c12) | Teve discussão de preço mas ficou em silêncio há mais de 14 dias |
| 4 | WARM PROSPECTS | Azul (#3498db) | Interesse genuíno (pediu fotos, relógios específicos) mas nunca concluiu negócio |
| 5 | DORMANT | Teal (#1abc9c) | Boa relação, várias mensagens, em silêncio há mais de 30 dias |
| 6 | COLD LEADS | Cinza (#7f8c8d) | Consulta única, engajamento mínimo, em silêncio há mais de 60 dias |

**O subagente também gera:**
- Abertura de reengajamento em português para cada contato
- Referencia o contexto real da conversa (ganchos pessoais, interesses em relógios)

### Fase 5 — RENDERIZAÇÃO (agente principal)
- Gerar dashboard HTML autossuficiente
- Saída: `~/nexus/dashboards/watch-reengagement.html`
- Limpar arquivos temporários: `rm ~/nexus/data/temp-watch-*`

## Detecção de Sinais (Palavras-chave em Português)

| Tipo | Palavras-chave |
|------|----------------|
| Comprador | fechado, pago, enviado, rastreio, transferência, PIX, entrega, comprei, recebi |
| Preço | quanto, preço, valor, R$, numbers with "k", mil, parcela |
| Interesse | tenho interesse, me manda foto, disponível, qual o valor, tem esse, quero ver |
| Pessoal | viagem, férias, casamento, aniversário, presente, formatura, nascimento |
| Relógios | Rolex, Omega, AP, Audemars Piguet, Patek, JLC, Jaeger, Cartier, Tudor, Seiko, IWC, Hublot, Richard Mille, Submariner, Daytona, Speedmaster, Royal Oak, Nautilus, Santos, Reverso |

## Tratamento de Erros
- Bridge do WhatsApp não está rodando → alertar o usuário, indicar o start-bridge.bat
- Contato não tem mensagens desde junho de 2025 → ignorar (inativo demais)
- Timeout de subagente → tentar novamente uma vez, depois ignorar o lote com aviso

## Recursos do Dashboard de Saída
- Tema escuro, HTML autossuficiente (sem dependências externas)
- 6 seções com categorias codificadas por cor
- Cada cartão de contato mostra: nome, última atividade, pontuação, categoria, ganchos pessoais, abertura sugerida
- Botão de copiar em cada abertura
- Cabeçalho com estatísticas: total de contatos, contagens por categoria, timestamp da última varredura
- Abre no navegador Edge

## Localização dos Arquivos
| Arquivo | Finalidade |
|---------|------------|
| `nexus/dashboards/watch-reengagement.html` | Dashboard de saída (regenerado a cada execução) |
| `nexus/workflows/watch-reengagement-workflow.md` | Este arquivo |
