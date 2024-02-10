# Passworm - Infrastructure
All infrastructure definitions related to the [Passworm app](https://github.com/UrskaN/passworm-app/tree/main) project will be available in this repo.

Information about app functionality and settings can be found within the [app repo](https://github.com/UrskaN/passworm-app/tree/main). Some exaples of the app in action can be found in [this Google Slides](https://docs.google.com/presentation/d/1-V0qDYywvR-MJL4Bw_PlQB0Ugi-8lQZwFLaD6qjs640/edit?usp=sharing) presentation.

## Virtualization with Cloud-init Script
All actions and steps that need to be performed within a provided VM where the application should run are scripted as a cloud-init script. Key steps are:
- Configuration of deploy keys and cloning of source code from a private repo.
- Installation of all required modules.
- Configuration of environmental variables required by the app.
- Routing from port 80 to port 3000.

All secrets are hardcoded into the script. For an actual production application they would be stored within a vault or something and retrieved in a secure way. But this will have to do for now.

Script has been tested with Multipass 1.12.2+mac for a VM running Ubuntu 22.04.3 LTS. All VM configuration parameters have been left as default values since the demo app is not really demanding. To launch a VM using the provided cloud-init script, adjust the name of the VM and run: 
```
multipass launch jammy --name <vm-name> --cloud-init cloud-config.yml
```

Note: Multipass might coplain that launch failed with the following error: `timed out waiting for initialization to complete`. However the VM will actually run as expected (as well as the app within).

If all goes as expected, the app will be available on IP assigned to the VM.
![Screenshot of the Passworm login page running in a local VM.](<img/passworm-screenshot.png>)

### Ideas for Improvement
- Configure static IP for the VM


## Conteinerization
Included docker-compose.yml file launches the app and related services as Docker containers:

1. Passworm - the main thing, a node.jst app for password sharing
2. MongoDB - database to store app data
3. NGINX - reverse proxy to access the app
    - configuration is defined in nginx.conf
4. Mailhog - for development and testing of email notifications

### System Requirements
- docker
- docker compose

### Private Repo Access
To pull and run passworm-app image, access to private repo is requred to login into repository. In case of GitHub Container Repository an access token must be issued with appropriate permissions and passed as password on login.

### Launch the Stack
Create an .env file and fill in the values defined with sample values in .env.example or configure reqired settings in some other way. You can change the target environment by setting appropriate value in service passworm > environment > NODE_ENV (development / test / production)

Then launch the stack by running
```
docker compose up -d
```

## Orchestration with Kubernetes
The whole stack previously defined in docker-compose.yml is now set up to be deployed onto a Kubernetes cluster. This includes:
1. Passworm - 3 replicas
2. MongoDB - 1 replica
3. Mailhog - 1 replica
4. NGINX as Ingress

Application is available at https://passworm.essa-vm-04.lrk.si/login

All manifest files are in `kubernetes/manifests` directory. 

### Sealed Secrets
To keep things in the spirit of GitOps, all secrets are encrypted and commited along with other files. Once deployed, secrets are decrypted in cluster and applied. Sealed-secrets (serverside) and kubeseal (locally) are used for this purpose. 

### Pointers to Other Reqirements
- cert-manager is defined and included in Ingress definition
- PersistentVolume and PVC are defined and used with MongoDB
- image for Passworm app is custom and built using Github Actions pipeline (previous assignment). It now uses tini as PID 1 process and runs app as nonroot user.
- readiness probes are added to the app
    - liveness: always returns 200 (if it's capable of responding, it's live)
    - readines: positive response when database is connected (will stop responding with 200 if the connection is dropped)

### Deployment & GitOps
Any change in files in directory `kubernetes/manifests` on main branch will be automatically deployed to the cluster using ArgoCD. Definitions for its resources are in `kubernetes/argocd`.

#### Rolling Update
Updated passworm deployment to use newer version of docker image (updated from 2.0.0 to 2.0.1) with 1 additional pod allowed and no fewer than original 3.

![Screenshot of rolling update in action.](<img/rolling.gif>)

#### Blue/Green Deployment
New version of passworm (image version 2.1.1) app was deployed as v2 before switching ingress to service passworm-v2. Previous version is left running to allow for easy rollback.

![Screenshot of deployments just after creating v2.](<img/blue-green-depl.png>)

![Screenshot of pods after v2 pods have been created.](<img/blue-green-pods.png>)