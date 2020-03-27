---
layout: post
title: "Avoiding Azure Functions Downtime with Slots"
date: 2020-03-27 05:00:00
description: "What happens when you upload a new version of your Azure Function?"
image: ""
---

This week I was building a Terraform module to deploy an Azure Functions infrastructure – including Storage Account, Service Plan and the Function App – so later the development team can simply use the `func` CLI to deploy they functions themselves.

In the middle of my experiments, I thought about the Function update process: what happens when I change the source code and deploy a new version?

> Here I am only exploring the downtime issue. Please refer to [this](https://vgaltes.com/post/deploying-azure-functions-using-terraform) article to learn how to deploy Azure Functions with Terraform._

## Test #01 – Terraform Does Everything ™️
In this scenario, the Function App deployed by Terraform points directly to a zip file in an Azure Container, like the code below:
```terraform
resource "azurerm_function_app" "fn_app" {
  name                      = "my-name"
  location                  = "westeurope"
  resource_group_name       = "my-rg"
  app_service_plan_id       = "my-plan"
  storage_connection_string = "my-sa-connection-string"
  version                   = "~3"

  app_settings = {
    https_only                   = true
    FUNCTIONS_WORKER_RUNTIME     = "node"
    WEBSITE_NODE_DEFAULT_VERSION = "~12"
    # the secret sauce!
    # points to a zip file in a container
    WEBSITE_USE_ZIP              = "https://${azurerm_storage_account.mysa.name}.blob.core.windows.net/${azurerm_storage_container.my-ct.name}/my-code-1.0.1.zip${data.azurerm_storage_account_sas.my-sas.sas}"
  }
}
```

From the developer's perspective, the Function release process looks like this:
1. Change code
2. Create a zip file with the source code, with a specific name like `my-code-1.0.1.zip`
3. Upload the zip to an Azure Container
4. Change the Terraform code to point to the uploaded file
5. Run `terraform apply`

Let's assume the Function exposes a simple HTTP endpoint. If before step `4` we start sending traffic to the function – one request every half second – this is what we get:
```bash
This is my code v1.0.1! - 15:49:30
This is my code v1.0.1! - 15:49:31
This is my code v1.0.1! - 15:49:32
This is my code v1.0.1! - 15:49:32
This is my code v1.0.1! - 15:49:33
The service is unavailable. - 15:49:34
This is my code v1.0.2! - 15:49:54
This is my code v1.0.2! - 15:49:55
This is my code v1.0.2! - 15:49:56
This is my code v1.0.2! - 15:49:56
This is my code v1.0.2! - 15:49:57
```

During the `terraform apply` execution we have a **~20 seconds downtime**! Not good!

## Test #02 – Terraform + func ™️
_Alright, the whole zip name changing dance should be causing downtime... `func` will solve this!_ That's what I though.

In the second scenario, we still create the whole Functions App infrastructure with Terraform, but we omit the `WEBSITE_USE_ZIP` variable: instead, the developer will use the `func` CLI to deploy the Function itself, pointing to the Function App created by Terraform.

Again, the flow would look like this:
1. Deploy infrastructure with Terraform
2. Output the Function App name, let's say `fn-app`
3. Within the Function repository, use the `func` CLI to deploy, like `func azure functionapp publish fn-app`
4. Update the source code
5. Deploy again with `func`

And, once again during the step `5`, we got almost **~20 seconds of downtime** :(
```bash
This is my code v1.0.1! - 15:59:20
This is my code v1.0.1! - 15:59:21
This is my code v1.0.1! - 15:59:22
This is my code v1.0.1! - 15:59:22
This is my code v1.0.1! - 15:59:23
The service is unavailable. - 15:59:24
The service is unavailable. - 15:59:25
This is my code v1.0.2! - 15:59:44
This is my code v1.0.2! - 15:59:45
This is my code v1.0.2! - 15:59:46
This is my code v1.0.2! - 15:59:46
This is my code v1.0.2! - 15:59:47
```

> The standard way of developing and deploying Azure Functions – with the `func` CLI – **will cause downtime** D:

Although twenty seconds may OK for you, it's a thing to be aware of.

## Slots to the Rescue
There is one Azure way to solve it: Slots! Basically, it allows an user to deploy different Functions and hot swap them, without downtime. So one might have a Slot called `prod` with with a Function on the version `1.0.1` and another called `staging` with a Function named `1.0.2`. Once `staging` is tested and ready to go, you _swap_ them, so `prod` becomes `1.0.2`.

## Test #03 - Using slots
1. Create the two previous mentioned slots (you need a Resource Group with a Function App already created):
  - `az functionapp deployment slot create --name fn-app --resource-group rg --slot staging`
  - `az functionapp deployment slot create --name fn-app --resource-group rg --slot prod`
2. Zip your Function folder and deploy it to `prod` (you can only deploy Functions to Slots with a zip file and using the `az` CLI):
  - `az functionapp deployment source config-zip -g rg -n fn-app --src my-code-1.0.1.zip --slot prod`
3. Change the source code and deploy it to the `staging` – like deploying a new version:
  - `az functionapp deployment source config-zip -g rg -n fn-app --src my-code-1.0.2.zip --slot staging`
4. Once you are happy with the `staging` version, _swap_ it with `prod`. Think about it like _promoting_ the new version:
  - `az webapp deployment slot swap -g rg -n fn-app --slot staging --target-slot prod`

If you continuously send requests to the `prod` Function during the swap, you'll see something like this:
```bash
This is my code v1.0.1! - 17:10:22
This is my code v1.0.1! - 17:10:23
This is my code v1.0.1! - 17:10:23
This is my code v1.0.1! - 17:10:24
This is my code v1.0.2! - 17:10:24
This is my code v1.0.2! - 17:10:25
This is my code v1.0.1! - 17:10:25
This is my code v1.0.1! - 17:10:25
This is my code v1.0.1! - 17:10:25
This is my code v1.0.2! - 17:10:26
This is my code v1.0.2! - 17:10:27
This is my code v1.0.2! - 17:10:27
This is my code v1.0.1! - 17:10:28
This is my code v1.0.2! - 17:10:28
This is my code v1.0.2! - 17:10:29
This is my code v1.0.1! - 17:10:30
This is my code v1.0.1! - 17:10:31
This is my code v1.0.1! - 17:10:31
This is my code v1.0.2! - 17:10:32
This is my code v1.0.2! - 17:10:32
This is my code v1.0.2! - 17:10:33
This is my code v1.0.2! - 17:10:34
...
```

The Function is being _hot swapped_, and traffic is hitting both Functions until the swap is over. DONE! You've migrated your function without any downtime ✨

Let's use Slots everywhere, right? Well, if you can afford, yes. The basic (called Consumer) Azure Functions plan has a limit of _one_ Slot, leaving you with no other Slot to do the swap. Meanwhile, you can have multiple Slots with the Premium plan. The only issue is the price: Functions in the Premium plan are charged by _hour_ instead of execution. Check it out:

![](/img/azure_functions_price_consumer.png)
<center>_Azure Functions Consumer Tier_</center>

![](/img/azure_functions_price_premium.png)
<center>_Azure Functions Premium Tier_</center>

So, yeah, as far as I know, Azure Functions with zero downtime is only for the ones who can afford.
