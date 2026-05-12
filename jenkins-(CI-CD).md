# ✅ AWS EC2 Instance Setup

## 1️⃣ Launch Two EC2 Instances:

- `devm` (Dev Machine)
- `setm` (SIT Server)
- Choose **Ubuntu** as the OS and **allow HTTP traffic** for both.

---

## 🧰 Jenkins Setup on `devm`

### 🔧 Step 1: Update and Install Java 21

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
````

---

### 🔧 Step 2: Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
```

### 🔑 Step 3: Get Jenkins Initial Password

```bash
sudo systemctl status jenkins
```

### 🌐 Step 4: Access Jenkins on Browser

Open in your browser:

```
http://<devm-public-ip>:8080
```

### ⚙️ Step 5: Initial Jenkins Setup

* Paste the password when prompted.
* Install **Suggested Plugins**.
* Create admin account.
* Keep remaining settings as default.

> ✅ You should now see the **Jenkins Dashboard**.

🎯 *Experiment: Jenkins setup is complete.*

---

## ⚙️ Maven Setup on `devm` (Build WAR File)

### 📥 Step 1: Download and Extract Maven

```bash
cd /opt
sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
sudo tar -xvzf apache-maven-3.9.9-bin.tar.gz
sudo mv apache-maven-3.9.9 maven
```

### 🛠️ Step 2: Configure Environment Variables

```bash
sudo vim ~/.profile
```

Add the following lines at the bottom:

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export M2_HOME=/opt/maven
export MAVEN_HOME=/opt/maven
export PATH=${M2_HOME}/bin:${PATH}
```

Then reload:

```bash
source ~/.profile
mvn --version
```

---

## 🧩 Configure Jenkins Tools

### 🔌 Step 1: Install Maven Integration Plugin

* Go to Jenkins dashboard → Manage Jenkins → Plugins.
* Click on **Available Plugins**.
* Search for **Maven Integration Plugin**.
* Select and click **Install**.

### ⚙️ Step 2: Configure JDK & Maven in Jenkins

* Go to **Manage Jenkins → Tools**.
* Add JDK:

  * Name: `java-21`
  * JDK Path: `/usr/lib/jvm/java-21-openjdk-amd64`
* Add Maven:

  * Name: `maven`
  * Maven Path: `/opt/maven`
* Click **Apply and Save**.

---

## 🛠️ Create Maven Job (WAR Project)

### 📝 Step 1: Create a New Job

* Go to Jenkins dashboard → **New Item** → Select **Maven Project**
* Give your project a name and click OK.

### 🔗 Step 2: Configure Git SCM

* Under **Source Code Management**, choose **Git**.
* Fork the repository: [https://github.com/Mahi-Repalle/practice](https://github.com/Mahi-Repalle/practice)
* Use the **HTTPS** URL from your forked repo and paste it in the **Repository URL**.

### 🏗️ Step 3: Add Build Goals

```bash
clean install
```

### 📦 Step 4: Run the Build and Check WAR Creation

> Note:
> If the job execution is taking a lot of time:
>
> * Log out of Jenkins
> * Stop the `devm` instance
> * Restart the instance
> * Start Jenkins
> * Run the job again

🎯 *Experiment: WAR file build is complete.*

---

## 🌐 Setup Tomcat on `setm`

### 🔧 Step 1: Install Tomcat

```bash
sudo apt update
sudo apt install -y tomcat9 tomcat9-admin
```

### 🔐 Step 2: Configure `tomcat-users.xml`
Goto /etc/tomcat9

```bash
cd /etc/tomcat9
```

Edit the file and add the following before the last line:
To edit
```bash
sudo nano tomcat-users.xml
```

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="admin" roles="manager-gui,manager-script"/>
```

### 🔁 Step 3: Restart Tomcat

```bash
sudo service tomcat9 restart
```

### 🌍 Step 4: Access Tomcat

```
http://<setm-public-ip>:8080
```

---

## 🔁 Deploy WAR from Jenkins to SIT

### 🔌 Step 1: Install Deploy to Container Plugin

* Go to Jenkins dashboard → Manage Jenkins → Plugins
* Under **Available**, search for:

  * `Deploy to container plugin`
* Select and **Install** it.

### ⚙️ Step 2: Add Post-Build Action in Job

* Go to Jenkins dashboard → Select the WAR job → Configure.
* Scroll to **Post-build Actions**.
* Click **Add Post-build Action** → Select **Deploy WAR/EAR to a container**.

Configure:

* WAR/EAR files: `**/*.war`
* Context path: `sit`
* Container: **Tomcat 9.x**
* Add Jenkins credentials:

  * Username: `admin`
  * Password: `admin`
* URL: `http://<setm-public-ip>:8080`

Click **Apply and Save**.

### ▶️ Step 3: Run the Job

* The WAR file should now be deployed to Tomcat.

### 🌐 Step 4: Verify in Browser

```
http://<setm-public-ip>:8080/sit
```

🎯 *Experiment: Deployment to SIT server is complete.*

---

## 🤖 Jenkins CI/CD Automation Setup

### 🔁 Enable SCM Polling (or GitHub Webhook)

* Go to Jenkins Dashboard → Select Job → Configure
* Scroll to **Build Triggers**
* Check **Poll SCM**
* In the schedule field, enter:

```
* * * * *
```

(5 asterisks with space in between for polling every minute)

* Click **Apply and Save**

### ⚙️ Make Code Changes to Trigger Build

Changes can be made:

1. **Via CLI** on your local machine
2. **Using GitHub GUI** on the website

> Any change in the repository (e.g., modifying `index.jsp`) will trigger:
>
> * A new Jenkins build
> * Automatic deployment to the SIT server

🎯 *Jenkins CI/CD Automation is now functional.*

---

> ✅ All experiments are successfully completed from Jenkins installation to CI/CD deployment.
