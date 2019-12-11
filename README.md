# DevOps-for-Data-Scientist-in-Manufacturing-Case-Study-of-R
DevOps for Data Scientist in Manufacturing: Case Study of R

### Jack Xue, Principal Cloud Solution Architect
### Marc Wolfson, Senior Specialist of DevOps
### KC Munnings, Senior Cloud Solution Architect
### David Crook, Senior Software Development Engineer

In the last ten months, we have been working with a Fortune 500 Manufacturing company to modernize their Data Scientist team’s workflow under the company’s toolset selection and security requirements. 

For the code repository, the company’s selection is an on-premises GitLab. For the development environment, Azure Databricks is selected. They also require that all build processes must be carried out on machines in their company’s VNET. Role-backed authentication, RBAC, needs to be applied everywhere. Moreover, the keys for service authentication must be stored in Azure Key Vault.

There are two challenges we faced in the journey. In this presentation, we will emphasize how we overcome these challenges with reusable code examples.

The first challenge is that since the company uses on-premises GitLab and thus the code change checked into the repository could not directly trigged the CI pipeline in Azure DevOps. Luckily, the on-premises GitLab can host a webhook which sends out JSON messages to Azure Logic App which parses the JSON message and identifies if an Azure DevOps action should be carried out. If so the JSON message is forwarded to the Azure DevOps which will start the CI process as configured. 

The second challenge is that for R coding, CRAN package testthat is commonly used for unit tests and regression tests implemented as test cases and test suite. However, the Databricks workspace file system presents a unique challenge: the method test_file() of this package in the script /Shared/r_model/test_suite.r, fails to find test case files in its subdirectory, such as /Shared/r_model/tests/test_cases.r. However, if the test case file is in DBFS then the method can locate it. Based on this observation, we code test_cases.r in workspace, and export it to the build agent VM where Databricks CLI was also installed, and then copy this file back to DBFS by using the CLI. With this workaround, we are able to run /Shared/r_model/test_suite.r that generates the desired JunitReporter in XML for Azure DevOps to display. 

Once these two challenges been overcome, we can leverage the CI/CD processes provided by Azure DevOps to build the needed pipelines and gatekeepers for the whole workflow automation.

Enforcing one of the principles of DevOps that all actions carried out in pipelines should be environment agonistic, we aim to craft the automation processes within Azure Databricks that provide the transparency and governance required to ensure high quality with minimal overhead. Here is our approach: in Azure DevOps we simply add the components required to the agents running the automation in pipelines. To secure the pipelines we stored all keys, including the Databricks workspace token, in Azure Key Vault. The pipeline dynamically pulls the information from Azure Key Vault at run time. This provides two tangible benefits:
1.	No keys are exposed or hard coded into the pipeline; and
2.	The keys can be rotated by security teams without impacting any existing pipelines that leverage these keys.

The overall workflow becomes the following:
1.	A data scientist creates a topic branch from dev branch in GitLab
2.	She imports the notebooks, update them and run them with tests in Databricks.
3.	Once satisfied, she exports the notebooks and checks them into GitLab and creates a merge request.
4.	Merge Request automatically kicks off a verification build and test in a verification workspace and notifies the approver.
5.	If satisfied, the approver approves the merge.

Moreover, once ready to move from dev to staging environment, one with correct GitLab permissions can create a merge request from dev branch which will again trigger a verification build and test in Databricks and the positive result will trigger Azure DevOps to migrate the notebooks from dev to stage environments.

This process can be repeated to move from stage to production environments.

## Acknowledgement:
Brent Samodien, Prashant Karbhari, Andrej Kyselica and Bunty Ranu also contributed to this project in different stages.

## Reference:

Build & Deliver AI Content to Global Partners
https://microsoft.sharepoint.com/:p:/t/WWCMachineLearning/EY2h4bILQpxFr5C01l5mxlQB2XM1b8MhgL2BftCOMNeH0g?e=9BK823

Unit tests and regression tests of R model in Databricks with library TestThat
https://github.com/JackXueIndiana/unit-test-r-model-in-databricks-with-testthat


R Model Container Sample
https://github.com/kcm117/rmodel-container-sample


