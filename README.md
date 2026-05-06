#  Secure Materials Simulation Pipeline (Encryption + Vault + Docker)

##  Project Overview

This project demonstrates a **secure scientific simulation workflow** for advanced materials research using:

- AES-256 Encryption
- HashiCorp Vault (Key Management)
- Role-Based Access Control (RBAC)
- Docker Containerization
- Secure Output Persistence

---

## System Architecture Flow


Dataset → Encryption → Vault Key Storage → Access Control → Docker Execution → Decryption → Simulation → Secure Output Storage


---


```bash
STEP 1 — Create Dataset
nano graphene_simulation_data.txt
Sample Data
Material: Graphene Composite
Density: 2.26 g/cm3
Thermal Conductivity: 5000 W/mK
Elastic Modulus: 1 TPa
Confidentiality: HIGH

---

STEP 2 — Encrypt Dataset (AES-256)
openssl enc -aes-256-cbc -salt -in graphene_simulation_data.txt -out graphene_simulation_data.enc -pass stdin
Explanation
AES-256 encryption ensures confidentiality
Password is provided at runtime (no hardcoding)

----

STEP 3 — Start Vault (Key Management)
Download Vault. 
wget https://releases.hashicorp.com/vault/1.15.2/vault_1.15.2_linux_amd64.zip 
Install unzip. 
sudo apt install unzip 
Unzip: 
unzip vault_1.15.2_linux_amd64.zip 
Move binary: 
sudo mv vault /usr/local/bin/
Start Vault
vault server -dev

New terminal:

export VAULT_ADDR='http://127.0.0.1:8200'
vault login

----

STEP 4 — Store Encryption Key in Vault
vault kv put secret/material_key key=$(openssl rand -base64 24)
Explanation
Random key generated dynamically
Stored securely inside Vault

----

STEP 5 — Access Control (RBAC)
nano policy.hcl
Policy File
path "secret/data/material_key" {
  capabilities = ["read"]
}

Apply policy:

vault policy write research-policy policy.hcl

Create token:

vault token create -policy=research-policy
Explanation
Only authorized users can access encryption key

----

STEP 6 — Install Docker
sudo apt install docker.io -y
sudo systemctl start docker

---

STEP 7 — Dockerfile
nano Dockerfile
Add:
FROM ubuntu:22.04

RUN apt update && apt install -y openssl

WORKDIR /research

COPY graphene_simulation_data.enc .

RUN mkdir -p /research/output

CMD ["sh", "-c", "openssl enc -aes-256-cbc -d -in graphene_simulation_data.enc -out /research/output/decrypted.txt -pass pass:$VAULT_KEY"]
---


----

STEP 8 — Build Docker Image
docker build -t secure-materials .

----

STEP 9 — fetch key from VAULT securely
export VAULT_KEY=$(vault kv get -field=key secret/material_key)
# explaination: this command will fetch key securely
STEP 10 — Run Container (Persistent Output)
docker run -e ENC_KEY=$VAULT_KEY -v $(pwd)/output:/research/output secure-materials
-e ENC_KEY → sends key inside the container
Output Generated
output/simulation_output.txt

----
Final Project Structure
secure-material-simulation/
│
├── graphene_simulation_data.txt
├── graphene_simulation_data.enc
├── Secret Management (HashiCorp Vault)
├── Dockerfile
├── policy.hcl
└── output/
    └── simulation_output.txt
