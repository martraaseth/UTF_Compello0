### Documentation is included in the Documentation folder ###


#### Sopra Steria ReFrameWork Template ####
**Robotic Enterprise Framework**

* extended template from UiPath designed with Sopra Steria's own naming conventions and workflows
* built on top of *Transactional Business Process* template
* using *State Machine* layout for the phases of automation project
* offering high level exception handling and application recovery
* keeps external settings in *Config.xlsx* file and Orchestrator assets
* pulls credentials from *Credential Manager* and Orchestrator assets
* uploads data to queues in Orchestrator with new state and workflow
* gets transaction data from Orchestrator queue and updates back status
* takes screenshots in case of application exceptions
* provides extra utility workflows like sending a templated email
* runs sample Notepad application with dummy Excel input data from Orchestrator
* retry login if it fails is now supported
* kill process workflow is redefined and made generic
* send exception email with screenshot attached added to the template


### How It Works ###

1. **INITIALIZE PROCESS**
 + *InitiAllSettings* - Load config data from file and from assets
 + *InitiAllApplications* - Login to applications as per Config("OpenApps") field
   + *GetAppCredentials* - From Orchestrator assets or local Credential Manager
 + If failing with a system error the process will transition to end process.
 + If failing in a login workflow, the state will be retried a number of times defined. Then transition to end process state.

2. **UPLOAD TRANSACTION DATA**
   + ./Framework/*UploadTransactionData* - Uploads data to Orchestrator queue from. This can be spreadsheet, API, database etc.
   + This state checks if the process has uploaded transaction data before. Transaction data is only to be uploaded once during the process, even if system error occurs and the process is restarted. 
   + The flag is set inside *UploadTransactionData*. 
 
3. **GET TRANSACTION DATA**
   + ./Framework/*GetTransactionData* - Fetches from Orchestrator queue as per Config("TransactionQueue")
   + ./*GetTransactionData* - Sample gets data from Orchestrator queue.

4. **PROCESS TRANSACTION**
 + Try *ProcessTransaction*
 + If application exceptions happen
   + *SaveErrorScreen* - In Config("ErrorsFolder") with the exception message
   + If Config("ExMailAlert") flag is set to true the *SendExceptionEmail* will be invoked. This will send mail with exception and screenshot.
   + Going to re/INITIALIZE
 + *SetTransactionStatus* - As Success, Failed or Rejected with reason
   + ./Framework/*SetTransactionStatus* - Updates the Orchestrator queue item
   + ./*SetTransactionStatus* - If Orchestrator queue has 0 < # Max of retries the queue item will be retried if system error occurs. 

5. **END PROCESS**
 + *CloseAllApplications*
 + If *CloseAllApplications* fails the *KillAllProcessses* will be invoked. The process will kill application defined in Config("ApplicationsToKill")


### For New Project ###

1. Check out the Config.xlsx file and add/customize any required fields and values
2. Implement OpenApp and CloseApp workflows, linking them in the Config.xlsx fields
3. Implement GetTransactionData and SetTransactionStatus or use ./Framework versions for Orchestrator queues
4. Implement ProcessTransaction workflow and any invoked others
