### **Deploying a Static Web Application in Jenkins using the "Publish Over SSH" Plugin**

#### **1. Launch Two EC2 Instances**

- One for **Jenkins** (CI/CD Server)
- One for **Deployment** (Web Server)

#### **2. Install Jenkins on the Jenkins Server**

- Follow standard Jenkins installation steps.
- Ensure Jenkins is running.

#### **3. Install Apache (httpd) and PHP on the Deployment Server**

```sh
sudo yum install -y httpd php php-cli php-common
php -v  # Verify PHP installation
```

#### **4. Start and Enable the HTTPD Service**

```sh
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

#### **5. Setup Jenkins**

- Complete the initial setup.
- Install required plugins.

#### **6. Install the "Publish Over SSH" Plugin**

- Go to **Manage Jenkins** → **Plugins** → Install **Publish Over SSH**.

#### **7. Configure "Publish Over SSH"**

- Go to **Manage Jenkins** → **System**.
- Under **Publish over SSH**, click "Add".
  1.  **Key Authentication**
      - Copy the private key (`.pem`) content and paste it into the SSH Key field.
  2.  **Enter Remote Host Details**
      - **Name:** Any identifier
      - **Hostname:** Remote server’s public IP
      - **Username:** `ec2-user` (for Amazon Linux)
      - **Remote Directory:** Path where the files will be deployed (e.g., `/var/www/html`)
  3.  Click **Test Configuration** to verify SSH connectivity.

#### **8. Set Permissions for the Web Directory on the Deployment Server**

```sh
sudo chown -R ec2-user:ec2-user /var/www/html
sudo chmod -R 755 /var/www/html
```

#### **9. Create a Freestyle Jenkins Job**

- Navigate to **Jenkins Dashboard** → **New Item** → **Freestyle Project**.

#### **10. Configure Source Code Management (SCM)**

- Select **Git**.
- Enter your **GitHub repository URL**.
- Provide branch details (e.g., `main`).

#### **11. Add Build Steps**

- Click **Add build step** → **Send files or execute commands over SSH**.

  1.  **Transfer Source Files**
      - **Source files:** `**/*` (Transfers all files)
  2.  **Execute Post-Deployment Commands**

      - **Exec Command:**
        ```sh
        sudo systemctl restart httpd
        ```

  3.  **Advanced Options (if required)**
      - Check **Use SFTP for Exec** (if needed).

#### **12. Save the Job Configuration.**

#### **13. Click "Build Now" to Trigger Deployment.**

#### **14. Verify Deployment**

- Open the web application using the **public IP** of the Deployment Server in a browser.
