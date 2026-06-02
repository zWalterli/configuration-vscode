# Claude Code Global Guidelines

## Token Optimization

@RTK.md

## Agents

Agents are auto-routed by Claude Code based on task description. See `AGENTS.md` for reference.

**Available agents:**
- `dotnet-agent` — .NET APIs, Clean Architecture, DDD, EF Core
- `database-agent` — Data modeling, query optimization, schema design
- `sqlserver-agent` — T-SQL, indexes, execution plans
- `react-agent` — React components, hooks, state management
- `angular-agent` — Angular applications, RxJS, Signals
- `devops-agent` — CI/CD, Docker, Kubernetes, infrastructure
- `aws-agent` — AWS infrastructure, Lambda, API Gateway
- `azure-agent` — Azure infrastructure, APIM, Entra ID
- `javascript-agent` — Vanilla JS, browser APIs, Node.js
- `typescript-agent` — Strict typing, generics, advanced patterns

## Global Patterns

### Code Quality
- Always prefer simplicity over complexity
- No premature abstractions
- Delete unused code completely (no stubs or `// removed` comments)
- Production-ready by default
- Follow SOLID principles

### Error Handling
- Validate only at system boundaries (user input, external APIs)
- Trust internal code and framework guarantees
- Use global exception middleware when applicable
- Avoid unnecessary try/catch

### Testing
- Test behavior, not implementation
- Cover success AND failure cases
- Use framework-native tools (xUnit, pytest, Jest, etc)
- Integration tests hit real databases/services when possible

### Performance
- Use async/await everywhere
- Profile before optimizing
- Pagination for large result sets
- Caching only when necessary

### Logging & Observability
- Log important actions and failures
- Never log sensitive data (passwords, tokens, PII)
- Use structured logging
- Context matters more than verbosity

## Global Tools & Systems

### Git
- **Repository**: GitHub
- **Branch prefix**: `feature/`, `fix/`, `chore/`
- **Default branch**: `main`
- **Commits**: Português (PT-BR), imperative mood, concise (1-2 linhas max)
- **Format**: `<type>: <description>` (fix, feat, refactor, docs, chore, test)

### CLI
- **Token optimization**: RTK (all commands automatically wrapped)
- **Shell**: PowerShell (Windows), Bash available for POSIX scripts
- **Git user**: Walterli Valadares Junior

### Communication
- **Responses**: Short, concise, no unnecessary summaries
- **Tone**: Direct, professional
- **Updates**: Only at key moments (found something, changing direction, blocker)
- **Code references**: `file_path:line_number` format

## Commit Examples

```
feat: Adicionar suporte a JWT no middleware de autenticação
fix: Corrigir query N+1 em carregamento de usuários
refactor: Simplificar lógica de validação de e-mail
docs: Atualizar guia de configuração do Docker
chore: Atualizar dependências do projeto
test: Adicionar testes para cálculo de juros
```

## Workflow

1. **Planning**: Use EnterPlanMode for non-trivial tasks
2. **Execution**: Edit existing files > create new ones
3. **Testing**: Verify changes in running app before declaring done
4. **Git**: Create new commits (don't amend) unless explicitly asked
5. **PRs**: Create with clear summary, test plan, and context
