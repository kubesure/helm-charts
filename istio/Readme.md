## TODO

1. Ingress
   - [X] JWT Request AuthN (keycloak) & Authz  - Done. Need to figure out how jwksUri hostname can be a ingress ip and not node ip and nodeport  
   - [X] TLS endpoint and certs
   - [X] MTLS premium 
   - [X] Rate Limits - premium - Rate limits are only possible as circut breaker to protect the client
2. Virtual service - Taffic management 
   - [] Version based canaries - premium quote/party 
   - [] Weighted versions - premium
   - [] Load balancing - Random quote/party 
   - [X] Host base routes - premium quote/party  
   - [X] Rate limits - premium Rate limits are only possible as circut breaker to protect the client
   - [X] timeouts - premium
   - [X] circut breakers premium
   - [] Service entry for headless services - premium redis
   - [] Service entry for VM based IP's - Policy Mysql
3. Security services
   - [X] Strict MTLS & secure naming for stateless servcies premium 
   - [X] Disabled MTLS for statelful sets with istio sidecare premium redis. Redis has replicaiton issue. 
   - [] Port base destination routes N/A as of now  
4. Egress
   - [] ???         

## ingress

kubectl get svc istio-ingressgateway -n istio-system

export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
export SECURE_GATEWAY_URL=$INGRESS_HOST:$SECURE_INGRESS_PORT


## mtls

curl -v -H "HOST: esyhealth.kubesure.io" \
-H "Content-Type: application/json" \
--cacert kubesure.io.crt \
--cert client.esyhealth.kubesure.io.crt --key client.esyhealth.kubesure.io.key \
--resolve "esyhealth.kubesure.io:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
https://esyhealth.kubesure.io:${SECURE_INGRESS_PORT}/api/v1/healths/premiums \
-d '{"code": "1A","sumInsured": "100000", "dateOfBirth": "1990-06-07"}' | jq

## secure simple tLS

curl -v -H "HOST: esyhealth.kubesure.io" \
-H "Content-Type: application/json" \
--cacert kubesure.io.crt \
--resolve "esyhealth.kubesure.io:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
https://esyhealth.kubesure.io:${SECURE_INGRESS_PORT}/api/v1/healths/premiums \
-d '{"code": "1A","sumInsured": "100000", "dateOfBirth": "1990-06-07"}' | jq

## insecure

curl -X POST http://esyhealth.kubesure.io:${INGRESS_PORT}/api/v1/healths/premiums \
--resolve "esyhealth.kubesure.io:$INGRESS_PORT:$INGRESS_HOST" \
-H "Content-Type: application/json" \
-H "HOST: esyhealth.kubesure.io" \
-d '{"code": "1A","sumInsured": "100000", "dateOfBirth": "1990-06-07"}' | jq

## Circuit breaker

kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio curl -quiet http://premiumcalc-svc:8000/api/v1/healths/ premiums -H "Content-Type: application/json" -H "HOST: esyhealth.kubesure.io" -d '{"code": "1A","sumInsured": "100000", "dateOfBirth": "1990-06-07"}'