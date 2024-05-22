# Maximize Your Development Potential: Free Red Hat Developer Subscription for Custom Apache Server Deployment on RHEL in the Cloud

As a developer, having access to powerful tools and platforms is crucial for building, testing, and deploying applications. Red Hat offers a no-cost developer subscription that allows individual developers to use up to 16 systems for demos, prototyping, QA, small production uses, and cloud access. In this blog post, I will guide you through registering for the subscription and demonstrate some of the new features of Red Hat cloud services that are particularly suited for developers. Let's walk through a practical example where we create a custom RHEL image that transforms into an Apache web server serving a "Hello, World!" page, and deploy it on a public cloud.

## Registering for the No-Cost Developer Subscription

To get started, follow these steps to register for the Red Hat Developer Subscription:

1. Visit the [Red Hat Developer Subscription page](https://developers.redhat.com/register).
2. Sign up for a Red Hat account if you don't already have one.
3. Log in and navigate to the subscription management page.
4. Activate your no-cost developer subscription.
5. Verify your new user and try to log in to [Red Hat Console](https://console.redhat.com) 

## Using Red Hat Cloud Services

### 1. Create and Customize a Red Hat Linux Image via blueprints

Red Hat cloud services allow you to create and customize Red Hat Enterprise Linux (RHEL) images using blueprints. Create a blueprint for your **golden image**, modify it over time as your needs change, and use it to build and deploy images on demand. Here's how to do it, Let's create a blueprint for our Apache Demo:

![blueprint wizard](blueprint_wizard.png)

1. **Create a Blueprint:**
   - Log in to the Red Hat Hybrid Cloud Console.
   - Navigate to the [Image Builder](https://console.redhat.com/preview/insights/image-builder) service and create a new blueprint.
   - Click on **Create blueprint** button
   - Choose your desired RHEL version, architecture, and target environments. For this demo we use RHEL 9, x86_64 and Google Cloud Platform accordingly.

2. **Setting Google Cloud Source:**
   --TODO--

3. **Register your system**
   - Your no cost subscription allows up to 16 systems
   - Open the Activation key dropdown and create a default key if there is no key. 

4. **OpenSCAP profile:**
   - This step allows you to add openSCAP profile for your image, in our demo we skip this step. Click **next**.

5. **File system configuration:**
   - This step configures the partitioning of the image.
   - Keep the recommended automatic partitioning for this Demo.

6. **Manage content**
   This step allow you to customize the repositories and packages to build the image.
   Go to **Additional packages** search and add these packages:
   - httpd: Apache server
   - ansible-core: For running first boot playbook
   - rhel-system-roles: For using RHEL ansible roles

7. **First boot**
   This step configure the image with a custom script that will execute on its first boot. The script can be shell, python, yml, etc. Just add on top shebang string.
   For this demo we will use Ansible playbook for transforms the system into Apache server:

     ```yml
     #!/usr/bin/ansible-playbook

     ---
    - hosts: localhost
    connection: local
    become: yes
    gather_facts: yes

    tasks:
        - name: Ensure firewalld is installed
        yum:
            name: firewalld
            state: present

        - name: Ensure firewalld is started and enabled
        systemd:
            name: firewalld
            state: started
            enabled: true

        - name: Install Apache
        yum:
            name: httpd
            state: present

        - name: Start and enable Apache
        systemd:
            name: httpd
            state: started
            enabled: true

        - name: Deploy Hello World HTML page
        copy:
            content: |
            <html>
            <head>
                <title>Hello World</title>
            </head>
            <body>
                <h1>Hello World</h1>
                <p>This is a basic HTML page served from Apache on a RHEL machine in GCP with <a href='https://console.redhat.com'>Image builder service</a>.</p>
            </body>
            </html>
            dest: /var/www/html/index.html
        notify:
            - restart apache

        - name: Configure firewall for web console
        include_role:
            name: redhat.rhel_system_roles.firewall
        vars:
            firewall:
            services:
                - name: http
                state: enabled

        - name: Correct SELinux context for custom HTML page
        command: restorecon -v /var/www/html/index.html

    handlers:
        - name: restart apache
        systemd:
            name: httpd
            state: resta/varrted
        
8. **Save and build the blueprint**
   After giving a name and description to our new blueprint, have a double check on the review section and save your blueprint. Open the save button dropdown and click **Save changes and build image** this creates the blueprint and also build the image. Red Hat makes it easy to manage your blueprints for future use:
   
    **Store Blueprints:**
   - All your blueprints are stored in the Red Hat Hybrid Cloud Console, allowing you to reuse and modify them as needed.

   **Export and Import Blueprints:**
   - Export blueprints for sharing with your team or importing into other projects. This feature is particularly useful for maintaining consistency across multiple workloads. 


### 2. Deploying to public cloud

In this demo we will deploy our Apache server to Google cloud, keep in mind that you can deploy it to AWS or Azure as well. 

The build process takes a few minutes, once the image has been built successfully, the **Launch** button appears, click it to open the launch wizard, just make sure that you configured the google cloud source before.

![launch wizard](launch_wizard.png)

7. **Compute configuration**
  - Select your google cloud source.
  - Select a machine type, you can filter by vcpus, memory and capacity. Type *vcpus=1 and memory>2000* and pick **n1-standard-1** for this demo.
8. **SSH key**
Upload a public key or choose an existing one, to avoid failures use **ed25519** 
 - You can create a new public key by running ```ssh-keygen -t ed25519```
9. **Launch**
   - Review and launch your new Apache server!
   - This process might take a minute, once it finished a table with IP address and ssh command will be shown. Copy your new Apache server IP Address.

### 3. Expose HTTP connection
*Reminder: this is a demo for dev or demo environment, not production*

To allow HTTP connection in our GCP instance, log in to your GCP console -> VM Instances -> Edit your new instance -> Enable Allow HTTP traffic in Network interfaces section.

And that's it! your new Apache server is running with your custom RHEL image!

![hello world page](hello_world.png)