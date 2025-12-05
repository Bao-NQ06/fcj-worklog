---
title: "Worklog Week 9"
date: 2024-11-04
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives: 

* Complete the transition to the **AWS SAM (Serverless Application Model)** development model.
* **Refactor** and re-implement the basic CRUD functionalities according to the SAM structure.
* Resolve environment-related issues to achieve a **successful Deployment** status on AWS.
* Integrate **Docker** to standardize the environment for `sam build`.

---

### Tasks to be Deployed This Week:

| Day | Task | Start Date | Completion Date | Resources |
| :--- | :--- | :--- | :--- | :--- |
| Monday | - **In-depth research on AWS SAM:** Understand the `template.yaml` structure and how Serverless resources (Lambda, API Gateway) operate within the SAM model. <br> - Plan the detailed migration of existing source code to the SAM structure. | 04/11/2024 | 04/11/2024 | AWS SAM Documentation, AWS Study Group |
| Tuesday | - **Source Code Refactoring:** Start rewriting basic CRUD functionalities (create/update posts) using the SAM pattern (Handlers and Event Triggers). <br> - **Docker Integration:** Install and configure Docker to ensure the correct Python version for the `sam build` process. | 05/11/2024 | 06/11/2024 | Docker Documentation, SAM CLI |
| Wednesday | - **Local Debugging and Testing:** Execute `sam local invoke` and `sam local start-api`. <br> - **Encountered critical issues** in the Local environment (Dependency errors, environment conflicts, DynamoDB local connection issues). | 06/11/2024 | 07/11/2024 | SAM CLI Error Reports, Stack Overflow |
| Thursday | - **Strategic Decision:** The Backend Team decided to switch to a **deploy-then-test** strategy on the actual AWS environment to overcome local debugging barriers, accepting the high risk. <br> - Focus on fixing configuration errors in `template.yaml` in preparation for `sam deploy`. | 07/11/2024 | 08/11/2024 | |
| Friday | - **Successful Deployment:** Executed `sam deploy --guided` and finally deployed the project to the AWS environment. <br> - **Basic Verification:** Tested the created and functioning API Endpoints, confirming CRUD functionality is online. | 08/11/2024 | 08/11/2024 | AWS CloudFormation Deployment Logs |

---

### Week 9 Achievements: 

* **Completed the technology transition** to the **AWS SAM** development model for the entire project.
* **Successfully refactored** the basic CRUD functionalities into the SAM Serverless structure.
* Resolved environment issues by using **Docker** to ensure the `sam build` process uses the required Python version correctly.
* **Achieved a critical milestone:** Successfully deployed the project to the AWS environment, overcoming local debugging hurdles.
* The **Travel-Guided project** now has a working API version on a real Cloud environment (though deeper testing is still required).
* ...