# **Ansible Deployment Project**

## **Overview**
This project automates the deployment of application containers using Ansible. It provisions EC2 instances on AWS, configures Docker, and deploys multiple application instances on `dev` and `staging` environments using Docker Compose. The project follows production-grade best practices, ensuring idempotency, scalability, and maintainability.

---

## **Features**
- **Infrastructure Provisioning**:
    - Creates and configures EC2 instances (`dev` and `staging`) using the `amazon.aws.ec2_instance` module.
    - Manages security groups to allow SSH and application API traffic.
    - Attaches and configures EBS volumes to instances for persistent storage.

- **Application Deployment**:
    - Deploys 3 instances of the application container using Docker Compose on each EC2 instance.
    - Configures application-specific parameters (CPU, memory, storage, and API port) dynamically based on environment variables.
    - Implements persistent storage using a shared Docker volume (`secret-keys-volume`).

- **Idempotency**:
    - Ensures tasks are executed only when required to avoid unnecessary changes (e.g., Docker containers are recreated only when needed).

- **Linting and Standards**:
    - Configured with `ansible-lint` and `yamllint` for enforcing best practices and coding standards.

---

## **Project Structure**

```
├── ansible.cfg
├── group_vars
│         ├── all.yml
│         ├── dev.yml
│         └── staging.yml
├── inventories
│         ├── dev
│         │         └── hosts.yml
│         ├── localhost.yml
│         └── staging
│             └── hosts.yml
└── playbooks
    ├── debug-vars.yml
    ├── host_vars
    │         ├── dev_host1.yml
    │         └── staging_host2.yml
    ├── roles
    │         ├── common
    │         │         └── tasks
    │         │             └── main.yml
    │         ├── docker
    │         │         └── tasks
    │         │             └── main.yml
    │         ├── firewall
    │         │         └── tasks
    │         │             └── main.yml
    │         ├── geerlingguy.docker
    │         │         ├── LICENSE
    │         │         ├── README.md
    │         │         ├── defaults
    │         │         │         └── main.yml
    │         │         ├── handlers
    │         │         │         └── main.yml
    │         │         ├── meta
    │         │         │         └── main.yml
    │         │         ├── molecule
    │         │         │         └── default
    │         │         │             ├── converge.yml
    │         │         │             └── molecule.yml
    │         │         ├── tasks
    │         │         │         ├── docker-compose.yml
    │         │         │         ├── docker-users.yml
    │         │         │         ├── main.yml
    │         │         │         ├── setup-Debian.yml
    │         │         │         └── setup-RedHat.yml
    │         │         └── vars
    │         │             ├── Alpine.yml
    │         │             ├── Archlinux.yml
    │         │             └── main.yml
    │         └── provision
    │             └── tasks
    │             └── main.yml
    ├── site.yml
    └── templates
        └── docker-compose.yml.j2
```

## **Setup and Usage**

### **1. Prerequisites**
Ensure the following tools are installed on your local machine:
- **Python 3.13+**
- **pip** (Python package manager)
- **Ansible 2.18+**
- **AWS CLI** (Configured with appropriate credentials)
- **yamllint** and **ansible-lint** for linting.

Install required Python packages:
```bash
pip install boto boto3 ansible-lint yamllint
```

Install required Ansible roles:
```bash
ansible-galaxy install -r requirements.yml
```

---

### **2. Configuration**

1. **AWS Credentials**:
   Ensure you have the AWS credentials file configured in `~/.aws/credentials`.

2. **Inventory Files**:
   Update the `hosts.yml` files in the `inventories/dev` and `inventories/staging` folders with the correct details after provisioning.

3. **Group Variables**:
   Update `group_vars/dev.yml` and `group_vars/staging.yml` with environment-specific parameters:
   `yaml
   # Example (group_vars/dev.yml)
   app_image: nginx:latest
   app_instances: 3
   app_memory_limit: "1g"
   app_cpu_limit: "0.3"
   external_url: "https://dev/approve"
   debug: true
   `

4. **Docker Compose Template**:
   Modify `templates/docker-compose.yml.j2` if the application image or structure changes.

---

### **3. Deployment**

To deploy the project, follow these steps:

#### **Step 1: Provision EC2 Instances**
Run the following playbook to provision the required EC2 instances in the **dev** and **staging** environments and to deploy the application on the provisioned instances:
```bash
ansible-playbook -i inventories/ playbooks/site.yml
```

---

### **4. Linting**

#### **Yamllint**:
Use `yamllint` to check YAML syntax:
```bash
yamllint playbooks/ roles/ inventories/
```

---

### **5. Validation**

After deployment, validate the application:
1. Verify that the Docker containers are running:
   ```bash
   docker ps
   ```

2. Test the application endpoints for **dev** and **staging**:
    - Dev: `http://<dev_instance_ip>:8181`
    - Staging: `http://<staging_instance_ip>:8181`

3. Confirm the persistent volume (`secret-keys-volume`) is correctly mounted and persists data across container restarts.
