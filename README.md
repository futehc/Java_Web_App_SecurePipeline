# CI/CD Pipeline for Secure Web Application Deployment

## Overview
This Jenkins pipeline automates the build, test, security scanning, and deployment of a  java web application using Docker, Ansible, and Kubernetes.

## Features
- **Security Scans**
  - [OWASP Dependency Check](https://jeremylong.github.io/DependencyCheck/) for detecting vulnerable dependencies.
  - [TruffleHog](https://github.com/trufflesecurity/trufflehog) for scanning secrets in the repository.
  - [Checkov](https://www.checkov.io/) for Infrastructure as Code security analysis.
- **Static Code Analysis**
  - SonarQube integration for code quality and security analysis.
- **Automated Build and Test**
  - Maven-based packaging and JUnit test execution.
- **Deployment Automation**
  - Artifacts are pushed to an Ansible server for automated deployment.

## Pipeline Stages
1. **Security Scanning**
   - Runs OWASP Dependency Check, TruffleHog, and Checkov for security analysis.
2. **Static Code Analysis**
   - Uses SonarQube to analyze the source code.
3. **Build Web Application**
   - Packages the application using Maven.
4. **Test Web Application**
   - Executes unit tests and generates reports.
5. **Push Artifact to Ansible Server**
   - Transfers the packaged JAR file to a remote server.
6. **Deploy Application using Ansible**
   - Executes an Ansible playbook for automated deployment.

## Setup Instructions
1. Ensure you have the following tools configured:
   - Jenkins with necessary plugins installed:
     - GitHub Commit Status Setter
     - SonarQube Scanner
     - SSH Publisher
     - OWASP Dependency Check Plugin
   - SonarQube Server
   - Ansible Control Node
   - DockerHub Credentials stored in Jenkins
   - Azure Storage Account for Trivy reports

2. **Configure Jenkins Credentials**
   - Add required secrets under **Manage Jenkins > Credentials**:
     - `DOCKERHUB_CREDS`
     - `ANSIBLE_SLACK_TOKEN`

3. **Configure Webhooks (Optional)**
   - Enable GitHub webhooks for commit status updates.

4. **Run the Pipeline**
   - Manually trigger or configure Jenkins to run on code changes.

## Notifications
- Slack notifications are sent for success, failure, and unstable builds.

## Future Improvements
- Integrate dynamic secrets management.
- Implement a rollback mechanism in case of deployment failure.
- Add end-to-end tests using Selenium.

---
