# API_REST_Fundamentals
### Notes from Building an API with ASP.NET Core by Shawn Wildermuth Pluralsight course.

How does HTTP Work?

Client send a Request has **_verb_** (which is you want to do), **_headers_** (which include additional information) and **_content._**
Then the server send a Response, There's a **_status code_** for whether it succedded or not, **_headers_** that might include information and it may also include **_content._**

More importants verbs
- GET: Retrieve a resource (READ)
- POST: Add a new resource (CREATE)
- PUT: Update an existing resource (UPDATE)
- PATCH: Update an existing resource with set of changes
- DELETE: Remove the existing resource (DELETE)

REST Concept (REpresentation State Transfer):
- Separation of Client and Server
- Server Requests are Stateless
- Cacheable Requests
- Uniform Interface

What are Resources? People, invoices, payments, products, etc. Represent the objects in our system. (Domain models or Entities).

What are URIs? (Uniform Resources Identifier) Are just paths to Resources like api.yourserver.com/invoice
Query Strings can be used for non-data elements (they're not part ot the URI itself, are optional arguments)

Our controller, When create the controller we create a class that inherits from ControllerBase

```C#
public class ClientController : ControllerBase
```

 - The attributes
 We need to add the Route attribute
 [Route("api/[controller]")] --> That means the route will be api/client
 
 - The method, now we can create a get method
 
``` C#
[HttpGet]
public object GetClients(){
    return new { Name = "Juan Carlos", Surname = "Fuentes" };
}
```
 
We can return a status code for indicated if the respond will be ok or wrong returning an ActionResult
 
```C#
[HttpGet]
public async Task<ActionResult> Get()
{
    try
    {
        var results = await _repository.GetAllClientsAsync();

        return Ok(results);
    }
    catch (Exception)
    {
        return this.StatusCode(StatusCodes.Status500InternalServerError, "Database Failed")
    }

}
```
 
Is better return a Model instead of Entity, why?

  - Payload is a contract with your users
  - Likely want to filter data for security too
  - Surrogate Keys are useful too
  
 We can use AutoMapper.Extensions.Microsoft.DependencyInjection from NuGet Packages for map the entity (Client.cs) with the model (ClietnModel.cs)
 
 In ConfigureServices method from Startup class we have to add below the depency injection
 
 ```C#
 services.AddScoped<IClientRepository, ClientRepository>();
 
 services.AddAutoMapper(Assembly.GetExecutingAssembly());
 ```
 
With Automapper we can map complex object, if we have a Location class that has multiple properties inside our Entity Client for expample

```C#
this.CreateMap<Client, ClientModel>()
  .ForMember(c => c.Venue, o => o.MapFrom( m => m.Location.VenueNmae));
```

And create a new class ClientProfile.cs than inherit from Profile
 
 ```C#
 public class ClientProfile : Profile
 {
    public ClientProfile()
      {
        this.CreateMap<Client, ClientModel>();
      }
  }
```

Then we return to our controller and inject the mapper in the constructor

```C#

private readonly IMapper _mapper;

public ClientController(IClientRepository repository, IMapper mapper)
{
    _repository = repository;
    _mapper = mapper;
 }
 ```
 
 And in the try of our Get method
 
 ```C#
 var results = await _repository.GetAllClientsAsync();

ClientMode[] models = _mapper.Map<ClientModel[]>(results);

return Ok(models);
```
      
And now returns only just we need in the ClientModel.  Is better if we returns the model

```C#
return _mapper.Map<ClientModel[]>(results);
```

and in the method header specify the type to return

```C#
public async Task<ActionResult<ClientModel[]>> Get()
```

In the Get attribute we can specify a name, because we can't have two get methods
For the second get method we can add this

```C#
[Route("{getclient: int}")] // This will be like api/client/getclient/1
public ActionResult<Client> Get(int id)
{
    try
    {
        var result = await _repository.GetClientAsync(id);
        if (result == null) return NotFound();
        return _mapper<ClientModel>(result);
    }
```

POST Verb
For receive data in the method we need to do one of this two thing:
 - Add [FromBody] in the parameter's merhod
 
 ```C#
 public aync Task<ActionResult<Client>> Post([FromBody] ClientModel model)
 ```
 
 - Add [ApiController] attribute before the class declaration, This attribute tells the system a lot about waht we want as far as expectations.
   - The first is it's going to attempt to do body binding because it knows it's an API.
   
```C#
[Route("api/[controller]"])
[ApiController]
public class ClientController : ControllerBase
```

In the Post method we have to do the reverse with the modeling code

```C#
var client = _mapper.Map<Client>(model);
```

In the Post method is better return a Created($"/api/clients/{client.Code}", _mapper.Map<ClientModel>(client)); this indicate the 201 status code.
But we don't want to hardCode, we can use LinkGenerator, this is in the Microsoft.AspNetCore routing namespace for versions after 2.2.
We have to add in the constructor
 
```C#
public ClientsController(IClientRepository repository, IMapper mapper, LinkGenerator linkGenerator)
```

And then in the post method

```C#
var location = _linGenerator.GetPathByAction("Get", "Clients", new { code = model.Code });
```

Adding Model Validation
 - In the model we can add attributes to the properties for the validation
 [Required]
 [Range(1, 100)]
 [StringLength(100, MinimunLenght = 10)]
 
If we want the client will be unique we can do this in the Post method

```C#
var client = await _repository.GetClientAsync(model.Code);
if (client != null)
{ return BadRequest("This client exist") }
```

For the Put Verb we send in the querystring the code and in the body the changes to do
And the method will be so that

```C#
[HttpPut("{code}")]
public async Task<ActionResult<ClientModel>> Put (string code, ClientModel model)
{
  var oldClient = await _repository.GetClientAsync(code);
  if (oldClient == null) return NotFound($"Could not find client wiht code of {code}");
  
  _mapper.Map(model, oldClient);
  
  if (await _repository.SaveChangesAsync()) { return _mapper.Map<ClientModel>(oldClient); }
}
```

- Associate Controller
The association hierarchy can be deep or shallow
We can have an associate controller, and the attribute will be so:

```C#
[ApiController]
[Route("api/clients/{code}/invoices")]
```

And in the Get for example, add the parameter in the method

```C#
[HttpGet]
public async <Task<ActionResult<InviceModel[]>> Get(string code) {}
```

For an individual invoice, the method will be so

```C#
[HttpGet("{id:ing}")]
public async Task<ActionResult<InvoiceModel>> Get(string code, int id) {}
```

- Functional APIs
REST defines URIs as resources, but execption exsit...
...don't be afraid of functionsl APIs, but avoid RPC style APIs at all costs

When sould you use Functional APIs?
- For operational needs (like clearing the cache, restarting servers, etc.)
- Avoid fro reporting

```C#
[HttpOptions("reloadconfig")] // This is a non-resource-based verb and specify the name of the verb
public IActionResult ReloadConfig(){
  try {
    var root = (IConfigurationRoot)_config; // IConfigurationRoot guarantees it's the root of the configuration
    root.Reload();
    return Ok();
  }
}
```

# CORS

- Es un mecanistmo que nos permite configurar restricciones sobre los recursos de nuestra API que sean requeridos por fuera del domino donde el recurso se encuentra hospedado.
- Es un estándar W3C
- Ayuda a evitar llamadas maliciosas a nuestros servicios o recursos

### Request Headers

- Origin
- Access-Control-Request-Method
- Access-Control-Request-Headers

### Response Headers

- Access-Control-Allow-Origin
- Access-Control-Allow-Credentials
- Access-Control-Expose-Headers
- Access-Control-Max-Age
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers

Ejemplo

```C#
app.UseCors(builder =>
 builder.WithOrigins("url")
 .WithMthods("GET", "POST", "PUT", "DELETE")
 .AllowAnyHeader())
 .Build();
```

