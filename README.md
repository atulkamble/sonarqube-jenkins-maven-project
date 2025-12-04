Below is a **complete, production-ready SonarQube + Jenkins + Maven project setup** for **Amazon Linux EC2**, including:

✔️ EC2 setup
✔️ SonarQube installation
✔️ Jenkins installation
✔️ Maven installation
✔️ SonarQube + Jenkins + Maven integration
✔️ Full sample Java/Maven project with `sonar-project.properties`
✔️ Jenkins Pipeline (Declarative + Scripted options)

---

# 🟩 **1. Launch Amazon Linux EC2 & Install Dependencies**

### **Step 1 — Update System**

```bash
sudo yum update -y
sudo yum install git wget unzip -y
```

---

# 🟦 **2. Install Java 17 (Required for SonarQube 10+)**

```bash
sudo amazon-linux-extras install java-openjdk17 -y
java -version
```

---

# 🟧 **3. Install Maven**

```bash
sudo yum install maven -y
mvn -version
```

---

# 🟥 **4. Install SonarQube on Amazon Linux**

### **Create a dedicated user**

```bash
sudo useradd sonar
sudo passwd sonar
```

### **Download SonarQube**

```bash
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.0.89995.zip
sudo unzip sonarqube-10.5.0.89995.zip
sudo mv sonarqube-10.5.0.89995 sonarqube
sudo chown -R sonar:sonar /opt/sonarqube
```

### **Make SonarQube service**

```bash
sudo tee /etc/systemd/system/sonar.service <<EOF
[Unit]
Description=SonarQube service
After=network.target

[Service]
Type=forking
User=sonar
Group=sonar
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
EOF
```

### **Start service**

```bash
sudo systemctl daemon-reload
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```

### **Access SonarQube (Browser)**

```
http://<EC2-Public-IP>:9000
```

Login:

```
username: admin
password: admin
```

---

# 🟪 **5. Install Jenkins on Amazon Linux**

```bash
sudo yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum upgrade
sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

# 🟫 **6. Integrate Jenkins with SonarQube**

### **Install Plugins in Jenkins**

✔️ SonarQube Scanner
✔️ Maven Integration
✔️ Pipeline
✔️ GitHub

### **Add SonarQube Server**

**Jenkins → Manage Jenkins → System → SonarQube Servers**

| Field | Value                                         |
| ----- | --------------------------------------------- |
| Name  | `MySonarQube`                                 |
| URL   | `http://<EC2-IP>:9000`                        |
| Token | Generate from SonarQube → My Account → Tokens |

---

# 🟧 **7. Create Jenkins SonarQube Scanner Tool**

**Manage Jenkins → Global Tool Configuration**

Add:

* SonarQube Scanner
* Maven
* JDK 17

---

# 🟩 **8. Sample Java Maven Project (Push to GitHub)**

### 📁 **Project Structure**

```
sonarqube-maven-sample/
│── src/
│   └── main/java/com/example/App.java
│── src/test/java/com/example/AppTest.java
│── pom.xml
│── sonar-project.properties
```

---

# 📄 **pom.xml**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>sonarqube-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

---

# 📄 **sonar-project.properties**

```properties
sonar.projectKey=sonarqube-demo
sonar.projectName=SonarQube Maven Demo
sonar.projectVersion=1.0
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
```

---

# 🟦 **9. Jenkins Pipeline (Declarative)**

Create file: **Jenkinsfile**

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'jdk17'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/atulkamble/sonarqube-maven-sample.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

---

# 🟥 **10. Scripted Pipeline Option**

```groovy
node {
    stage('Checkout') {
        git 'https://github.com/atulkamble/sonarqube-maven-sample.git'
    }

    stage('Build') {
        sh 'mvn clean package'
    }

    stage('SonarQube Analysis') {
        withSonarQubeEnv('MySonarQube') {
            sh 'mvn sonar:sonar'
        }
    }

    stage('Quality Gate') {
        waitForQualityGate abortPipeline: true
    }
}
```

---
