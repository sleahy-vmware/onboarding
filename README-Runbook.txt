Onboarding Run book

Phase 1 - capture machine data from vRA 7
  1. Copy the vRA8OnboardingUtility-P3-01May2022.zip to a folder on your LCM appliance. LCM will need network access to vRA 7 portal and web manager on port 443.
     Note: FIPS Mode needs to be disabled on the LCM VM to allow the script to run.
          In LCM From My Service dashboard -> Lifecycle Operations -> Settings page. Under System Administration, click System Details. Disable the FIPS Mode Compliance check box. Click Update.
  2. Extract the contents of the zip with the following command
       unzip vRA8OnboardingUtility-P3-01May2022.zip
  3. Modify the onboarding.ini file with your environment details. To capture 7.x data you are only required to populate the general and vra7source section of onboarding.ini file. There is a sample onboarding.ini file included to demonstrate how the file should be populated. 
     NOTE: Pound Sign ( £ ) in the IAAS password is not supported, this is due to a limitation in curl.
  4. Run the captureMachineData.py script to capture data from vRA 7 with command
        python3 captureMachineData.py
        Note: This script can also be run on any system that has python 3 installed, curl and network access to all vRA 7 IAAS Web Manager and vRA 7 appliance.

     Alternatively you can capture machine data directly from discovered machines in vRA 8/Cloud with command
        python3 captureDiscoveredMachineData.py
    
  5. Copy the newly created machine_data.csv and machine_data_custom_properties.csv that are generated to a system that you can modify them easily.
     Note: The DB file generated is required for validation and onboarding process
          Filenames may differ slightly depending on values populated in the onboarding.ini file.

Phase 2 - Review and update machine data, and custom property export csv
  1. Open the machine_data.csv in a suitable editor.
    The file will need to be updated with the following changes, all changes are optional.
      - The import column can be modified to omit a machine from the onboarding process, set the value to no and the machine will be skipped during onboarding
      - The newOwner column can be updated with the desired owner of the deployment/machine in vRA 8. This column can be left blank and the existing owner in vRA 7 will be assumed as the owner in vRA 8.
      - The project column can be updated with the desired target project for the deployment/machine in vRA 8. This column can be left blank and the Business Group name in vRA 7 will be assumed as the project name in vRA 8.
      - The endpointFQDN column can be updated with the FQDN that is configured on the corresponding cloud account in vRA 8. This is optional and is only required for a particular set of use cases where the fqdn in 7 does not match the fqdn in vRA 8.
      - The cloudTemplateName column can be updated with a specific cloud template to be associated with each deployment. If this column is left blank the defaults specified in the onboarding.ini file will be used. If you do not want to associate cloud templates this can also be specified in the onboarding.ini file 

      Once all changes have been completed you can save and close the file.

  2. Open the machine_data_custom_properties.csv in a suitable editor.
    Note: if you do not want to include any custom properties with the onboarded machine you can delete the machine_data_custom_properties.csv and set importprop = False in the properties section of the onboarding.ini file.
    The file can be updated as follows.
      - Remove any properties that you do not want to be included in the onboarding process
      - Populate the newPropertyName column if you want a property in 7 to have a new name in vRA 8 once onboarding is completed. The associated value will be the same as per vRA 7

    Any properties that remain in this file will be set as custom properties during the onboarding process.

    Once you have made all required changes you can save and close the file.

Phase 3 - Validate Dataset
  1. Copy the updated csv files back into the folder on LCM VM.
       machine_data.csv and machine_data_custom_properties.csv
  2. Modify the onboarding.ini file with your environment details. All section are required with either vraonprem or vracloud depending on your target vRA instance
  3. Run the validateOnboardingCSV.py script with the following command
        python3 validateOnboardingCSV.py
  4. Review the output and address and validation errors. There is a guide included that can provide some trobleshooting tips Troubleshooting_validation_error_for_bulk_onboarding.txt

Phase 4 - Update delta
  Once you have populated your csv file with projects and owners in some cases this process can take weeks and in a busy system this can lead to multiple deployments that are not included in the onboarding process or have been deleted and can be removed from the onboarding process. To help support this use case you can run the data capture again with the -d flag and we support bulk delete of machines from the dataset to help with deleted deployments.
  1. Capturing new content in 7.x can be achieved by running the captureMachineData.py script with the delta flag. The following command can be run.
      python3 captureMachineData.py -d
      NOTE: IF RUN WITHOUT THE -d FLAG. THE DATASET WILL BE DELETED AND YOU WILL NEED TO START THE ENTIRE PROCESS AGAIN FROM THE BEGINNING.

  2. If you ran the captureMachineData.py with the delta flag you will need to update the csv file with any required changes for the newly added deployments inline with Phase 2. Make sure to copy the updated file back to the LCM VM once all changes have been saved.

  3. Delete machines in the dataset relating to deleted deployments in vRA 7.x. When you run the validateOnboardingCSV.py it will print out a list of Unidentified VMs. This list can be copied into the deletedvms.ini file and will be deleted from the dataset the next time you run validateOnboardingCSV.py. You will also need to make sure the onboarding.ini file has the 'deletedvms' parameter set. This should be set to deletedvms = deletedvms.ini

  4. If you ran captureMachineData.py with the delta flag and/or configured the deletedvms.ini with deleted deployments you will need to run the validateOnboardingCSV.py script again as per Phase 3.


Phase 5 - Onboard Machines
  1. Run the onboardMachines.py script with the following command. This script can be run in a dry-run mode where the onboarding plans are created and not executed but this is not a required step. The dry-run mode is to allow a review of what deployments will be created in what project. DO NOT RUN ANY ONBOARDING PLANS CREATED BY THE SCRIPT FROM THE UI. DEPLOYMENT OWNERSHIP WILL BE INCORRECT. Plans should be executed only by the script
      python3 onboardMachines.py

Phase 6 - Clean Up
  1. Run the postOnboardingCleanUp.py script to fix project member temporary changes. The script can be run with the following command. 
      python3 postOnboardingCleanUp.py
  Note: This step not required for vRA 8.8 GA or later.

