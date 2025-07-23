# Jenkins Full CI/CD Pipeline for Node.js + MongoDB App

This project demonstrates a complete CI/CD pipeline built with **Jenkins**, using a Node.js application with a MongoDB backend. The pipeline automates building, testing, and Deployment.

## 🧰 Stack
- **Jenkins** (CI/CD automation)
- **Node.js** (Application)
- **MongoDB** (Database)
- **Jenkins Plugins**:
  - Git
  - Blue Ocean
  - NodeJS Plugin

---

## 📁 Project Structure

```bash
.
├── App-SourceCode
├── Jenkinsfile
├── .gitignore
└── README.md
```
## Continuous Integration
- Using Feature based branch (New Branch for new features)
- Develop the feature and build test (Unit/integration)
- **Scan Dependencies** -> Linting -> **Sast(Quality Gate)** -> Build Image -> **Image Scanning**
<div style="text-align: center;">
<img src="./Images/CI.png" alt="Jenkins" width="750" height="350" style="border-radius: 15px;">
</div>

> ###  CI Pipeline Stages
- Checkout Code
- Install Dependencies
- Lint Code
- Dependency Scanning
- Unit Testing
- Code Coverage


- > Skills Applied
  - *tools*, *parameters*, *environment* & *Options*
  - Credentials:
    - `withCredentials`
    - Using `Credentials()` in `environment{}`
  - `catchError`
  - `publishHTML`





## Continuous Deployment
- `Deploy` the new feature in dev Env
- Run `integration testing`
- After success, Create `Pull Request` for review `PR`
<div style="text-align: center;">
<img src="./Images/CD.png" alt="Jenkins" width="600" height="300" style="border-radius: 15px;">
</div>

## Continuous Delivery
- update configuration (ImageTag)
- DAST testing
- After success, Approve `Pull Request`
- Deploy to production using strategies

<div style="text-align: center;">
<img src="./Images/CDelivery.png" alt="Jenkins" width="750" height="350" style="border-radius: 15px;">
</div>

## Post Build
- Collect reports
- Notify admin using slack/email
<div style="text-align: center;">
<img src="./Images/Postbuild.png" alt="Jenkins" width="750" height="350" style="border-radius: 15px;">
</div>


## Best Practice


### Linting Stage
Linting is the process of analyzing your code for *potential errors*, *style issues*, and *bad practices* without executing it.
- In Node.js, we typically use `ESLint` popular JavaScript linter.