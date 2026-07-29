# Workflow de Desenvolvimento

> Este arquivo estende [common/git-workflow.md](./git-workflow.md) com o processo completo de desenvolvimento de features que acontece antes das operações git.

O Feature Implementation Workflow descreve o pipeline de desenvolvimento: pesquisa, planejamento, TDD, revisão de código e commit.

## Workflow de Implementação de Features

1. **Planeje Primeiro**
   - Use o agente **planner** para criar plano de implementação
   - Identifique dependências e riscos
   - Divida em fases
   - Consulte skills existentes em `skills/`

2. **Abordagem TDD**
   - Use o agente **tdd**
   - Escreva testes primeiro (RED)
   - Implemente para passar nos testes (GREEN)
   - Refatore (IMPROVE)
   - Verifique 80%+ de cobertura

3. **Revisão de Código**
   - Use o agente **code-reviewer** imediatamente após escrever código
   - Se Python: também use **python-reviewer**
   - Se TypeScript: também use **typescript-reviewer**
   - Resolva issues CRITICAL e HIGH

4. **Revisão de Segurança**
   - Use o agente **security-reviewer**
   - Verifique autenticação, autorização, SQLi, XSS
   - Nunca hardcode secrets

5. **Commit & Push**
   - Mensagens de commit detalhadas
   - Siga o formato conventional commits
   - Veja [git-workflow.md](./git-workflow.md) para formato

6. **Verificações Pré-Review**
   - Todos os checks automatizados (CI/CD) estão passando
   - Resolva conflitos de merge
   - Branch atualizada com a branch alvo