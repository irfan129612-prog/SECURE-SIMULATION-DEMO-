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
openssl enc -aes-256-cbc -salt \
-in graphene_simulation_data.txt \
-out graphene_simulation_data.enc \
-pass stdin
Explanation
AES-256 encryption ensures confidentiality
Password is provided at runtime (no hardcoding)

----

STEP 3 — Start Vault (Key Management)
vault server -dev

New terminal:

export VAULT_ADDR='http://127.0.0.1:8200'
vault login

----

STEP 4 — Store Encryption Key in Vault
vault kv put secret/material_key key=$(openssl rand -base64 24)
🔐 Explanation
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

RUN apt update && apt install -y python3 openssl

WORKDIR /research

COPY graphene_simulation_data.enc .
COPY simulate.py .

RUN mkdir -p /research/output

CMD ["python3", "simulate.py"]

---

STEP 8 — Simulation Script
nano simulate.py
Python Script
import subprocess

# Key fetched securely (no hardcoding in production)
key = input("Enter decryption key: ")

# Decrypt dataset
subprocess.run([
    "openssl", "enc", "-aes-256-cbc", "-d",
    "-in", "graphene_simulation_data.enc",
    "-out", "decrypted.txt",
    "-pass", f"pass:{key}"
])

# Read decrypted data
with open("decrypted.txt", "r") as f:
    data = f.read()

# Save secure output
with open("/research/output/simulation_output.txt", "w") as f:
    f.write("SECURE SIMULATION RESULT\n\n")
    f.write(data)

print("✔ Simulation completed securely.")

----

STEP 9 — Build Docker Image
docker build -t secure-materials .

----

STEP 10 — Run Container (Persistent Output)
docker run -v $(pwd)/output:/research/output secure-materials

Output Generated
output/simulation_output.txt

----
Final Project Structure
secure-material-simulation/
│
├── graphene_simulation_data.txt
├── graphene_simulation_data.enc
├── simulate.py
├── Dockerfile
├── policy.hcl
└── output/
    └── simulation_output.txt
