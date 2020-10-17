---
title: "Creating Dynamic Azure ARM Templates"
date: 2020-07-05T00:00:00+01:00
draft: true
image: images/cover.jpg
categories:
    - Azure
---

In this post I'll demonstrate how you can dynamically create resources or set properties for resources in your Azure ARM templates.

For example, you might have a template which accepts an array. For each element in that array, you want to create resources, or set properties for a resource.

The first objective will demonstrate how to create a dynamic number of properties associated with a resource. The second object will show you how to dynamically create a number of resources. The examples will revolve around creating virtual networks and subnets.

## ARM template functions

There's a bunch of [ARM template functions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions) available that you do all kinds of things within your templates: logical operators and conditional actions, string manipulation, array conversions or iteration, arithmetic. All kinds of things.

Using functions within ARM templates make the json files more than just declarative build documents. They enable you to get creative and implement some programmable logic in to your templates. This can help with making your templates being more versatile. 

For this demo I'll focus on the functions `[length()]`, `[concat()]` and `[copyIndex()]`.

## Objective 1

Take the scenario where your current template contains a single virtual network resource and you hardcode the virtual network's address space properties and all the subnets too. 

Maybe you decide to create some logic in your scripts where you dynamically create x many subnets. To do this, we are going to dynamically add x many properties to the `Microsoft.Network/VirtualNetworks` resource.

Let's start with the below example using those hardcoded properties and subnets.

```JSON
...
{
	"name": "vnet01",
	"type": "Microsoft.Network/VirtualNetworks",
	"apiVersion": "2019-09-01",
	"location": "uksouth",
	"properties": {
		"addressSpace": {
			"addressPrefixes": [
				"192.168.0.0/16"
			]
		},
		"subnets": [
			{
				"name": "subnet-1",
				"properties": {
					"addressPrefix": "192.168.1.0/24"
				}
			},
			{
				"name": "subnet-2",
				"properties": {
					"addressPrefix": "192.168.2.0/24"
				}
			},
			{
				"name": "subnet-3",
				"properties": {
					"addressPrefix": "192.168.3.0/24"
				}
			}
		]
	}
}
...
```

Our objective is to dynamically create as many subnets as many there are elements in the parameter value named `subnets`, which is an array of just address prefixes, e.g. `192.168.1.0/24`, `192.168.2.0/24` and `192.168.3.0/24`.

For the `properties` object we're going to heavily modify the `subnets` array and replace it with the `copy` element. This element enables us to create a dynamic number of resources or properties. The `copy` element is effectively what allows us to create a loop within our template. It accepts an array of three objects: `name`, `count` and `input`.

- `name`: the name of our loop. We use this name to reference an iterable.
- `count`:  the number of times we want to iterate over our loop.</li><li>`input`: this is where we specify values for our resource's properties.

An example of the `copy` element looks like the below.

```JSON
...
{
	"name": "vnet01",
	"type": "Microsoft.Network/VirtualNetworks",
	"apiVersion": "2019-09-01",
	"location": "uksouth",
	"properties": {
		"addressSpace": {
			"addressPrefixes": [
				"192.168.0.0/16"
			]
		},
		"copy": [
			{
				"name": "subnets",
				"count": 3,
				"input": {
					"name": "[concat('subnet-', copyIndex('subnets', 1))]",
					"properties": {
						"addressPrefix": "[concat('192.168.', copyIndex('subnets', 1), '0/24')"
					}
				}
			}
		]
	}
}
...
```

The above simply creates 3 subnet with far fewer lines. You'll notice two functions I haven't explained yet: `[concat()]` and `[copyIndex()]`. The Microsoft docs do a good job on explaining these, but...

- `[concat()]` allows you to concatenate arrays or strings. It accepts at a minimum of 1 argument and any number of additional arguments. You can see this is used to define a unique name for each subnet.
- `[copyIndex()]` allows you to access the position/index of the current iterable in the loop. We pass two parameters: the name of the loop "subnets" and an offset. Using the offset is what enables us to start with creating subnet names from "subnets-1" rather than "subnets-0".


You notice I use both `[concat()]` and `[copyIndex()]` for the `addressPrefix` property. This enables me to bump the subnet by 1 and correlate the subnet name with the 3rd subnet octet.

There's definitely room for improvement here. The `count` object of the `copy` element is hardcoded at the value of 3. What we could do is leverage a parameter of our json value, which contains an array of strings for all the subnets we want in our virtual network.

The below is what an ideal could look like that creates subnets based on all the items in a given array:

```JSON
{
	"$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
	"parameters": {
		"subnetAddressSpaces": {
			"type": "array",
			"defaultValue": [
				"192.168.1.0/24",
				"192.168.2.0/24",
				"192.168.3.0/24"
			]
		}
	},
	"variables": {},
	"resources": [
		{
			"name": "vnet01",
			"type": "Microsoft.Network/VirtualNetworks",
			"apiVersion": "2019-09-01",
			"location": "uksouth",
			"properties": {
				"addressSpace": {
					"addressPrefixes": [
						"192.168.0.0/16"
					]
				},
				"copy": [
					{
						"name": "subnets",
						"count": "[length(parameters('subnetAddressSpaces'))]",
						"input": {
							"name": "[concat('subnet-', copyIndex('subnets', 1))]",
							"properties": {
								"addressPrefix": "[parameters('subnetAddressSpaces')[copyIndex('subnets')]]"
							}
						}
					}
				]
			}
		}
	]
}
```

In the above you can see I've included what the parameter definition this time, and defining the array of subset address spaces by specifying `defaultValue`.

What I'm doing differently here is using two different functions `[length()]` and `[parameters()]`.

- `[length()]` is what enables us to gather the number of elements within an array. In this case the value will be 3. This function can also be used to get the number of characters in a string or number of root-level properties in a given object.
- `[parameters()]` is hopefully obvious, but if not it's how we can access the value of a parameter by passing one of the template's parameter names. 

What the above in mind, you can see for the `count` object we now have a means to loop for x many iterations for y many elements in the `subnetAddressSpaces` array. You'll also notice, we're no longer concatenating a string to create our subnets' address space. Instead we're directly accessing the value in the array by referencing the index position of the returned by `[copyIndex()]`. In other words, we're grabbing the current value of the iterable in the loop.

Hopefully now you get the sense that this whole combination of the `copy` element, with functions `[length()]` and `[copyIndex()]` is pretty much a for in-range loop interpreted by Azure within our json.

## Objective 2

As for the second objective (dynamically creating resources, instead of dynamically setting properties of a resource's properties... wait, what?!), there's hardly any difference, except there's no need to use the `input` object within the `copy` element. 

Below I'll share a complete working example of the template and a short PowerShell snippet to create the deployment.

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualNetworkAddressSpaces": {
            "type": "array"
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[concat('vnet-', copyIndex('vnetloop', 1))]",
            "type": "Microsoft.Network/VirtualNetworks",
            "apiVersion": "2019-09-01",
            "location": "uksouth",
            "dependsOn": [],
            "tags": {},
            "copy": {
                "name": "vnetloop",
                "count": "[length(parameters('virtualNetworkAddressSpaces'))]"
            },
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[parameters('virtualNetworkAddressSpaces')[copyIndex('vnetloop')]]"
                    ]
                },
                "subnets": [
                    {
                        "name": "subnet0",
                        "properties": {
                            "addressPrefix": "[parameters('virtualNetworkAddressSpaces')[copyIndex('vnetloop')]]"
                        }
                    }
                ]
            }
        }
    ]
}
```

```powershell
$TemplateParameters = @{
    "virtualNetworkAddressSpaces" = @(
        "192.168.1.0/24",
        "192.168.2.0/24",
        "192.168.3.0/24"
    )
}

New-AzResourceGroupDeployment -Name "mytestdeployment" -ResourceGroupName "rg-mytest" -TemplateFile ".\vnet-template-2.json" -TemplateParameterObject $TemplateParameters
```

With the above we're still making use of the [length()] function for the count property of the copy element. This still enables us to loop over the number of elements within the given array. You'll also notice we're just creating 1 subnet for each virtual network. Each subnet's width is the entire width virtual network's address space.

## Summary

I know these examples are really short. You're really unlikely to want to _just_ dynamically create subnets or virtual networks, but hopefully this is offers you insightful examples to get you a working syntax for whatever it is you're trying to do!

The below Microsoft doc links may be helpful for you, if not ping me and I'll be happy to help!

- [ARM Template Functions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions)
- [Property iteration in ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/copy-properties)