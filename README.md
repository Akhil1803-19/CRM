✅ Prerequisites
•	Power Platform CLI (PAC) installed: 
Shell
npm install -g pac
Show more lines
•	Solution Packager (part of Power Platform tools).
•	GitHub Actions enabled for your repository.
•	Azure Service Principal created and registered in Azure AD.
•	API Permissions granted: 
o	Dynamics CRM → user_impersonation.
•	Admin Consent applied for the app registration.
•	GitHub Secrets configured: 
o	POWERPLATFORM_CLIENT_ID
o	POWERPLATFORM_CLIENT_SECRET
o	POWERPLATFORM_TENANT_ID
o	Environment URLs: CRMSTAGE_ENV_URL, CRMSIT_ENV_URL, CRMUAT_ENV_URL, PROD_ENV_URL.
________________________________________
✅ Access Needed
•	Azure Portal: 
o	Ability to create App Registrations and Service Principals.
•	Power Platform Admin Center: 
o	Environment Admin role for DEV, STAGE, SIT, UAT, PROD.
o	Ability to add Application Users and assign roles.
•	GitHub Repository: 
o	Admin access to configure Actions and Secrets.
•	CommonMaster Environment: 
o	Base solution layer setup.
•	Individual DEV Environments for each squad.
•	CRMSTAGE, SIT, UAT, PROD environments provisioned and accessible.


✅ Step 1: Register an App in Azure AD (Service Principal)
1.	Sign in to Azure Portal: https://portal.azure.com
2.	Navigate to: 
o	Azure Active Directory → App registrations → New registration.
3.	Fill in: 
o	Name: PowerPlatform-CICD
o	Supported account types: Select Single tenant (recommended).
o	Redirect URI: Leave blank (not needed for service principal).
4.	Click Register.
5.	After registration, note: 
o	Application (client) ID → This will be your POWERPLATFORM_CLIENT_ID.
o	Directory (tenant) ID → This will be your POWERPLATFORM_TENANT_ID.
________________________________________
✅ Step 2: Create a Client Secret
1.	In the same app, go to Certificates & secrets.
2.	Click New client secret.
3.	Add: 
o	Description: e.g., GitHub Actions CI/CD.
o	Expiry: Choose 6 months or 12 months.
4.	Click Add.
5.	Copy the Value immediately → This is your POWERPLATFORM_CLIENT_SECRET.
________________________________________
✅ Step 3: Assign API Permissions
1.	Go to API permissions → Add a permission.
2.	Select: 
o	Dynamics CRM → Delegated permissions → user_impersonation.
3.	Click Add permissions.
4.	Click Grant admin consent for your organization.
________________________________________
✅ Step 4: Add Service Principal to Power Platform Environment
1.	Go to Power Platform Admin Center: https://admin.powerplatform.microsoft.com
2.	Select your environment (DEV, STAGE, SIT, UAT, PROD).
3.	Navigate to: 
o	Settings → Users + Permissions → Application Users.
4.	Click New app user.
5.	Select the app you registered (PowerPlatform-CICD).
6.	Assign System Administrator role (or appropriate role for deployments).
7.	Save.
________________________________________
✅ Step 5: Configure GitHub Secrets
In your GitHub repository:
•	Go to Settings → Secrets and variables → Actions → New repository secret.
•	Add: 
o	POWERPLATFORM_CLIENT_ID = Application ID
o	POWERPLATFORM_CLIENT_SECRET = Client Secret
o	POWERPLATFORM_TENANT_ID = Tenant ID
o	CRMSTAGE_ENV_URL, CRMSIT_ENV_URL, CRMUAT_ENV_URL, PROD_ENV_URL = Environment URLs
________________________________________
✅ Step 6: Authenticate in GitHub Actions
Use Power Platform CLI in workflows:
Shell
pac auth create --clientId ${{ secrets.POWERPLATFORM_CLIENT_ID }} \
--clientSecret ${{ secrets.POWERPLATFORM_CLIENT_SECRET }} \
--tenantId ${{ secrets.POWERPLATFORM_TENANT_ID }} \
--url ${{ secrets.CRMSTAGE_ENV_URL }}
``
Show more lines
________________________________________
✅ Step 7: Verify Authentication
Run:
Shell
pac auth list
Show more lines
You should see your environments listed.
________________________________________
🔒 Why This Matters
•	Secure: No username/password stored.
•	Automated: Works with GitHub Actions without manual login.
•	Scalable: Can be reused across multiple environments.



Branching Strategy :
Recommended Branching Strategy: Environment-Aligned GitFlow
1. main (Production Branch)
•	Represents production-ready managed solutions.
•	Only code that passed UAT and is approved for release is merged here.
•	Deploy to ADIBCRM (Production) happens from this branch.
________________________________________
2. develop (Integration Branch)
•	Consolidates all squads’ work before SIT.
•	Daily merges from squad feature branches.
•	Deploy to CRMSTAGE and SIT happens from this branch.
________________________________________
3. Feature Branches
•	Each squad creates branches for their solutions: 
•	feature/squad1-solutionA
•	feature/squad1-solutionB
•	feature/squad2-solutionA
•	Used for active development in CRMDEV environments.
•	Export unmanaged solutions from DEV → commit here.
________________________________________
4. Release Branches
•	Created when preparing for UAT: 
•	release/v1.0
•	Contains managed solutions ready for UAT testing.
•	After UAT approval → merge into main.
________________________________________
5. Hotfix Branches
•	For urgent production fixes: 
•	hotfix/v1.0.1
•	Based on main.
•	After fix → merge back into main and develop.
________________________________________
🔗 Workflow Connection with Branches
•	export-dev.yml → Runs on feature branches (DEV work).
•	import-stage.yml → Runs on develop (integration).
•	build-managed.yml → Runs on release branches (prepare managed solutions).
•	deploy-sit.yml → Triggered from develop.
•	deploy-uat.yml → Triggered from release.
•	deploy-prod.yml → Triggered from main.
•	hotfix.yml → Triggered from hotfix branches.
________________________________________
✅ Visual Flow
feature/* → develop → release/* → main
              ↑          ↑
           hotfix/*   deploy-prod


GITHUB : Repository Structure Recap
/solutions
  /CommonMaster
    /BaseSolution
  /Squad1
    /SolutionA
    /SolutionB
  /Squad2
    /SolutionA
    /SolutionB
  /Squad3
    /SolutionA
    /SolutionB
/.github/workflows
  export-dev.yml
  import-stage.yml
  build-managed.yml
  deploy-sit.yml
  deploy-uat.yml
  deploy-prod.yml
  hotfix.yml
________________________________________
✅ Folder Functionalities
1. /solutions/CommonMaster/BaseSolution
•	Purpose: Holds the foundation solution layer shared across all squads.
•	Functionality: 
o	Acts as a dependency for all squad solutions.
o	Ensures consistency in entities, forms, and components.
________________________________________
2. /solutions/SquadX/SolutionA and /SolutionB
•	Purpose: Each squad has two solutions for modular development.
•	Functionality: 
o	Stores exported solution files (unmanaged from DEV).
o	Used by workflows for: 
	Export (from DEV)
	Build Managed (convert to managed)
	Deploy (to SIT, UAT, PROD)
________________________________________
3. .github/workflows
•	Purpose: Contains all automation scripts for CI/CD.
•	Functionality: 
o	export-dev.yml → Pulls solutions from DEV environments into /solutions.
o	import-stage.yml → Imports all squad solutions into CRMSTAGE for integration.
o	build-managed.yml → Converts solutions in /solutions to managed format.
o	deploy-sit.yml / deploy-uat.yml / deploy-prod.yml → Deploys managed solutions to respective environments.
o	hotfix.yml → Handles emergency fixes via CRMDM.
________________________________________
🔗 How They Work Together
•	Solutions folders = Source of truth for all solution files.
•	Workflow files = Automation logic that uses these folders as input/output.
•	Example: 
o	export-dev.yml exports solutions → saves in /solutions/Squad1/SolutionA.
o	build-managed.yml reads from /solutions/Squad1/SolutionA → creates managed package.
o	deploy-prod.yml uses managed package from /solutions → deploys to PROD.


🔄 CI/CD Workflow Overview
Each YAML file represents a stage in the lifecycle of your CRM solutions. They are connected sequentially and sometimes triggered manually or automatically.
________________________________________
🧩 1. export-dev.yml
Purpose: Export unmanaged solutions from DEV environments (Squad1, Squad2, Squad3).
•	Trigger: On code push or manually.
•	Output: Unmanaged .zip files stored in the repo.
•	Connection: Feeds into import-stage.yml.
________________________________________
🧩 2. import-stage.yml
Purpose: Import all squad solutions into CRMSTAGE for daily integration.
•	Trigger: Scheduled daily or manual.
•	Input: Unmanaged solutions from export-dev.yml.
•	Output: Consolidated staging environment.
•	Connection: Ensures integration before managed build.
________________________________________
🧩 3. build-managed.yml
Purpose: Convert solutions to managed format for deployment.
•	Trigger: On merge to main or release branch.
•	Input: Unmanaged solutions from DEV or STAGE.
•	Output: Managed .zip files.
•	Connection: Prepares packages for deployment to SIT/UAT/PROD.
________________________________________
🧩 4. deploy-sit.yml
Purpose: Deploy managed solutions to CRMSIT01 for system testing.
•	Trigger: Manual or after build.
•	Input: Managed solutions from build-managed.yml.
•	Output: SIT environment updated.
•	Connection: Validates before UAT.
________________________________________
🧩 5. deploy-uat.yml
Purpose: Deploy to CRMUAT01 for business validation.
•	Trigger: Manual after SIT approval.
•	Input: Managed solutions from SIT.
•	Output: UAT environment updated.
•	Connection: Final check before production.
________________________________________
🧩 6. deploy-prod.yml
Purpose: Deploy to ADIBCRM (Production).
•	Trigger: Manual with release approval.
•	Input: UAT-approved managed solutions.
•	Output: Live production update.
•	Connection: Final stage of release pipeline.
________________________________________
🧩 7. hotfix.yml
Purpose: Handle emergency fixes via CRMDM (HotFix) environment.
•	Trigger: Manual.
•	Flow: 
1.	Fix in CRMDM (Unmanaged)
2.	Export as Managed
3.	Deploy to Pre-Prod → PROD
________________________________________
🔗 How They’re Connected
Plain Text
export-dev.yml → import-stage.yml → build-managed.yml → deploy-sit.yml → deploy-uat.yml → deploy-prod.yml
↑
hotfix.yml (parallel emergency path)
Each file is a step in the release pipeline, and they work together to:
•	Export → Integrate → Package → Test → Validate → Release


