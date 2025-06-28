
## Projeto de gerenciamento de Maquinas Virtuais no Azure

<a id="sumario"></a>

- [Escopo do projeto](#topico_01)
    - [Azure Resource Manager](#sub-topico-01)
    - [Azure Resource Group](#sub-topico-02)
    - [Deploy vnet](#sub-topico-03)
    - [Deploy virtual machine](#sub-topico-04)

## Escopo do projeto<a id="topico_01"></a>

![Escopo do Projeto](https://raw.githubusercontent.com/tarsiogomes/az-104/main/images/escopo-projeto.png "Diagrama de Escopo do Projeto")

### Azure Resource Manager: <a id="sub-topico-01"></a>

> Link para adicionar [Azure Resource Manager](https://portal.azure.com/#view/Microsoft_Azure_Resources/ResourceManagerBlade/~/overview)

### Azure Resource Group<a id="sub-topico-02"></a>

> Link para adicionar [Resource Group](https://portal.azure.com/#browse/resourcegroups)

### Deploy vnet<a id="sub-topico-03"></a>

> Virtual Network criada e adiciona as subnets:

![VNET](https://github.com/tarsiogomes/az-104/blob/main/images/vnet.png)

### Deploy virtual machine<a id="sub-topico-04"></a>

![Virtual Machines](https://github.com/tarsiogomes/az-104/blob/main/images/virtual-machines.png)

## ARM - Azure Resource Manager

> Servidor Windows Server:
~~~
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string"
        },
        "networkInterfaceName": {
            "type": "string"
        },
        "networkSecurityGroupName": {
            "type": "string"
        },
        "networkSecurityGroupRules": {
            "type": "array"
        },
        "subnetName": {
            "type": "string"
        },
        "virtualNetworkId": {
            "type": "string"
        },
        "publicIpAddressName": {
            "type": "string"
        },
        "publicIpAddressType": {
            "type": "string"
        },
        "publicIpAddressSku": {
            "type": "string"
        },
        "pipDeleteOption": {
            "type": "string"
        },
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineComputerName": {
            "type": "string"
        },
        "virtualMachineRG": {
            "type": "string"
        },
        "osDiskType": {
            "type": "string"
        },
        "osDiskDeleteOption": {
            "type": "string"
        },
        "virtualMachineSize": {
            "type": "string"
        },
        "nicDeleteOption": {
            "type": "string"
        },
        "hibernationEnabled": {
            "type": "bool"
        },
        "adminUsername": {
            "type": "string"
        },
        "adminPassword": {
            "type": "secureString"
        },
        "patchMode": {
            "type": "string"
        },
        "enablePeriodicAssessment": {
            "type": "string"
        },
        "enableHotpatching": {
            "type": "bool"
        }
    },
    "variables": {
        "nsgId": "[resourceId(resourceGroup().name, 'Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroupName'))]",
        "vnetId": "[parameters('virtualNetworkId')]",
        "vnetName": "[last(split(variables('vnetId'), '/'))]",
        "subnetRef": "[concat(variables('vnetId'), '/subnets/', parameters('subnetName'))]"
    },
    "resources": [
        {
            "name": "[parameters('networkInterfaceName')]",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2022-11-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkSecurityGroups/', parameters('networkSecurityGroupName'))]",
                "[concat('Microsoft.Network/publicIpAddresses/', parameters('publicIpAddressName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIpAddress": {
                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', parameters('publicIpAddressName'))]",
                                "properties": {
                                    "deleteOption": "[parameters('pipDeleteOption')]"
                                }
                            }
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[variables('nsgId')]"
                }
            }
        },
        {
            "name": "[parameters('networkSecurityGroupName')]",
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-05-01",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": "[parameters('networkSecurityGroupRules')]"
            }
        },
        {
            "name": "[parameters('publicIpAddressName')]",
            "type": "Microsoft.Network/publicIpAddresses",
            "apiVersion": "2020-08-01",
            "location": "[parameters('location')]",
            "properties": {
                "publicIpAllocationMethod": "[parameters('publicIpAddressType')]"
            },
            "sku": {
                "name": "[parameters('publicIpAddressSku')]"
            }
        },
        {
            "name": "[parameters('virtualMachineName')]",
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2024-03-01",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('networkInterfaceName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "createOption": "fromImage",
                        "managedDisk": {
                            "storageAccountType": "[parameters('osDiskType')]"
                        },
                        "deleteOption": "[parameters('osDiskDeleteOption')]"
                    },
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-datacenter-gensecond",
                        "version": "latest"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkInterfaceName'))]",
                            "properties": {
                                "deleteOption": "[parameters('nicDeleteOption')]"
                            }
                        }
                    ]
                },
                "securityProfile": {},
                "additionalCapabilities": {
                    "hibernationEnabled": false
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachineComputerName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]",
                    "windowsConfiguration": {
                        "enableAutomaticUpdates": true,
                        "provisionVmAgent": true,
                        "patchSettings": {
                            "patchMode": "[parameters('patchMode')]",
                            "assessmentMode": "[parameters('enablePeriodicAssessment')]",
                            "enableHotpatching": "[parameters('enableHotpatching')]"
                        }
                    }
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true
                    }
                }
            }
        }
    ],
    "outputs": {
        "adminUsername": {
            "type": "string",
            "value": "[parameters('adminUsername')]"
        }
    }
}
~~~

> Servidor Ubuntu Server:

~~~
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachines_sv_linux_01_name": {
            "defaultValue": "sv-linux-01",
            "type": "String"
        },
        "disks_sv_linux_01_OsDisk_1_20bba10540d24f43939b958680f0e41d_externalid": {
            "defaultValue": "/subscriptions/c08a3587-14a7-4a46-9850-a1e804418751/resourceGroups/az-104-project-01/providers/Microsoft.Compute/disks/sv-linux-01_OsDisk_1_20bba10540d24f43939b958680f0e41d",
            "type": "String"
        },
        "networkInterfaces_sv_linux_01500_externalid": {
            "defaultValue": "/subscriptions/c08a3587-14a7-4a46-9850-a1e804418751/resourceGroups/az-104-project-01/providers/Microsoft.Network/networkInterfaces/sv-linux-01500",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2024-11-01",
            "name": "[parameters('virtualMachines_sv_linux_01_name')]",
            "location": "brazilsouth",
            "tags": {
                "centro-custo": "100"
            },
            "properties": {
                "hardwareProfile": {
                    "vmSize": "Standard_D2s_v3"
                },
                "additionalCapabilities": {
                    "hibernationEnabled": false
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "canonical",
                        "offer": "ubuntu-24_04-lts",
                        "sku": "server",
                        "version": "latest"
                    },
                    "osDisk": {
                        "osType": "Linux",
                        "name": "[concat(parameters('virtualMachines_sv_linux_01_name'), '_OsDisk_1_20bba10540d24f43939b958680f0e41d')]",
                        "createOption": "FromImage",
                        "caching": "ReadWrite",
                        "managedDisk": {
                            "storageAccountType": "Standard_LRS",
                            "id": "[parameters('disks_sv_linux_01_OsDisk_1_20bba10540d24f43939b958680f0e41d_externalid')]"
                        },
                        "deleteOption": "Delete",
                        "diskSizeGB": 30
                    },
                    "dataDisks": [],
                    "diskControllerType": "SCSI"
                },
                "osProfile": {
                    "computerName": "[parameters('virtualMachines_sv_linux_01_name')]",
                    "adminUsername": "gomes",
                    "linuxConfiguration": {
                        "disablePasswordAuthentication": true,
                        "ssh": {
                            "publicKeys": [
                                {
                                    "path": "/home/gomes/.ssh/authorized_keys",
                                    "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDObkmKau0MaaSELA/9xYoZljPGKqDt4yaGvhaKLJLOegue0KztYg1ALCd7jEqmruhw2sgNJZR9461RWnkZSZFOHpg7OVS+tPFzrxIPp/H+wMOrQ2EUX2TfQOfD5RtdeGmZS9L2KXszJgLmwTu9Go9fkDk9KuzcL9Wx0aqqQVElj31j5h8bdjq9MaCzlE08mo3jVrt0SRl2WzNlHORZ4QoDYN8VKREpJTneSNs6aHi89e+VKPXYlZ+SwlrYHHGX2Bzr0WV0o97hxV2GLdHe/hJCSHgrJPER05xuE1Xkzn3UVrbhYmA2gBHD9J9FIsRrhzupVYcpZOLghlrk6dRO0KK0PfVNQFaTchIJQUKHnwW9AoBDWD7DBWCwjLUE7YV/5TacilE5zcPTeaKZo/tCk+FEneFQ4NPjUJnLqyC0lVKbycvO8XOaT4FB00rgwjmXxwz26nZZ4JOQaWspj7M6rRe4HwNaoU6UWqmbMfn8FhUhKk2gAIxcc0eQ3cPZgA6cqTU= generated-by-azure"
                                }
                            ]
                        },
                        "provisionVMAgent": true,
                        "patchSettings": {
                            "patchMode": "ImageDefault",
                            "assessmentMode": "ImageDefault"
                        }
                    },
                    "secrets": [],
                    "allowExtensionOperations": true,
                    "requireGuestProvisionSignal": true
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[parameters('networkInterfaces_sv_linux_01500_externalid')]",
                            "properties": {
                                "deleteOption": "Detach"
                            }
                        }
                    ]
                },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true
                    }
                }
            }
        }
    ]
}
~~~

[Voltar ⬆️](#sumario)