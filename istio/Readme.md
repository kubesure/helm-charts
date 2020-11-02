## TODO 1.1

1. Ingress
   - [X] JWT Request AuthN (keycloak) & Authz  - Done. Need to figure out how jwksUri hostname can be a ingress ip and not node ip and nodeport  
   - [X] TLS endpoint and certs
   - [X] MTLS premium 
   - [X] Rate Limits - premium - Rate limits are only possible as circut breaker to protect the client
2. Virtual service - Taffic management 
   - [X] Version based canaries - premium quote/party 
   - [X] Weighted versions - premium
   - [X] Load balancing - Random premium 
   - [X] Host base routes - premium quote/party  
   - [X] Rate limits - premium Rate limits are only possible as circut breaker to protect the client
   - [X] timeouts - premium
   - [X] circut breakers premium
   - [X] Service entry for VM based MSQL DB - Policy
   - [X] Service entry for external Kafka - Publisher
3. Security services
   - [X] Strict MTLS & secure naming for stateless servcies premium 
   - [X] Disabled MTLS for statelful sets with istio sidecare premium redis. Redis has replicaiton issue. 
    
4. Egress
   - [X] To external Kafka - Works only once after SE creation - need to raise and issue with Istio
      
## TODO 1.2

1. Virtual service 
   - [] Service entry for headless services within K8s - premium redis
2. Egress 
   - [] AWS S3
3. Security 
   - [] Port base destination routes 
4. [] configure all pending services in the mesh         

## ingress

kubectl get svc istio-ingressgateway -n istio-system

export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
export SECURE_GATEWAY_URL=$INGRESS_HOST:$SECURE_INGRESS_PORT


## mtls - V1

curl -v -H "HOST: esyhealth.kubesure.io" \
-H "Content-Type: application/json" \
--cacert kubesure.io.crt \
--cert client.esyhealth.kubesure.io.crt --key client.esyhealth.kubesure.io.key \
--resolve "esyhealth.kubesure.io:$SECURE_INGRESS_PORT:$INGRESS_HOST" \
https://esyhealth.kubesure.io:${SECURE_INGRESS_PORT}/api/v1/healths/premiums \
-d '{"code": "1A","sumInsured": "100000", "dateOfBirth": "1990-06-07"}' | jq

## mtls - V2

curl -v -H "HOST: esyhealth.kubesure.io" \
-H "Content-Type: application/json" \
-H "client-channel: mobile" \
-H "client-id: paytm" \
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