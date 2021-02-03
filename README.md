# API_REST_Fundamentals
### Notes from Building an API with ASP.NET Core by Shawn Wildermuth Pluralsight course.

How does HTTP Work?

Client send a Request has **_verb_** (which is you want to do), **_headers_** (which include additional information) and **_content._**
Then the server send a Response, There's a **_status code_** for whether it succedded or not, **_headers_** that might include information and it may also include **_content._**

More importants verbs
- GET: Retrieve a resource
- POST: Add a new resource
- PUT: Update an existing resource
- PATCH: Update an existing resource with set of changes
- DELETE: Remove the existing resource

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












