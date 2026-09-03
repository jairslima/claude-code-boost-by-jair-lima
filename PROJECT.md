# Claude Code Boost — by Jair Lima

## Descrição

Skill para o Claude Code que adiciona um comando `/boost`: pipeline de raciocínio profundo multiagente (Orchestrator → DeepInvestigator/DeepCoder → validação com loop de correção) para tarefas de engenharia difíceis. Inspirado no `/boost` do Google Antigravity (anunciado 31/08/2026).

## Stack e dependências

- Nenhuma dependência de runtime: é um único arquivo Markdown (`skills/boost/SKILL.md`) com frontmatter YAML lido pelo próprio Claude Code.
- Requer Claude Code com suporte a Skills e ao `Agent` tool (subagentes).

## Estrutura de pastas

```
claude-code-boost-by-jair-lima/
├── skills/
│   └── boost/
│       └── SKILL.md   ← arquivo principal, único conteúdo funcional
├── README.md
├── LICENSE             (MIT)
└── PROJECT.md          (este arquivo)
```

## Comandos essenciais

Não há build/test automatizado — é um arquivo de instruções em linguagem natural. Validação é manual: instalar em `~/.claude/skills/boost/`, chamar `/boost <tarefa>` numa sessão real do Claude Code e conferir se o comportamento segue o fluxo descrito (decomposição → subagentes → validação → relatório).

Instalação local para teste:

```bash
cp -r skills/boost ~/.claude/skills/boost
```

## Decisões arquiteturais

- **Sem override de `model` nas chamadas de Agent**: instrução explícita do usuário (Jair) — a escolha de modelo é sempre dele, nunca da skill.
- **Máximo 3 giros de correção** no loop de validação, para não entrar em loop infinito gastando tokens sem convergir.
- **"Regra do Diff"**: a skill proíbe reportar a tarefa como concluída sem o Orchestrator ter lido e conseguido explicar o diff final inteiro — mitiga o risco (citado no artigo que inspirou a skill) de aceitar patches de multiagentes sem entender.
- Repositório público, sem segredos/API keys — apenas instrução textual.

## Estado atual

Publicado em 2026-09-03. Skill funcional e instalada localmente na máquina do autor (`C:\Users\jairs\.claude\skills\boost\SKILL.md`). Ainda sem uso real registrado em tarefa de produção.

## Próximos passos

- Coletar feedback do primeiro uso real de `/boost` e ajustar o fluxo se o comportamento observado divergir do esperado.
- Considerar adicionar exemplos de prompt de despacho para DeepInvestigator/DeepCoder no próprio SKILL.md se a prática mostrar que agentes despachados sem exemplo saem genéricos demais.

## Problemas conhecidos / bugs abertos

Nenhum registrado até o momento (skill recém-criada, sem uso real ainda).
