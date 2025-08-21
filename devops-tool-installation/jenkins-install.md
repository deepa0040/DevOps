Jenkins Install
Reference: 
- Jenkins: https://www.jenkins.io/doc/book/installing/linux/
- Java: https://www.jenkins.io/doc/book/installing/linux/#installation-of-java

Step1: Jenkins
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

Step2: Java
```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

Step3: Start Jenkins at boot
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```
