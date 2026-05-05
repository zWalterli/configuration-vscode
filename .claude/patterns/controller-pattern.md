# Controller Pattern (ASP.NET Core API)

## Purpose

Handle HTTP requests and delegate processing to services.

## Rules

- Must be thin (no business logic)
- Must call Application Services
- Must return standardized responses
- Must validate input (ModelState or FluentValidation)
- Must use proper HTTP status codes

## Structure

- Inject services via constructor
- Use DTOs for input/output
- Handle success and error responses
- Use cancellation token when needed

## Example

```csharp
[ApiController]
[Route("api/v1/disciplinas")]
public class DisciplinaController : ControllerBase
{
    private readonly IDisciplinaService _service;

    public DisciplinaController(IDisciplinaService service)
    {
        _service = service;
    }

    [HttpGet]
    public async Task<IActionResult> ObterAtivos()
    {
        var result = await _service.ObterAtivosAsync();
        return Ok(result);
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> ObterPorId(Guid id)
    {
        var result = await _service.ObterPorIdAsync(id);

        if (result == null)
            return NotFound();

        return Ok(result);
    }

    [HttpPost]
    public async Task<IActionResult> Criar([FromBody] CriarDisciplinaDTO dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var result = await _service.CriarAsync(dto);

        if (result.IsFailed)
            return BadRequest(result.Errors);

        return CreatedAtAction(nameof(ObterPorId), new { id = dto.Nome }, null);
    }
}
```

Best Practices

- Use meaningful routes (RESTful)
- Use versioning (/v1/)
- Return correct status codes:
  200 OK
  201 Created
  400 BadRequest
  404 NotFound

Keep controller minimal

Anti-patterns
❌ Business logic inside controller
❌ Direct repository usage
❌ Returning Entity
❌ Try/catch desnecessário (usar middleware)
❌ Fat controllers
