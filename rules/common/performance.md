# Performance e Otimização

## Estratégia de Seleção de Modelo

O AI Pack usa DeepSeek V4 Flash como modelo padrão. Para tarefas que exigem raciocínio mais profundo, considere modelos mais potentes disponíveis na plataforma.

## Gerenciamento de Contexto

Evite os últimos 20% da janela de contexto para:
- Refatoração de larga escala
- Implementação de features em múltiplos arquivos
- Debug de interações complexas

Tarefas com menor sensibilidade a contexto:
- Edições em arquivo único
- Criação de utilitários independentes
- Atualizações de documentação
- Bug fixes simples

## Resolução de Erros de Build

Se o build falhar:
1. Use o agente **build-error-resolver**
2. Analise mensagens de erro
3. Corrija incrementalmente
4. Verifique após cada correção