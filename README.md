# Traefik notes

Traefik architecture:
- Providers [file, k8, docker](like plugins for configurations)
- Static & dynamic configuration
- EntryPoint (Frontend)
- Service (Backend)
- Routers - Rules Host(`example.com`), link entrypoint to the service
- Middlewares (e.g. Compress) are optional
- Treafik starts with static config which points to the dynamic config
- Entrypoints binds to ip/address
- Service defines backend
-Touters maps entrypoint to service through rules

Static configurations include entry points and resolvers.
Dynamic configurations includes services, routers and middlewares.

Traefik looks for traefik.yaml for config file.



Examples:

## Layer 7 HTTP proxy

``` yaml
## Load balancing between two servers
## see all.yaml
http:
    routers:
        allBackendRouter:
            rule: "Host(`localhost`)"
            service: allBackend
    services:
        allBackend:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
                    - url: "http://localhost:2222/"
```

``` yaml
## Passing load through path
## see split.yaml
http:
    routers:
        allbackendrouter:
            rule: "Host(`localhost`) && Path(`/`)"
            service: allbackend
        app1router:
            rule: "Host(`localhost`) && Path(`/app1`)"
            service: app1backend
        app2router:
            rule: "Host(`localhost`) && Path(`/app2`)"
            service: app2backend
    services:
        app1backend:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
        app2backend:
            loadBalancer:
                servers:
                    - url: "http://localhost:2222/"
        allbackend:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
                    - url: "http://localhost:2222/"
```

``` yaml
## Block destination endpoint by Path
## see blockadmin.yaml
http:
    routers:
        allBackendRouter:
            rule: "Host(`localhost`)"
            service: allBackend
        adminblocker:
            rule: "Host(`localhost`) && Path(`/admin`)"
            service: blackhole
    services:
        blackhole:
            loadBalancer:
                servers:
        allBackend:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
                    - url: "http://localhost:2222/"
```



``` yaml
## Weighted Round Robin
## see wrrr.yaml
http:
    routers:
        allbackendrouter:
            rule: "Host(`localhost`)"
            service: wrr
        
    services:
        wrr:
            weighted:
                services:
                    - name: backend1
                      weight: 5
                    - name: backend2
                      weight: 1
        backend1:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
        backend2:
            loadBalancer:
                servers:
                    - url: "http://localhost:2222/"
```

## Layer 4 TCP proxy
``` yaml
tcp:
    routers:
        allBackendRouter:
            rule: "HostSNI(`*`)"
            service: allBackend
    services:
        allBackend:
            loadBalancer:
                servers:
                    - address: "localhost:1111"
                    - address: "localhost:2222"
```



## Other
``` yaml
## Jaeger tracing
tracing:
  jaeger:
    localAgentHostPort: 127.0.0.1:6831
```

``` yaml
## Basic Authorization
http:
    routers:
        allBackendRouter:
            rule: "Host(`localhost`)"
            service: allBackend
        allBackendRouterSecured:
            entryPoints:
                - http
            middlewares: 
                - auth
            rule: "(PathPrefix(`/dashboard`) || PathPrefix(`/api`))"
            service: api@internal
            
    services:
        allBackend:
            loadBalancer:
                servers:
                    - url: "http://localhost:1111/"
                    - url: "http://localhost:2222/"
                passHostHeader: false        
    middlewares:
        auth:
            basicAuth:
                users: 
                    - "admin:{SHA}2iRQKCF1byRkk6qCLh1AdHhU1tQ="
```

