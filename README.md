# Azure Lab Governance Sample  

**The code contained in this repository is a sample. It is for demonstration purposes only.** 

This repository contains basic code and guidance for lightweight governance in Azure associated with an Innovation Lab.

### Management Group

A Management Group should be created to help enforce governance across the Subscriptions used in the Lab. The following Policies should be implemented at the Mgmt Group level. This will prevent anyone given access to the Subscritions to have the ability to change the Policies.


### Azure Policies

1. IP restriction for App Service

    These 3 JSON files include Azure Policies that work in conjunction with each other to lock down the networking for App Serivce. **NOTE: that this applies to all App Services kinds: WebApp, FunctionApp, APIApp. And it only applies to the front-end of the service. It does not restrict the back-end / source control side of the service. These samples can easily be expanded to include the SCM side.** 

    * RestrictAppServiceInboundIPAddress.json
    This custom Policy will Append an Allow rule for an inbound IP block for all new App Services., when this is in place App Service will automatically include a Deny all at the lowest priority. 

    * AuditAppServiceInboundIPRestriction.json
    This custom Policy will Audit whether the above defined IP rule is in place. 

    * AuditAppServiceInboundNotAllowed.json
    This custom Policy will Audit whether there are any Allow rules other than your defined IP block.  



### Policy Compliance and Clean-up

With these Policies in place, you can retrieve and work with the non-compliant resources with the Azure CLI. 

For instance, if you are implementing this governance on an existing Subscriptions, you may have a number of non-compliance App Services. Here is a simple way to bring them to compliance: 

````
    # You will have to add the name of your Mgmt Group and then run this twice, once for each Audit Policy above. You need to get the name of your policy which is a GUID and is easiest to get from the Azure Portal. 

    ids=$(az policy state list -m "<My Mgmt Group>" --resource-type "Microsoft.Web/sites/config" --filter "(isCompliant eq false and policyDefinitionName eq '<the name of your policy>')" --select "resourceId" --query "[].resourceId" -o tsv)

    # Just substitute your IP address block
    for id in $ids
    do
    az webapp config access-restriction add --priority 100 --action "Allow" --ip-address "<your ip address block>" --rule-name "AllowedLabIPBlock" --ids $id
    done

````

