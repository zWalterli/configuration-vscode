Standard repository pattern:

- Interface in Domain
- Implementation in Infrastructure
- Use EF Core DbContext
- No business logic

Example:

```csharp
using System.Linq.Expressions;
using Microsoft.EntityFrameworkCore;
using SabespService.Domain.BancoPreco.Entities;
using SabespService.Domain.BancoPreco.Repositories;
using SabespService.Domain.BancoPreco.Repositories.Query;
using SabespService.Domain.SharedKernel;
using SabespService.Domain.SharedKernel.Enums;

namespace SabespService.Infrastructure.Persistence.Repositories;

public class AplicacaoRepository(ApplicationDbContext _context) : IAplicacaoRepository
{
    public async Task<List<Aplicacao>> ObterPorIdExternoAsync(List<Guid> idsExternos, CancellationToken ct = default)
    {
        return await _context.Aplicacoes
            .Where(a => idsExternos.Contains(a.IdExterno))
            .ToListAsync(ct);
    }

    public async Task<Aplicacao?> ObterAplicacaoPorTituloAsync(string titulo, CancellationToken ct = default)
    {
        return await _context.Aplicacoes
            .FirstOrDefaultAsync(a => EF.Functions.Like(a.Titulo, $"%{titulo}%"), ct);
    }

    public async Task AtualizarAsync(Aplicacao aplicacao, CancellationToken cancellationToken = default)
    {
        _context.Aplicacoes.Update(aplicacao);
        await Task.CompletedTask;
    }

    public async Task<Aplicacao?> GetByIdAsync(Guid idExterno, CancellationToken cancellationToken = default)
    {
        return await _context.Aplicacoes.FirstOrDefaultAsync(a => a.IdExterno == idExterno, cancellationToken);
    }

    public async Task<Aplicacao?> GetByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        return await _context.Aplicacoes.FirstOrDefaultAsync(a => a.Id == id, cancellationToken);
    }

    public async Task InserirAsync(Aplicacao aplicacao, CancellationToken cancellationToken = default)
    {
        await _context.Aplicacoes.AddAsync(aplicacao, cancellationToken);
    }

    public async Task<(List<(Aplicacao aplicacao, bool isFavorite)> Items, int TotalCount)> ListarAsync(
        FiltroAplicacao filtros,
        int usuarioId,
        bool? isFavorite,
        List<SortExpression> sorts,
        CancellationToken ct)
    {
        var query = _context.Aplicacoes.AsNoTracking();

        var favoritosQuery = _context.Favoritos
            .Where(f => f.IdUsuario == usuarioId && f.TipoEntidade == TipoFavoritoEnum.APLICACAO);

        if (!string.IsNullOrEmpty(filtros.Titulo))
            query = query.Where(a =>
                EF.Functions.Like(a.Titulo, $"%{filtros.Titulo}%"));

        if (filtros.Ativo.HasValue)
            query = query.Where(a => a.Ativo == filtros.Ativo.Value);

        if (isFavorite.HasValue)
        {
            query = isFavorite.Value
                ? query.Where(a => favoritosQuery.Any(f => f.IdEntidade == a.Id))
                : query.Where(a => !favoritosQuery.Any(f => f.IdEntidade == a.Id));
        }

        var totalCount = await query.CountAsync(ct);

        var queryComFavorito = query
            .Select(a => new AplicacaoComFavorito
            {
                Aplicacao = a,
                IsFavorite = favoritosQuery.Any(f => f.IdEntidade == a.Id)
            });

        var allowedSorts = new Dictionary<string, Expression<Func<AplicacaoComFavorito, object>>>
        {
            { "titulo", x => x.Aplicacao.Titulo },
            { "ativo", x => x.Aplicacao.Ativo },
            { "id", x => x.Aplicacao.Id },
            { "isfavorite", x => x.IsFavorite }
        };

        sorts ??= new List<SortExpression>();

        queryComFavorito = queryComFavorito.ApplySorting(
            sorts,
            allowedSorts,
            x => x.Aplicacao.Id
        );

        var items = await queryComFavorito
            .Skip((filtros.Pagina - 1) * filtros.TamanhoPagina)
            .Take(filtros.TamanhoPagina)
            .ToListAsync(ct);

        var result = items
            .Select(x => (x.Aplicacao, x.IsFavorite))
            .ToList();

        return (result, totalCount);
    }
}
```
