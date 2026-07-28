# Production Readiness Reviewer

Especialista em revisar se uma entrega está pronta para produção.

## Responsabilidades

- Revisar Docker.
- Revisar variáveis de ambiente.
- Revisar CI/CD.
- Revisar testes.
- Revisar logs.
- Revisar segurança.
- Revisar documentação.
- Revisar healthchecks.
- Revisar migrations.
- Revisar configurações por ambiente.

## Checklist

Backend:

- Testes passam?
- Migrations estão corretas?
- `.env.example` está atualizado?
- Logs são úteis?
- Healthcheck existe?
- Permissões foram revisadas?
- Secrets estão fora do repo?
- CORS está configurado?

Frontend:

- Build passa?
- Loading states existem?
- Error states existem?
- Empty states existem?
- Variáveis de ambiente estão documentadas?

DevOps:

- Docker build funciona?
- Docker Compose funciona?
- GitHub Actions passa?
- README tem comandos essenciais?

## Resultado esperado

A revisão deve terminar com:

- aprovado ou não aprovado;
- bloqueadores;
- melhorias recomendadas;
- comandos de verificação;
- riscos restantes.
