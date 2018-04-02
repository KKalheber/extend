## Isolation Requirements

If you are operating a multi-tenant SaaS platform and use Auth0 Extend to allow your customers to extend your product with custom code, one key decision you need to make is related to the scope of isolation for executing extensions. Scope of isolation implies the following guarantees in the Auth0 Extend system: 

1. Two extensions executing in different scopes of isolation are guaranteed to be isolated in terms of in-memory state, access to disk, and execution integrity (e.g. one extension cannot affect the processing of the other extension). They are also guaranteed their fair share of computing resources (CPU, memory). 

2. Two extensions execution within the same scope of isolation may or may not share in-memory state, access to disk, and be able to affect each other's processing. This allows for a level of processing optimization between extensions (e.g. in-memory connection polling to external systems), but it cannot be relied upon as a guarantee. 

For example, if user A and user B of your system both author extension code, you will typically want Auth0 Extend to execute these extensions isolated from one another. This way code of user A cannot access any data or affect processing of user B. On the other hand, if user A authors multiple extensions in your system, it may be not only reasonable but desired to execute both extensions with no isolation so that they can share in-memory data or optimize connection management to external systems. 

The question you need to ask yourself in the context of your platform is: what is the concept in my platform which defines a trust or isolation boundary for data and processing? Here are a few possible answers and considerations: 

1. **Tenant**. For majority of multi-tenant SaaS platforms, a tenant of the platform is at the same time the most granular trust and isolation boundary. A tenant can be a subscriber, an account, an organziation, or even an individual user. As long as all configuration and secret data can be shared across code working on behalf of a specific tenant, but not shared with code working on behalf of other tenants, a tenant is likely your desired scope of isolation for Auth0 Extend extensions.  

2. **User**. In some multi-tenant systems, a tenant represents a high level account or organization, and there are many users or administrators who are part of it. If your system allows all those users to create their own extensions, and it is important that they are isolated from one another, a user may be the desired scope of isolation. 

3. **Staging vs production**. If your system differentiates between extensions that execute in a staging vs production environment, it is likely desirable for the execution contexts of these extensions to be isolated. This way a bug that affects processing in the staging environment will not affect execution of extensions in production. 

4. **Single extension**. In an extreme case, you can say that every extension in the system must be completely isolated from any other extension, without regard for the fact that security of the system may not require that level of granularity. One reason to require this level of isolation is optimizing for stability: ensuring that execution of one extension will never affect execution of other extensions (for example due to an uncaught exception). One disadvantage of this approach is that it prevents optomizations that rely on sharing in-memory data between extensions. Another is related to scalabiltiy of the system (see below). 

Choice of the scope of isolation has impact on the scalability and performance of the system: the more granular scope of isolation, the fewer extensions per second you will be able to process given a specific size of an Auth0 Extend deployment. Supporting more isolated environments for extension execution requires more system resources. Scope of isolation set at the tenant level will consume fewer resources than scope of isolation set at user or extension level, assuming there are many users or extensions per tenant. 

In most cases, scope of isolation set at tenant level is adequate. 