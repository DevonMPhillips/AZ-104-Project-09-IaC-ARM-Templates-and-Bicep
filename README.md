# AZ-104-Project-09-IaC-ARM-Templates-and-Bicep
DMP Consulting wants to standardize infrastructure deployments across Development, Test, and Production environments.  Instead of manually deploying resources, the company wants Infrastructure as Code (IaC) templates that can be reused across multiple environments.


---


**Project Goal**

Implement Infrastructure as Code (IaC) to standardize Azure deployments for DMP Consulting. Rather than manually creating resources through the Azure portal, you'll deploy and manage infrastructure using ARM Templates and Bicep.

This project introduces repeatable, version-controlled deployments that can be reused across Development, Test, and Production environments.


---


**Scenario**


DMP Consulting has been deploying Azure resources manually through the Azure portal.

As the company grows, this process has become inconsistent and difficult to maintain. Different administrators create resources with different naming conventions, configurations, and settings, increasing the risk of configuration drift.

Leadership has decided to adopt Infrastructure as Code (IaC) so that Azure environments can be deployed consistently, quickly, and repeatedly.

As the Azure Administrator, your task is to build a reusable deployment template that provisions the core infrastructure required for future Azure projects.


---


Architecture

<img width="2487" height="1259" alt="Blank diagram" src="https://github.com/user-attachments/assets/4169a446-b7b0-434b-bb93-77a4f9485d0f" />


---


**Create Resource Group**

az group create --name DMP-IaC-RG  --location centralus --tags owner=devon department=IT


<img width="957" height="657" alt="image" src="https://github.com/user-attachments/assets/d14247ec-9bb1-4bc2-b157-36eb1debaf29" />


---


**Create Virtual Network**

Configurations

| Setting        | Value           |
| -------------- | --------------- |
| Name           | DMP-VNet        |
| Address Space  | 10.0.0.0/16     |
| Subnet         | Frontend-Subnet |
| Subnet Address | 10.0.1.0/24     |


az network vnet create --resource-group DMP-IaC-RG --name DMP-VNet --address-prefixes 10.0.0.0/16 --subnet-name Frontend-Subnet --subnet-prefixes 10.0.1.0/24 --tags owner=devon department=IT


<img width="952" height="657" alt="image" src="https://github.com/user-attachments/assets/253eb4ff-f3e9-43a5-9c4e-b9464213da40" />


---


**Create Network Security Group**

az network nsg create --resource-group DMP-IaC-RG --location centralus --name DMP-Web-NSG --tags owner=devon department=IT


<img width="955" height="660" alt="image" src="https://github.com/user-attachments/assets/aab3456c-97eb-463e-9f80-3a3ac04d99ad" />


---


**Create Storage Account**

az storage account create --name dmpiacstorage --resource-group DMP-IaC-RG --location centralus --sku Standard_LRS --kind StorageV2 --tags owner=devon department=IT


<img width="955" height="612" alt="image" src="https://github.com/user-attachments/assets/2becc390-ab5e-47e8-99e6-ff7b282fed29" />


---


**Export the Deployment**

Convert the manually created infrastructure into Infrastructure as Code. Azure will generate an ARM template representing our deployed resources.

<img width="952" height="717" alt="image" src="https://github.com/user-attachments/assets/1e24ab26-32d7-46dd-8476-651a39f590d9" />

[template.json](https://github.com/user-attachments/files/29678147/template.json)

{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworks_DMP_VNet_name": {
            "defaultValue": "DMP-VNet",
            "type": "String"
        },
        "storageAccounts_dmpiacstorage_name": {
            "defaultValue": "dmpiacstorage",
            "type": "String"
        },
        "networkSecurityGroups_DMP_Web_NSG_name": {
            "defaultValue": "DMP-Web-NSG",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2025-05-01",
            "name": "[parameters('networkSecurityGroups_DMP_Web_NSG_name')]",
            "location": "centralus",
            "tags": {
                "owner": "devon",
                "department": "IT"
            },
            "properties": {
                "securityRules": []
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2025-05-01",
            "name": "[parameters('virtualNetworks_DMP_VNet_name')]",
            "location": "centralus",
            "tags": {
                "owner": "devon",
                "department": "IT"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "10.0.0.0/16"
                    ]
                },
                "privateEndpointVNetPolicies": "Disabled",
                "subnets": [
                    {
                        "name": "Frontend-Subnet",
                        "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_DMP_VNet_name'), 'Frontend-Subnet')]",
                        "properties": {
                            "addressPrefix": "10.0.1.0/24",
                            "delegations": [],
                            "privateEndpointNetworkPolicies": "Disabled",
                            "privateLinkServiceNetworkPolicies": "Enabled",
                            "defaultOutboundAccess": false
                        }
                    }
                ],
                "virtualNetworkPeerings": [],
                "enableDdosProtection": false
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2026-04-01",
            "name": "[parameters('storageAccounts_dmpiacstorage_name')]",
            "location": "centralus",
            "tags": {
                "owner": "devon",
                "department": "IT"
            },
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "StorageV2",
            "properties": {
                "allowCrossTenantReplication": false,
                "minimumTlsVersion": "TLS1_0",
                "allowBlobPublicAccess": false,
                "networkAcls": {
                    "ipv6Rules": [],
                    "bypass": "None",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": true,
                "encryption": {
                    "services": {
                        "file": {
                            "keyType": "Account",
                            "enabled": true
                        },
                        "blob": {
                            "keyType": "Account",
                            "enabled": true
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "accessTier": "Hot"
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2025-05-01",
            "name": "[concat(parameters('virtualNetworks_DMP_VNet_name'), '/Frontend-Subnet')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_DMP_VNet_name'))]"
            ],
            "properties": {
                "addressPrefix": "10.0.1.0/24",
                "delegations": [],
                "privateEndpointNetworkPolicies": "Disabled",
                "privateLinkServiceNetworkPolicies": "Enabled",
                "defaultOutboundAccess": false
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/blobServices",
            "apiVersion": "2026-04-01",
            "name": "[concat(parameters('storageAccounts_dmpiacstorage_name'), '/default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccounts_dmpiacstorage_name'))]"
            ],
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "properties": {
                "staticWebsite": {
                    "enabled": false
                },
                "cors": {
                    "corsRules": []
                },
                "deleteRetentionPolicy": {
                    "allowPermanentDelete": false,
                    "enabled": false
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/fileServices",
            "apiVersion": "2026-04-01",
            "name": "[concat(parameters('storageAccounts_dmpiacstorage_name'), '/default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccounts_dmpiacstorage_name'))]"
            ],
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "properties": {
                "protocolSettings": {
                    "smb": {}
                },
                "cors": {
                    "corsRules": []
                },
                "shareDeleteRetentionPolicy": {
                    "enabled": true,
                    "days": 7
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/queueServices",
            "apiVersion": "2026-04-01",
            "name": "[concat(parameters('storageAccounts_dmpiacstorage_name'), '/default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccounts_dmpiacstorage_name'))]"
            ],
            "properties": {
                "cors": {
                    "corsRules": []
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/tableServices",
            "apiVersion": "2026-04-01",
            "name": "[concat(parameters('storageAccounts_dmpiacstorage_name'), '/default')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccounts_dmpiacstorage_name'))]"
            ],
            "properties": {
                "cors": {
                    "corsRules": []
                }
            }
        }
    ]
}

Now if we wanted to make the deployment reusable we could modify:

Storage account name → parameter
Resource group location → parameter
VNet name → parameter

Instead of hardcoding values, allow them to be passed in during deployment.


---


**Convert ARM to Bicep**

az bicep decompile --file template.json


param virtualNetworks_DMP_VNet_name string = 'DMP-VNet'
param storageAccounts_dmpiacstorage_name string = 'dmpiacstorage'
param networkSecurityGroups_DMP_Web_NSG_name string = 'DMP-Web-NSG'

resource networkSecurityGroups_DMP_Web_NSG_name_resource 'Microsoft.Network/networkSecurityGroups@2025-05-01' = {
  name: networkSecurityGroups_DMP_Web_NSG_name
  location: 'centralus'
  tags: {
    owner: 'devon'
    department: 'IT'
  }
  properties: {
    securityRules: []
  }
}

resource virtualNetworks_DMP_VNet_name_resource 'Microsoft.Network/virtualNetworks@2025-05-01' = {
  name: virtualNetworks_DMP_VNet_name
  location: 'centralus'
  tags: {
    owner: 'devon'
    department: 'IT'
  }
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    privateEndpointVNetPolicies: 'Disabled'
    subnets: [
      {
        name: 'Frontend-Subnet'
        id: virtualNetworks_DMP_VNet_name_Frontend_Subnet.id
        properties: {
          addressPrefix: '10.0.1.0/24'
          delegations: []
          privateEndpointNetworkPolicies: 'Disabled'
          privateLinkServiceNetworkPolicies: 'Enabled'
          defaultOutboundAccess: false
        }
      }
    ]
    virtualNetworkPeerings: []
    enableDdosProtection: false
  }
}

resource storageAccounts_dmpiacstorage_name_resource 'Microsoft.Storage/storageAccounts@2026-04-01' = {
  name: storageAccounts_dmpiacstorage_name
  location: 'centralus'
  tags: {
    owner: 'devon'
    department: 'IT'
  }
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
  kind: 'StorageV2'
  properties: {
    allowCrossTenantReplication: false
    minimumTlsVersion: 'TLS1_0'
    allowBlobPublicAccess: false
    networkAcls: {
      ipv6Rules: []
      bypass: 'None'
      virtualNetworkRules: []
      ipRules: []
      defaultAction: 'Allow'
    }
    supportsHttpsTrafficOnly: true
    encryption: {
      services: {
        file: {
          keyType: 'Account'
          enabled: true
        }
        blob: {
          keyType: 'Account'
          enabled: true
        }
      }
      keySource: 'Microsoft.Storage'
    }
    accessTier: 'Hot'
  }
}

resource virtualNetworks_DMP_VNet_name_Frontend_Subnet 'Microsoft.Network/virtualNetworks/subnets@2025-05-01' = {
  name: '${virtualNetworks_DMP_VNet_name}/Frontend-Subnet'
  properties: {
    addressPrefix: '10.0.1.0/24'
    delegations: []
    privateEndpointNetworkPolicies: 'Disabled'
    privateLinkServiceNetworkPolicies: 'Enabled'
    defaultOutboundAccess: false
  }
  dependsOn: [
    virtualNetworks_DMP_VNet_name_resource
  ]
}

resource storageAccounts_dmpiacstorage_name_default 'Microsoft.Storage/storageAccounts/blobServices@2026-04-01' = {
  parent: storageAccounts_dmpiacstorage_name_resource
  name: 'default'
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
  properties: {
    staticWebsite: {
      enabled: false
    }
    cors: {
      corsRules: []
    }
    deleteRetentionPolicy: {
      allowPermanentDelete: false
      enabled: false
    }
  }
}

resource Microsoft_Storage_storageAccounts_fileServices_storageAccounts_dmpiacstorage_name_default 'Microsoft.Storage/storageAccounts/fileServices@2026-04-01' = {
  parent: storageAccounts_dmpiacstorage_name_resource
  name: 'default'
  sku: {
    name: 'Standard_LRS'
    tier: 'Standard'
  }
  properties: {
    protocolSettings: {
      smb: {}
    }
    cors: {
      corsRules: []
    }
    shareDeleteRetentionPolicy: {
      enabled: true
      days: 7
    }
  }
}

resource Microsoft_Storage_storageAccounts_queueServices_storageAccounts_dmpiacstorage_name_default 'Microsoft.Storage/storageAccounts/queueServices@2026-04-01' = {
  parent: storageAccounts_dmpiacstorage_name_resource
  name: 'default'
  properties: {
    cors: {
      corsRules: []
    }
  }
}

resource Microsoft_Storage_storageAccounts_tableServices_storageAccounts_dmpiacstorage_name_default 'Microsoft.Storage/storageAccounts/tableServices@2026-04-01' = {
  parent: storageAccounts_dmpiacstorage_name_resource
  name: 'default'
  properties: {
    cors: {
      corsRules: []
    }
  }
}


---


**Deploy Using Azure CLI**


Deploy using ARM:

az deployment group create --resource-group DMP-IaC-Test --template-file template.json --parameters parameters.json


Deploy using Biceps:

az deployment group create --resource-group DMP-IaC-Dev --template-file main.bicep

----

**Tasks Completed**

✔ Deploy infrastructure manually and through Infrastructure as Code
✔ Export existing Azure resources as an ARM template
✔ Interpret and modify ARM templates
✔ Convert ARM templates to Bicep
✔ Modify and deploy Bicep files
✔ Deploy Azure resources using the Azure CLI
