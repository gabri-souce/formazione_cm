📋 Formazione CM - Pipeline CI/CD con Ansible, Docker e Jenkins  !!!LEGGERE FILE info.txt!!!

🚀 Panoramica del Progetto

Questo progetto implementa una pipeline CI/CD completa che utilizza:

Ansible per l'automazione dell'infrastruttura
Docker per la containerizzazione
Jenkins per l'orchestrazione della pipeline
Registry Docker privato per la gestione delle immagini
📁 Struttura del Progetto

text
formazione_cm/
├── Jenkinsfile                 # Pipeline Jenkins principale
├── docker-compose.yml          # Docker Compose per tutti i servizi
├── container-playbook.yml      # Playbook Ansible principale
├── build-pipeline.yml          # Playbook per build applicazione
├── deploy-pipeline.yml         # Playbook per deploy applicazione
├── inventory.ini              # File inventory Ansible
├── .vault-pass                # Password per Ansible Vault (locale)
├── group_vars/
│   └── all/
│       ├── common.yml         # Variabili comuni
│       └── vault.yml          # Variabili cifrate (Ansible Vault)
├── files/
│   └── ssh-keys/              # Chiavi SSH generate automaticamente
├── images/
│   ├── alpine/                # Immagine Alpine con SSH
│   ├── ubuntu/                # Immagine Ubuntu con SSH
│   └── jenkins/               # Immagine Jenkins personalizzata
├── pipeline-app/              # Applicazione di esempio
└── roles/
    ├── registry/              # Ruolo per Docker Registry
    ├── container_build/       # Ruolo per build container
    └── container_deploy/      # Ruolo per deploy container
🛠️ Tecnologie Utilizzate

Ansible 2.14+ - Automazione infrastruttura
Docker 20.10+ - Containerizzazione
Jenkins 2.4+ - CI/CD Pipeline
Docker Registry 2.8+ - Registry privato
Alpine Linux 3.18+ - Container leggero
Ubuntu 20.04+ - Container completo
NGINX - Server web per applicazione demo
🚀 Funzionalità Implementate

✅ Step 1 - Docker Registry

Registry Docker privato su porta 5000
Storage persistente per le immagini
Accesso senza autenticazione (per development)
✅ Step 2 - Build Container Base

Container Alpine SSH: Immagine minimale con SSH abilitato
Container Ubuntu SSH: Immagine completa con SSH abilitato
Configurazione SSH con chiavi automatiche
Utente ansible-user con permessi sudo
✅ Step 3 - Ruoli Ansible

Ruolo Registry: Setup e configurazione Docker Registry
Ruolo Container Build: Build e push immagini base
Ruolo Container Deploy: Deploy e gestione container
✅ Step 4 - Ansible Vault

Cifratura delle credenziali sensibili
Password per utenti dei container
Credenziali di accesso sicure
✅ Step 5 - Jenkins Pipeline

Build automatica con tagging progressivo
Push automatico sul registry
Deploy automatizzato con Ansible
Integrazione con Git SCM
🔧 Installazione e Setup

Prerequisiti

bash
# Installare Docker e Docker Compose
sudo apt-get update
sudo apt-get install docker.io docker-compose

# Clonare il repository
git clone https://github.com/gabri-souce/formazione_cm.git
cd formazione_cm
Avvio dell'Infrastruttura

bash
# Avviare tutti i servizi con Docker Compose
docker-compose up -d

# Oppure avviare manualmente
docker run -d --name registry -p 5000:5000 -v registry-data:/var/lib/registry registry:2
Configurazione Jenkins

Accedere a Jenkins: http://localhost:8080
Ottenere la password iniziale:

bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Installare i plugin necessari:

Docker Plugin
Ansible Plugin
Git Plugin
Pipeline Plugin
SSH Agent Plugin
Configurare le credenziali:

SSH Key: ansible-ssh-key
Vault Password: ansible-vault-password
🎯 Utilizzo

Esecuzione Manuale dei Playbook

bash
# Eseguire il playbook completo
ansible-playbook -i inventory.ini container-playbook.yml --vault-password-file .vault-pass

# Solo build dei container
ansible-playbook -i inventory.ini container-playbook.yml --tags build --vault-password-file .vault-pass

# Solo deploy
ansible-playbook -i inventory.ini container-playbook.yml --tags deploy --vault-password-file .vault-pass
Accesso ai Container

bash
# Connettersi al container Alpine
ssh -i files/ssh-keys/id_rsa -p 2222 ansible-user@localhost

# Connettersi al container Ubuntu
ssh -i files/ssh-keys/id_rsa -p 2223 ansible-user@localhost
Verifica dello Stato

bash
# Verificare i container in esecuzione
docker ps

# Verificare le immagini nel registry
curl http://localhost:5000/v2/_catalog

# Verificare l'applicazione deployata
curl http://localhost:8081
🔒 Sicurezza

Ansible Vault

Le credenziali sensibili sono cifrate usando Ansible Vault:

bash
# Editare le variabili cifrate
ansible-vault edit group_vars/all/vault.yml --vault-password-file .vault-pass

# Visualizzare le variabili cifrate
ansible-vault view group_vars/all/vault.yml --vault-password-file .vault-pass
Variabili Cifrate

vault_registry_user: Utente del registry
vault_registry_password: Password del registry
vault_ssh_user_password: Password utente SSH
vault_jenkins_user: Utente Jenkins
vault_jenkins_password: Password Jenkins
📊 Pipeline Jenkins

Stage della Pipeline

Checkout Code: Clone del repository Git
Setup Environment: Configurazione chiavi SSH e vault
Verify Tools: Verifica strumenti installati
Build Base Images: Build immagini Alpine e Ubuntu
Build Application Image: Build applicazione con tagging
Deploy Application: Deploy automatico dell'applicazione
Verify Deployment: Verifica del deploy
Trigger Automatici

Build su push su branch main
Tagging progressivo con timestamp
Notifiche di successo/fallimento
🐛 Risoluzione Problemi

Problemi Comuni e Soluzioni

1. Permessi Docker

bash
# Fix permessi socket Docker
sudo chmod 666 /var/run/docker.sock
2. Registry non accessibile

bash
# Verificare che il registry sia running
docker ps | grep registry

# Riavviare il registry
docker restart registry
3. Chiavi SSH corrotte

bash
# Rigenerare le chiavi SSH
rm -f files/ssh-keys/id_rsa*
4. Problemi Ansible Vault

bash
# Verificare la password del vault
cat .vault-pass

# Rigenerare il file vault se necessario
echo "nuova-password" > .vault-pass
chmod 600 .vault-pass
📈 Monitoraggio

Logs dei Container

bash
# Logs di Jenkins
docker logs jenkins

# Logs del Registry
docker logs registry

# Logs dell'applicazione
docker logs pipeline-app
Stato dei Servizi

bash
# Verificare lo stato di tutti i container
docker-compose ps

# Verificare lo storage del registry
du -sh ~/registry-data/
🔮 Estensioni Future

Funzionalità da Implementare

Autenticazione sul Docker Registry
Scanner sicurezza per le immagini (Trivy)
Testing automatizzato dei container
Deploy su ambiente di produzione
Monitoraggio con Prometheus/Grafana
Notifiche via Slack/Email
Miglioramenti

Implementare HTTPS per il registry
Aggiungere health check ai container
Implementare backup automatici
Aggiungere documentazione API
👥 Contribuzione

Fork del progetto
Creare un branch per la feature (git checkout -b feature/AmazingFeature)
Commit delle modifiche (git commit -m 'Add AmazingFeature')
Push sul branch (git push origin feature/AmazingFeature)
Aprire una Pull Request
📝 Licenza

Questo progetto è licenziato sotto la Licenza MIT - vedere il file LICENSE per dettagli.

🤝 Supporto

Per problemi o domande:

Controllare la sezione Risoluzione Problemi
Aprire una issue
Contattare il maintainer del progetto
🎉 Successo!

La pipeline è ora completamente funzionante con:

✅ Build automatizzata di container multipli
✅ Registry Docker privato
✅ Pipeline CI/CD Jenkins
✅ Sicurezza con Ansible Vault
✅ Deploy automatizzato
Happy Coding! 🚀
