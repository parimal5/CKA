# note: -in file-name (order matters)

openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text

helm show values <chart> | yq ea '.. | path | join(".")' | grep -i nodePort
