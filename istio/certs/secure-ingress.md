## Generate Certificates 

1. Generate root certificate

openssl req -x509 -sha256 -nodes -days 365 \
-newkey rsa:2048 \
-subj '/O=Kubesure Inc./CN=kubesure.io' \
-keyout kubesure.io.key \
-out kubesure.io.crt

openssl x509 --in kubesure.io.ca.crt -text --noout

2. Generate Server cert and key from root cert

openssl req \
-out esyhealth.kubesure.io.csr \
-newkey rsa:2048 -nodes \
-keyout esyhealth.kubesure.io.key \
-subj "/CN=esyhealth.kubesure.io/O=esyhealth kubesure"

openssl x509 -req -days 365 \
-CA kubesure.io.crt \
-CAkey kubesure.io.key \
-set_serial 0 \
-in esyhealth.kubesure.io.csr \
-out esyhealth.kubesure.io.crt

openssl x509 --in esyhealth.kubesure.io.crt -text --noout

## client certs for MTLS

openssl req \
-out client.esyhealth.kubesure.io.csr \
-newkey rsa:2048 -nodes \
-keyout client.esyhealth.kubesure.io.key \
-subj "/CN=client.esyhealth.kubesure.io./O=client organization"

openssl x509 -req -days 365 \
-CA kubesure.io.crt \
-CAkey kubesure.io.key \
-set_serial 1 \
-in client.esyhealth.kubesure.io.csr \
-out client.esyhealth.kubesure.io.crt

## Simple TLS

kubectl delete -n istio-system secret esyhealth-credential

kubectl create -n istio-system secret tls esyhealth-credential \
--key=esyhealth.kubesure.io.key \
--cert=esyhealth.kubesure.io.crt

## Mutual TLS

kubectl delete -n istio-system secret esyhealth-credential
kubectl create -n istio-system secret generic esyhealth-credential \
--from-file=tls.key=esyhealth.kubesure.io.key \
--from-file=tls.crt=esyhealth.kubesure.io.crt \
--from-file=ca.crt=kubesure.io.crt