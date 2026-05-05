# Service Pattern (.NET - Application Layer)

## Purpose

Encapsulate business logic and orchestrate domain operations.

## Rules

- Must NOT access DbContext directly
- Must use repositories (interfaces)
- Must NOT contain infrastructure concerns
- Must handle business rules and validations
- Must return DTOs or Result pattern (e.g., FluentResults)
- Must be async

## Structure

- Interface: Application layer
- Implementation: Application layer
- Dependencies:
  - Repositories
  - Domain entities
  - UnitOfWork (if needed)

## Example

```csharp
public interface IDisciplinaService
{
    Task<List<DisciplinaDTO>> ObterAtivosAsync();
    Task<DisciplinaDTO?> ObterPorIdAsync(Guid id);
    Task<Result> CriarAsync(CriarDisciplinaDTO dto);
}

public class DisciplinaService : IDisciplinaService
{
    private readonly IDisciplinaRepository _repository;
    private readonly IApplicationDbContext _context;

    public DisciplinaService(
        IDisciplinaRepository repository,
        IApplicationDbContext context)
    {
        _repository = repository;
        _context = context;
    }

    public async Task<List<DisciplinaDTO>> ObterAtivosAsync()
    {
        var disciplinas = await _repository.ListarAtivasAsync();

        return disciplinas.Select(d => new DisciplinaDTO
        {
            Id = d.IdExterno,
            Nome = d.Nome,
            Descricao = d.Descricao,
            Ativo = d.Ativo
        }).ToList();
    }

    public async Task<DisciplinaDTO?> ObterPorIdAsync(Guid id)
    {
        var disciplina = await _repository.ObterPorIdAsync(id);
        if (disciplina == null) return null;

        return new DisciplinaDTO
        {
            Id = disciplina.IdExterno,
            Nome = disciplina.Nome,
            Descricao = disciplina.Descricao,
            Ativo = disciplina.Ativo
        };
    }

    public async Task<Result> CriarAsync(CriarDisciplinaDTO dto)
    {
        var existe = await _repository.ExistePorNomeAsync(dto.Nome);
        if (existe)
            return Result.Fail("Disciplina já existe.");

        var disciplina = Disciplina.Criar(dto.Nome, dto.Descricao);

        await _repository.AdicionarAsync(disciplina);
        await _context.SaveChangesAsync();

        return Result.Ok();
    }
}
```

Best Practices
Keep methods small and focused
Validate inputs early
Use domain methods for business logic
Do not expose entities directly
Use mapping (manual or AutoMapper)
Anti-patterns
❌ Business logic in controller
❌ Direct DbContext usage
❌ Returning Entity instead of DTO
❌ Sync methods