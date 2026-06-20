# Create a base pipeline

This folder holds the files for the lab _Create a Base Pipeline_ which is part of the **IBM-CD0215EN-Skills Network Introduction to CI/CD** course.
# Building a Tekton Pipeline using Python (Hands-On Lab)

Estimated time needed: 40 minutes

---

## Overview

Welcome to the hands-on lab for building a Tekton pipeline. In this lab, you will:

- Create a simple Tekton pipeline with one task
- Add parameters to tasks and pipelines
- Build a Git checkout task
- Extend a pipeline into a CI/CD workflow

You will also learn how Tekton enables reusable pipeline-as-code and how to structure pipelines for real-world DevOps workflows.

---

## Learning Objectives

By the end of this lab, you will be able to:

- Create a base Tekton pipeline and task
- Pass parameters into tasks and pipelines
- Clone a Git repository using Tekton tasks
- Build a multi-stage CI/CD pipeline

---

## Prerequisites

You should have:

- Basic understanding of YAML
- A GitHub account
- Intermediate CLI knowledge

---

## 1. Set Up Lab Environment

### Open Terminal

Go to:

Terminal → New Terminal

Then ensure you're in the project directory:

"bash  
cd /home/project  
"

---

### Clone Repository

"bash  
git clone https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git  
"

---

### Navigate to Lab Folder

"bash  
cd wtecc-CICD_PracticeCode/labs/01_base_pipeline/  
"

---

### Optional: Shorten Terminal Prompt

"bash  
export PS1=\"[\\[\\033[01;32m\\]\\u\\[\\033[00m\\]: \\[\\033[01;34m\\]\\W\\[\\033[00m\\]]\\$ \"  
"

---

## 2. Step 1: Create an Echo Task

Edit `tasks.yaml`.

### Task Definition

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Task  
metadata:  
  name: hello-world  
spec:  
  steps:  
    - name: echo  
      image: alpine:3  
      command: [/bin/echo]  
      args: [\"Hello World!\"]  
"

### Apply Task

"bash  
kubectl apply -f tasks.yaml  
"

---

## 3. Step 2: Create a Pipeline

Edit `pipeline.yaml`.

### Pipeline Definition

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Pipeline  
metadata:  
  name: hello-pipeline  
spec:  
  tasks:  
    - name: hello  
      taskRef:  
        name: hello-world  
"

### Apply Pipeline

"bash  
kubectl apply -f pipeline.yaml  
"

---

## 4. Step 3: Run Pipeline

"bash  
tkn pipeline start --showlog hello-pipeline  
"

Expected output:

"text  
Hello World!  
"

---

## 5. Step 4: Add Parameters to Task

Update `tasks.yaml`.

### Updated Task

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Task  
metadata:  
  name: echo  
spec:  
  params:  
    - name: message  
      type: string  
      description: The message to echo  

  steps:  
    - name: echo-message  
      image: alpine:3  
      command: [/bin/echo]  
      args: [\"$(params.message)\"]  
"

### Apply Changes

"bash  
kubectl apply -f tasks.yaml  
"

---

## 6. Step 5: Update Pipeline with Parameters

Update `pipeline.yaml`.

### Pipeline with Params

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Pipeline  
metadata:  
  name: hello-pipeline  
spec:  
  params:  
    - name: message  

  tasks:  
    - name: hello  
      taskRef:  
        name: echo  
      params:  
        - name: message  
          value: \"$(params.message)\"  
"

### Apply Pipeline

"bash  
kubectl apply -f pipeline.yaml  
"

---

## 7. Step 6: Run Parameterized Pipeline

"bash  
tkn pipeline start hello-pipeline \\  
  --showlog \\  
  -p message=\"Hello Tekton!\"  
"

Expected output:

"text  
Hello Tekton!  
"

---

## 8. Step 7: Create Checkout Task

Add separator:

"yaml  
---  
"

### Checkout Task

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Task  
metadata:  
  name: checkout  
spec:  
  params:  
    - name: repo-url  
      type: string  
      description: The URL of the git repo to clone  

    - name: branch  
      type: string  
      description: The branch to clone  

  steps:  
    - name: checkout  
      image: bitnami/git:latest  
      command: [\"git\"]  
      args: [\"clone\", \"--branch\", \"$(params.branch)\", \"$(params.repo-url)\"]  
"

### Apply

"bash  
kubectl apply -f tasks.yaml  
"

---

## 9. Step 8: Create CD Pipeline

### CD Pipeline Definition

"yaml  
apiVersion: tekton.dev/v1beta1  
kind: Pipeline  
metadata:  
  name: cd-pipeline  
spec:  
  params:  
    - name: repo-url  
    - name: branch  
      default: master  

  tasks:  
    - name: clone  
      taskRef:  
        name: checkout  
      params:  
        - name: repo-url  
          value: \"$(params.repo-url)\"  
        - name: branch  
          value: \"$(params.branch)\"  
"

### Apply

"bash  
kubectl apply -f pipeline.yaml  
"

---

## 10. Step 9: Run CD Pipeline

"bash  
tkn pipeline start cd-pipeline \\  
  --showlog \\  
  -p repo-url=\"https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git\" \\  
  -p branch=\"main\"  
"

Expected output:

"text  
Cloning into 'wtecc-CICD_PracticeCode'...  
"

---

## 11. Step 10: Add CI/CD Stages (Placeholder Tasks)

### Pipeline Stages

"yaml  
tasks:  

  - name: lint  
    taskRef:  
      name: echo  
    params:  
      - name: message  
        value: \"Calling Flake8 linter...\"  

  - name: tests  
    taskRef:  
      name: echo  
    params:  
      - name: message  
        value: \"Running unit tests with PyUnit...\"  

  - name: build  
    taskRef:  
      name: echo  
    params:  
      - name: message  
        value: \"Building image for $(params.repo-url)...\"  

  - name: deploy  
    taskRef:  
      name: echo  
    params:  
      - name: message  
        value: \"Deploying $(params.branch) branch of $(params.repo-url)...\"  
"

---

## 12. Final Run

"bash  
tkn pipeline start cd-pipeline \\  
  --showlog \\  
  -p repo-url=\"https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git\" \\  
  -p branch=\"main\"  
"

---

## Conclusion

You have successfully:

- Built a Tekton task
- Created CI/CD pipelines
- Passed parameters between pipelines and tasks
- Created a Git checkout task
- Simulated CI/CD stages

You now understand how Tekton pipelines form the foundation of Kubernetes-native CI/CD systems.