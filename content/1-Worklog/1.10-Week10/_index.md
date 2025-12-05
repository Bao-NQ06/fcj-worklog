---
title: "Worklog Week 10"
date: 2024-11-11
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives: 

* **Stabilize the AWS SAM/Serverless deployment environment.**
* **Focus on debugging** core issues: CORS, and loop errors in `template.yaml`.
* **Integrate Frontend/Backend** to enable basic display and interaction testing on the user interface.
* **Complete** the basic **Read** and **Delete** functionalities.
* **Participate in the AWS Cloud Mastery Series event** to receive guidance and address project inquiries.

---

### Tasks to be Deployed This Week:

| Day | Task | Start Date | Completion Date | Resources |
| :--- | :--- | :--- | :--- | :--- |
| Mon | - **Debug CORS:** Analyze CORS configuration in **API Gateway** and the response headers of **Lambda** to allow Frontend access. <br> - **Fix template loop error:** Check and optimize the `template.yaml` file to prevent loop errors during deployment (`sam deploy`). | 11/11/2024 | 11/11/2024 | API Gateway/CORS Documentation |
| Tue | - Continue strengthening the **Read** function (Displaying posts): Ensure data is queried from DynamoDB and returned in the correct JSON format for the Frontend. | 12/11/2024 | 12/11/2024 | |
| Wed | - **Frontend Integration:** Start combining the Frontend code with the project and test the deployed API Endpoints. <br> - **Successfully display** the list of posts on the interface. | 13/11/2024 | 13/11/2024 | Sample Frontend interface |
| Thu | - Deploy and test the **Delete** function (Deleting posts). <br> - **Encountered Error:** Identified an issue with **Authorization** and the **Sub ID** when executing the Delete function. | 14/11/2024 | 14/11/2024 | |
| Fri | - **Participation in AWS Cloud Mastery Series:** <br>&emsp; + Received guidance and clarified questions regarding Serverless, Rekognition. <br> - **Analyze Update/Rekognition error:** Begin applying mentor guidance to resolve authorization issues and Rekognition-related errors. | 15/11/2024 | 15/11/2024 | Mentor, AWS Cloud Mastery Series |

---

### Week 10 Achievements: 

* **Successfully fixed** the CORS error and stabilized the SAM deployment process (mitigated template loop errors).
* **Participated in the AWS Cloud Mastery Series event** and gathered necessary information to solve major project bugs.
* **Completed Frontend and Backend integration**, achieving the first user interface for testing.
* **Successfully deployed the Read** (Displaying posts) and **Delete** (Deleting posts) functionalities, which are operational on the web interface.
* **Identified and gained direction** to solve critical bottlenecks:
    * Authorization error: Lambda fails to retrieve/incorrectly process the **Sub ID** from the Cognito token, affecting privileged operations.
    * **Update** function error: Dependent on the image processing workflow involving **Rekognition** (mentor guidance obtained).
* The project has transitioned to the basic user testing phase.

---