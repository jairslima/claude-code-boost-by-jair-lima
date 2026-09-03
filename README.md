# Claude Code Boost — by Jair Lima

Uma [skill](https://docs.claude.com/en/docs/claude-code/skills) para o [Claude Code](https://claude.com/claude-code) que adiciona um comando `/boost`: um pipeline de raciocínio profundo multiagente para tarefas de engenharia realmente difíceis (race conditions, refatorações grandes, otimização de algoritmos, causa raiz em codebases grandes).

Inspirado no anúncio do [`/boost` do Google Antigravity](https://pasqualepillitteri.it/pt/news/13942/google-antigravity-boost-raciocinio-profundo) (31/08/2026), adaptado para as ferramentas nativas do Claude Code (subagentes via `Agent`, testes reais do projeto).

## Como funciona

Três camadas, igual ao original:

1. **Orchestrator** — decompõe o pedido em subtarefas verificáveis.
2. **Especialistas**
   - `DeepInvestigator`: agente(s) somente leitura, investigam causa raiz sem tocar em arquivos.
   - `DeepCoder`: agente(s) com permissão de escrita, implementam a solução em escopos separados.
3. **Validação** — roda os testes reais do projeto; se falhar, faz até 3 giros de correção realimentando o erro. Fecha com a "Regra do Diff": nunca reporta pronto sem ler e conseguir explicar o diff final inteiro.

Não é o modo padrão: só ativa quando você chama `/boost <tarefa>` explicitamente, porque custa mais tempo e tokens que uma resposta de turno único. A skill avisa quando a tarefa parece simples demais pra justificar o custo.

## Instalação

Copie a pasta `skills/boost` para dentro de `~/.claude/skills/`:

```bash
git clone https://github.com/jairslima/claude-code-boost-by-jair-lima.git
cp -r claude-code-boost-by-jair-lima/skills/boost ~/.claude/skills/boost
```

No Windows (PowerShell):

```powershell
git clone https://github.com/jairslima/claude-code-boost-by-jair-lima.git
Copy-Item -Recurse "claude-code-boost-by-jair-lima\skills\boost" "$env:USERPROFILE\.claude\skills\boost"
```

Reinicie o Claude Code (ou abra uma nova sessão) e o comando `/boost` fica disponível em qualquer projeto.

## Uso

```
/boost investigar por que a fila de processamento perde mensagens sob carga concorrente
```

## Licença

MIT — veja [LICENSE](LICENSE).
