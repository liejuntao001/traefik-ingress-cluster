

# Generate CA

cfssl gencert -initca ca/ca-csr.json | cfssljson -bare ca

# Generate certs

cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=config/ca-config.json \
    -profile=default \
    config/consul-csr.json | cfssljson -bare consul


cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=config/ca-config.json \
    -profile=default \
    config/traefik-csr.json | cfssljson -bare traefik

# Generate gossip key
export GOSSIP_ENCRYPTION_KEY=$(consul keygen)

# consul 
kubectl -n consul create secret generic consul \
  --from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
  --from-file=ca.pem \
  --from-file=consul.pem \
  --from-file=consul-key.pem


# traefik
kubectl -n kube-system create secret generic traefik-consul \
    --from-file=ca.pem \
    --from-file=traefik.pem \
    --from-file=traefik-key.pem
