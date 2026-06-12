# Release Notes — Mapa do Projeto

Projeto: gerador/publicador de release notes da Naja.
Dois arquivos principais: `index.html` (frontend) e `Publicar Release Notes.json` (workflow n8n).

> **Regra de manutenção:** sempre que qualquer arquivo do projeto for alterado, atualizar este `CLAUDE.md` imediatamente — linhas, funções, nós ou seções afetadas. O mapa deve refletir o estado atual do código. Não espere ser solicitado.

---

## index.html (888 linhas)

### CSS (L9–L509)

| Seção | Linhas | O que contém |
|---|---|---|
| CSS Variables (`:root`) | L11–L38 | Todas as variáveis de cor, radius, shadow |
| Header | L51–L93 | `.header-text`, `.logo-wrap` |
| Card | L94–L130 | `.card`, `.card-title`, `.step-num` |
| Form | L131–L187 | `.field-group`, `.select-wrapper`, `.step-row` |
| Buttons | L188–L249 | `.btn`, `.btn-primary`, `.btn-success`, `.btn-secondary`, `.spinner` |
| Tabs | L250–L279 | `.tabs`, `.tab-btn`, `.tab-panel` |
| Editor | L280–L326 | `.editor-toolbar`, `.editor-label`, `.version-badge`, `.editor-meta` |
| Preview / Toggle | L327–L398 | `.preview-toolbar`, `.view-toggle`, `.toggle-btn`, `.html-preview-wrap`, `iframe` |
| Alerts | L399–L415 | `.alert`, `.alert-success`, `.alert-error`, `.alert-warning` |
| Status Grid | L416–L439 | `.status-grid`, `.status-item`, `.status-dot` (estados: ok/fail/loading) |
| Misc | L440–L466 | `.divider`, `.action-row`, `.char-count`, `.spinner-inline` |
| Encaminhamentos | L467–L509 | `.enc-empty`, `.enc-cliente`, `.enc-modulo`, `.enc-item` |

### HTML (L512–L639)

| Elemento | Linhas | IDs relevantes |
|---|---|---|
| Header | L515–L524 | — |
| **Card 1** — Seleção | L526–L558 | `card-select`, `app-select`, `version-select`, `btn-gerar`, `spinner-versoes`, `spinner-gerar`, `alert-select` |
| Opções de app (ClickUp lists) | L537–L539 | values: `901318416890` (Naja Core), `901318321791` (C#), `901318416922` (Web) |
| **Card 2** — Editor | L561–L618 | `card-editor`, `version-label`, `char-count`, `btn-publicar`, `spinner-publicar`, `alert-editor` |
| Abas do editor | L574–L577 | tab "Texto do release notes" / "Encaminhamentos para Clientes" |
| Preview iframe | L594–L597 | `html-preview`, `preview-placeholder`, `preview-frame` |
| Textarea (editor) | L600 | `editor` |
| Painel encaminhamentos | L612–L617 | `tab-enc`, `enc-content` |
| **Card 3** — Confirmação | L620–L638 | `card-confirm`, `alert-publicacao`, `alert-icon`, `alert-msg` |
| Status integrações | L630–L632 | `status-supabase`, `status-movidesk` (ClickUp), `status-gchat` |

### JavaScript (L640–L987)

| Função / Constante | Linhas | O que faz |
|---|---|---|
| `WEBHOOK_VERSOES` | L641 | URL do webhook para buscar versões por `list_id` |
| `WEBHOOK_GERAR` | L642 | URL do webhook para gerar o release notes |
| `WEBHOOK_PUBLICAR` | L643 | URL do webhook para publicar |
| `onAppChange()` | L647–L689 | Carrega versões no select ao trocar aplicação; chama `WEBHOOK_VERSOES` |
| `trocarAba(aba, event)` | L690–L696 | Alterna entre as abas do editor |
| `gerarReleaseNotes()` | L697–L736 | Chama `WEBHOOK_GERAR`, preenche editor e preview; chama `gerarEncaminhamentos()` |
| `gerarEncaminhamentos(tasks, version)` | L737–L782 | Monta painel de encaminhamentos por cliente/módulo |
| `publicar()` | L783–L842 | Chama `WEBHOOK_PUBLICAR`; envia `content` (HTML para Supabase) e `text` (plain text para ClickUp); atualiza status indicators |
| `setStatus(id, state, label)` | L842–L847 | Atualiza visual do status (ok/fail/loading) |
| `escapeHtml(str)` | L848–L854 | Escapa caracteres HTML especiais |
| `applyInlineFormat(str)` | L855–L861 | Formata negrito/itálico markdown inline |
| `textToHtml(text)` | L862–L909 | Converte texto plain (markdown-like) em HTML para o iframe |
| `syncPreview()` | L910–L933 | Renderiza conteúdo do editor no iframe |
| `setViewMode(mode)` | L934–L954 | Alterna entre visualização preview e código |
| `updateCount()` | L955–L959 | Atualiza contador de caracteres |
| `mostrarAlerta(id, tipo, msg)` | L960–L966 | Exibe alerta (success/warning/error) |
| `resetar()` | L967–L987 | Limpa tudo e volta ao estado inicial |

---

## Publicar Release Notes.json (workflow n8n, 555 linhas)

### Nós (nodes)

| Nó | Linhas | Tipo | O que faz |
|---|---|---|---|
| `Receber Publicação` | L5–L19 | Webhook POST | Recebe payload em `/publicar-release-notes` |
| `Verificar Release Existente` | L21–L50 | Supabase getAll | Busca em `release_notes` pelo campo `version` |
| `Release já existe?` | L52–L85 | IF | Verifica se `$json.id` existe (branch true = atualizar, false = criar) |
| `Atualizar Release` | L87–L126 | Supabase update | Atualiza `content` e `published_at` pelo `id` |
| `Criar Release` | L128–L161 | Supabase insert | Insere `version`, `title`, `content` |
| `Retornar Confirmação Atualizar` | L163–L191 | Respond to Webhook | Resposta com headers CORS (branch atualizar) |
| `Retornar Confirmação Criar` | L193–L221 | Respond to Webhook | Resposta com headers CORS (branch criar) |
| `Atualizar Descrição no ClickUp Atualizar` | L222–L266 | HTTP PUT | PUT na API ClickUp `/list/{task_id}` com campo `content` = `body.text` (plain text) |
| `Atualizar Descrição no ClickUp Criar` | L267–L311 | HTTP PUT | PUT na API ClickUp `/list/{task_id}` com campo `description` = `body.text` (plain text) |
| `Preparar Resposta Final Atualizar` | L312–L324 | Code | Retorna `{ supabase: true, clickup: true }` |
| `Preparar Resposta Final Criar` | L325–L337 | Code | Retorna `{ supabase: true, clickup: true }` |
| `Montar Mensagem Google Chat` | L338–L350 | Code | Monta texto com resumo por cliente/módulo a partir de `tasks` |
| `Notificar Google Chat Atualizar` | L351–L377 | HTTP POST | Envia mensagem ao GChat (branch atualizar); body usa `JSON.stringify(...)` para escapar `\n` e caracteres especiais |
| `Notificar Google Chat Criar` | L378–L404 | HTTP POST | Envia mensagem ao GChat (branch criar); body usa `JSON.stringify(...)` para escapar `\n` e caracteres especiais |

### Fluxo

```
Receber Publicação
  └─► Montar Mensagem Google Chat ─► Verificar Release Existente ─► Release já existe?
                                           ├─[true]─► Atualizar Release ─► ClickUp Atualizar ─► GChat Atualizar ─► Preparar Atualizar ─► Retornar Atualizar
                                           └─[false]─► Criar Release    ─► ClickUp Criar     ─► GChat Criar     ─► Preparar Criar     ─► Retornar Criar
```

> `Montar Mensagem Google Chat` roda antes de tudo — garante que os nós de GChat consigam referenciar `$('Montar Mensagem Google Chat').first()` sem erro de nó não conectado.

### Credenciais e URLs externas

| Item | Localização no JSON |
|---|---|
| Supabase credential ID | `"id": "l9PJBZIJPsr9u4U5"` (nos nós Supabase) |
| ClickUp credential ID | `"id": "QJt52hlsET8584yU"` (nos nós HTTP ClickUp) |
| Google Chat webhook URL | L354 e L381 (nós `Notificar Google Chat *`) |
| ClickUp API key (header Authorization) | L231 e L276 |

---

## Como fazer manutenções

Indique o que quer mudar e onde. Exemplos:

- "alterar cor de destaque" → CSS Variables, `index.html:L11`
- "adicionar nova aplicação no select" → `index.html:L537–L539`
- "mudar URL do webhook de publicar" → `index.html:L643`
- "ajustar mensagem do Google Chat" → `Publicar Release Notes.json:L338–L350` (nó `Montar Mensagem Google Chat`)
- "adicionar campo no Criar Release" → `Publicar Release Notes.json:L128–L161`
- "mudar comportamento do botão publicar" → `index.html:L783–L841` (função `publicar()`)
