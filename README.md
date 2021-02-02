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
