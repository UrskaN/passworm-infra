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