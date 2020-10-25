## TODO

1. Ingress
   - [] JWT Request AuthN (keycloak) & Authz  - Done. Need to figure out how jwksUri hostname can be a ingress ip and not node ip and nodeport  
   - [] TLS endpoint and certs - done
   - [] MTLS premium - done
   - [] Rate Limits - premium - Rate limits are only possible as circut breaker to protect the client
2. Virtual service - Taffic management 
   - [] A/B routing - premium quote/party 
   - [] Version base canaries - premium quote/party 
   - [] Host base routes - premium 
   - [] Weighted versions - premium
   - [] Load balancing - Random quote/party 
   - [] Rate limits - premium Rate limits are only possible as circut breaker to protect the client
   - [] timeouts - premium  
   - [] circut breakers premium 
   - [] Service entry for headless services - premium redis
   - [] Service entry for VM based IP's - Policy Mysql
3. Security services
   - [] Strict MTLS & secure naming for stateless servcies quote/party 
   - [] Disabled MTLS for statelful sets with istio sidecare premium redis
   - [] Port base destination routes quote/party
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
