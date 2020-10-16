## TODO

1. Ingress
   - [] JWT Request AuthN (keycloak) & Authz  - Done. Need to figure out how jwksUri hostname can be a ingress ip and not node ip and nodeport  
   - [] TLS endpoint named host and certs 
   - [] MTLS
   - [] Rate Limits
2. Virtual service - Taffic management 
   - [] A/B routing 
   - [] Version base canaries  
   - [] Host base routes
   - [] Weighted versions
   - [] Rate limits 
   - [] timeouts
   - [] circut breakers
   - [] Service entry for headless services
   - [] Service entry for VM based IP's
3. Security services
   - [] Strict MTLS & secure naming for stateless servcies 
   - [] Disabled MTLS for statelful sets
   - [] Port base destination routes
4. Egress
   - [] ???         
