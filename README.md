# oadp-capstone

# Step 1 
Add OADP Operator to OLM

Clone the OADP Operator repository:
```
git clone git@github.com:konveyor/oadp-operator.git
```

Then follow the instructions [here](https://github.com/devarshshah15/oadp-operator/tree/olm-integrate#olm-integration).

# Step 2
Install OADP Operator from OperatorHub



# Step 3
Install OCS from OperatorHub

Navigate to the OpenShift console. Under the Administrator view, go to Operators on the left tab and click on OperatorHub. Search for the OpenShift Container Storage operator in the search bar. Click on it to install and subscribe to the operator. 

![OCS OperatorHub](/images/ocs_operatorhub.png)

If you go to Installed Operators and select the openshift-storage project, you should see the OpenShift Container Storage operator successfully installed:

![OCS Installed](/images/ocs_installed.png)
