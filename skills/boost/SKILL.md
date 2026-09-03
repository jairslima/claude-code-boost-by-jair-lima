---
name: boost
description: Ativa um pipeline de raciocínio profundo multiagente (Orchestrator → DeepCoder/DeepInvestigator → workers → validação) para tarefas de engenharia difíceis - race conditions, refatorações que tocam muitos arquivos, otimização de algoritmos, busca de causa raiz em codebase grande. Custa mais tempo/tokens que o modo normal - só usar quando o problema realmente exigir. Inspirado no /boost do Google Antigravity.
---

# /boost — raciocínio profundo sob demanda

Este comando NÃO é o padrão. Só ativa quando o usuário pede `/boost <tarefa>` explicitamente, porque consome bem mais tempo e tokens que uma resposta de turno único. Antes de começar, se a tarefa parecer trivial (bug óbvio, mudança de uma linha, pergunta simples), avise o usuário que `/boost` provavelmente não compensa aqui e pergunte se quer mesmo assim.

Casos em que vale a pena:
- Bugs de concorrência / race conditions
- Otimização algorítmica ou de estrutura de dados
- Refatoração que toca muitos arquivos ao mesmo tempo
- Investigação de causa raiz em codebase grande, onde o sintoma não é a causa

## Fluxo (três camadas)

### 1. Orchestrator (você mesmo, sem subagente)

Leia o pedido do usuário e o estado atual do workspace (arquivos relevantes, testes existentes, histórico git se houver). Quebre o problema em subtarefas discretas e verificáveis. Escreva essa decomposição como uma lista curta antes de despachar qualquer agente — isso é o plano de execução, não precisa de aprovação do usuário a menos que a tarefa seja arriscada (ver regras de ações reversíveis do sistema).

### 2. Especialistas (camada do meio)

Despache via `Agent` (tipo `general-purpose` ou `fork`, sem sobrescrever `model` — a escolha do modelo é sempre do usuário):

- **DeepInvestigator** — um ou mais agentes **somente leitura** (não editam arquivos) para investigar a causa raiz: rastrear o bug, mapear dependências, reproduzir a condição de corrida, explicar por que o sintoma aparece. Use `subagent_type: "Explore"` quando for busca/localização pura, ou `general-purpose` com instrução explícita de não editar nada quando precisar analisar lógica mais a fundo.
- **DeepCoder** — um ou mais agentes **com permissão de escrita** que implementam a solução para cada subtarefa da decomposição do Orchestrator, em escopos separados (arquivos/módulos diferentes) para não pisarem um no outro. Rode em paralelo quando os escopos não se sobrepõem; sequencialmente quando uma subtarefa depende do resultado da outra.

Cada agente despachado precisa de um prompt autocontido: o que investigar/implementar, o que está fora de escopo, e o que outro agente já está cobrindo (evita retrabalho e conflito de edição).

### 3. Validação e loop de correção (Orchestrator de novo)

Depois que os agentes voltarem:
1. Rode a suíte de testes real do projeto (ou o comando de build/lint equivalente) — nunca aceite "parece certo" sem rodar.
2. Se algo falhar, volte o diagnóstico de erro para uma nova rodada: despache outro DeepCoder (ou o mesmo, via SendMessage se ainda ativo) só para aquele ponto de falha, com o log do erro anexado ao prompt.
3. Repita no máximo 3 giros de correção. Se ainda falhar depois disso, pare e explique ao usuário o que foi tentado e onde travou — não insista indefinidamente.
4. Quando os testes passarem, faça você mesmo (Orchestrator) uma leitura final do diff completo antes de reportar como pronto.

## Regra do Diff (obrigatória no fechamento)

Nunca reporte a tarefa como concluída sem ter lido e conseguido explicar, linha por linha, o diff final combinado dos agentes. Se alguma mudança não fizer sentido pra você, não entregue — investigue ou remova antes de reportar.

## Relatório final ao usuário

Resuma em poucas frases: o que foi decomposto, quantos agentes rodaram e em que papel, o que os testes confirmaram, e qualquer ponto que ficou como dívida técnica ou merece revisão humana antes de dar por encerrado.
