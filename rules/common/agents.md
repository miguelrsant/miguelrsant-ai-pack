# Orquestração de Agentes

## Agentes Disponíveis

Os agentes estão definidos em `opencode.json` e seus prompts em `agents/`:

| Agente | Propósito | Quando Usar |
|--------|-----------|-------------|
| planner | Planejamento de implementação | Features complexas, refatoração |
| architect | Design de sistemas | Decisões arquiteturais |
| tdd | Desenvolvimento orientado a testes | Novas features, bug fixes |
| code-reviewer | Revisão de código | Após escrever código |
| security-reviewer | Análise de segurança | Antes de commits |
| build-error-resolver | Corrigir erros de build | Quando o build falha |
| e2e | Testes E2E | Fluxos críticos de usuário |
| documentation | Documentação | Atualizando docs |
| python-reviewer | Revisão Python | Projetos Python |
| typescript-reviewer | Revisão TypeScript | Projetos TS/JS |
| database-reviewer | Revisão de banco de dados | Migrations, queries |
| api-reviewer | Revisão de contratos API | OpenAPI, REST |
| production-reviewer | Prontidão para produção | Antes de deploy |

## Uso Imediato de Agentes

Sem necessidade de prompt do usuário:
1. Features complexas → Use **planner**
2. Código recém-escrito → Use **code-reviewer**
3. Bug fix ou nova feature → Use **tdd**
4. Decisão arquitetural → Use **architect**

## Execução Paralela de Tarefas

Sempre use execução paralela para operações independentes:

```markdown
# BOM: Execução paralela
Disparar 3 agentes em paralelo:
1. Agente 1: Análise de segurança do módulo auth
2. Agente 2: Revisão de performance do cache
3. Agente 3: Type checking de utilitários

# RUIM: Sequencial quando desnecessário
Primeiro agente 1, depois agente 2, depois agente 3
```

## Contrato de Delegação

Aplica-se a todo agente em toda profundidade:

1. **Sua mensagem final É a entrega.** Nunca termine com "aguardando agentes de fundo" — uma task disparada não é uma tarefa concluída.
2. **Se você delegar, você é dono da coleta.** Espere resultados, integre-os, então retorne. Delegação fire-and-forget é proibida.
3. **Decomponha apenas quando o trabalho não couber em um contexto.** Não re-delegue uma tarefa já dimensionada para um único agente.

## Análise Multi-Perspectiva

Para problemas complexos, use sub-agentes com papéis divididos:
- Revisor factual
- Engenheiro sênior
- Especialista em segurança
- Revisor de consistência
- Verificador de redundância