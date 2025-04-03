---
layout: "azurerm"
page_title: "Provider: Azure"
description: |-
  The Azure Provider is used to interact with the many resources supported by Azure Resource Manager (also known as AzureRM) through its APIs.

---

# Azure Provider

* [Azure's Data Sources](d)
* [Azure's Resources](r)

## Authenticating to Azure / -- supported by -- Terraform

* SUPPORTED methods
  * [-- via -- Azure CLI](guides/azure_cli.html.markdown)
    * if you run Terraform locally -> recommended
  * [-- via -- Managed Service Identity](guides/managed_service_identity.html.markdown)
    * if you run Terraform non-interactively (_Example:_ | CI server) -> recommended
  * [-- via -- Service Principal + Client Certificate](guides/service_principal_client_certificate.html)
    * if you run Terraform non-interactively (_Example:_ | CI server) -> recommended
  * [-- via -- Service Principal + Client Secret](guides/service_principal_client_secret.html)
    * if you run Terraform non-interactively (_Example:_ | CI server) -> recommended
  * [-- via -- OpenID Connect](guides/service_principal_oidc.html)
* requirements
  * ⚠️User, Service Principal or Managed Identity / run Terraform -> should have permissions -- to register -- [Azure Resource Providers](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types)⚠️ 
    * if INSUFFICIENT permissions -> set [`resource_provider_registrations=none`](#resource-provider-registrations)
      * Reason: 🧠prevent auto-registration🧠

## Example Usage

```hcl
# We strongly recommend using the required_providers block to set the
# Azure Provider source and version being used
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  resource_provider_registrations = "none" # This is only required when the User, Service Principal, or Identity running Terraform lacks the permissions to register Azure Resource Providers.
  features {}
}

# Create a resource group
resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

# Create a virtual network within the resource group
resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
}
```

## Argument Reference

* SUPPORTED arguments
  * `features`
    * REQUIRED
    * [here](#features----features)
  * `subscription_id`
    * REQUIRED
    * TODO: The Subscription ID which should be used. 
      * This can also be sourced from the `ARM_SUBSCRIPTION_ID` Environment Variable.
        * `subscription_id` property is required when performing a plan or apply operation, but is not required to run `terraform validate`.
  * `client_id` - (Optional) The Client ID which should be used. 
    * This can also be sourced from the `ARM_CLIENT_ID` Environment Variable.
  * `client_id_file_path` (Optional) The path to a file containing the Client ID which should be used. 
    * This can also be sourced from the `ARM_CLIENT_ID_FILE_PATH` Environment Variable.
  * `environment` - (Optional) The Cloud Environment which should be used. 
    * Possible values are `public`, `usgovernment`, `german`, and `china`. Defaults to `public`. 
    * This can also be sourced from the `ARM_ENVIRONMENT` Environment Variable. 
    * Not used when `metadata_host` is specified.
  * `tenant_id` - (Optional) The Tenant ID which should be used. 
    * This can also be sourced from the `ARM_TENANT_ID` Environment Variable.
  * `auxiliary_tenant_ids` - (Optional) List of auxiliary Tenant IDs required for multi-tenancy and cross-tenant scenarios. 
    * This can also be sourced from the `ARM_AUXILIARY_TENANT_IDS` Environment Variable.

---

When authenticating as a Service Principal using a Client Certificate, the following fields can be set:

* `client_certificate` - (Optional) A base64-encoded PKCS#12 bundle to be used as the client certificate for authentication. This can also be sourced from the `ARM_CLIENT_CERTIFICATE` environment variable.

* `client_certificate_password` - (Optional) The password associated with the Client Certificate. This can also be sourced from the `ARM_CLIENT_CERTIFICATE_PASSWORD` Environment Variable.

* `client_certificate_path` - (Optional) The path to the Client Certificate associated with the Service Principal which should be used. This can also be sourced from the `ARM_CLIENT_CERTIFICATE_PATH` Environment Variable.

More information on [how to configure a Service Principal using a Client Certificate can be found in this guide](guides/service_principal_client_certificate.html).

---

When authenticating as a Service Principal using a Client Secret, the following fields can be set:

* `client_secret` - (Optional) The Client Secret which should be used. This can also be sourced from the `ARM_CLIENT_SECRET` Environment Variable.

* `client_secret_file_path` - (Optional) The path to a file containing the Client Secret which should be used. This can also be sourced from the `ARM_CLIENT_SECRET_FILE_PATH` Environment Variable.

More information on [how to configure a Service Principal using a Client Secret can be found in this guide](guides/service_principal_client_secret.html).

---

When authenticating as a Service Principal using Open ID Connect, the following fields can be set:

* `oidc_request_token` - (Optional) The bearer token for the request to the OIDC provider. This can also be sourced from the `ARM_OIDC_REQUEST_TOKEN`, `ACTIONS_ID_TOKEN_REQUEST_TOKEN` or `SYSTEM_ACCESSTOKEN` Environment Variables. The provider will look for values in this order and use the first it finds configured.

* `oidc_request_url` - (Optional) The URL for the OIDC provider from which to request an ID token. This can also be sourced from the `ARM_OIDC_REQUEST_URL`, `ACTIONS_ID_TOKEN_REQUEST_URL` or `SYSTEM_OIDCREQUESTURI` Environment Variables. The provider will look for values in this order and use the first it finds configured.

* `ado_pipeline_service_connection_id` - (Optional) The Azure DevOps Pipeline Service Connection ID. This can also be sourced from the `ARM_ADO_PIPELINE_SERVICE_CONNECTION_ID` or `ARM_OIDC_AZURE_SERVICE_CONNECTION_ID` Environment Variables. The provider will look for values in this order and use the first it finds configured.

* `oidc_token` - (Optional) The ID token when authenticating using OpenID Connect (OIDC). This can also be sourced from the `ARM_OIDC_TOKEN` environment Variable.

* `oidc_token_file_path` - (Optional) The path to a file containing an ID token when authenticating using OpenID Connect (OIDC). This can also be sourced from the `ARM_OIDC_TOKEN_FILE_PATH` Environment Variable.

* `use_oidc` - (Optional) Should OIDC be used for Authentication? This can also be sourced from the `ARM_USE_OIDC` Environment Variable. Defaults to `false`.

More information on [how to configure a Service Principal using OpenID Connect can be found in this guide](guides/service_principal_oidc.html).

---

When authenticating using Managed Identity, the following fields can be set:

* `msi_endpoint` - (Optional) The path to a custom endpoint for Managed Identity - in most circumstances, this should be detected automatically. This can also be sourced from the `ARM_MSI_ENDPOINT` Environment Variable.

* `use_msi` - (Optional) Should Managed Identity be used for Authentication? This can also be sourced from the `ARM_USE_MSI` Environment Variable. Defaults to `false`.

More information on [how to configure a Service Principal using Managed Identity can be found in this guide](guides/managed_service_identity.html).

---

When authenticating using AKS Workload Identity, the following fields can be set:

* `use_aks_workload_identity` - (Optional) Should AKS Workload Identity be used for Authentication? This can also be sourced from the `ARM_USE_AKS_WORKLOAD_IDENTITY` Environment Variable. Defaults to `false`. When set, `client_id`, `tenant_id` and `oidc_token_file_path` will be detected from the environment and do not need to be specified.

More information on [how to configure AKS Workload Identity can be found in this guide](guides/aks_workload_identity.html).

---

For Azure CLI authentication, the following fields can be set:

* `use_cli` - (Optional) Should Azure CLI be used for authentication? This can also be sourced from the `ARM_USE_CLI` environment variable. Defaults to `true`.

---

For some advanced scenarios, such as where more granular permissions are necessary - the following properties can be set:

* `disable_terraform_partner_id` - (Optional) Disable sending the Terraform Partner ID if a custom `partner_id` isn't specified, which allows Microsoft to better understand the usage of Terraform. The Partner ID does not give HashiCorp any direct access to usage information. This can also be sourced from the `ARM_DISABLE_TERRAFORM_PARTNER_ID` environment variable. Defaults to `false`.

* `metadata_host` - (Optional) The Hostname of the Azure Metadata Service (for example `management.azure.com`), used to obtain the Cloud Environment when using a Custom Azure Environment. This can also be sourced from the `ARM_METADATA_HOSTNAME` Environment Variable.

~> **Note:** `environment` must be set to the requested environment name in the list of available environments held in the `metadata_host`.

* `partner_id` - (Optional) A GUID/UUID registered with Microsoft to facilitate partner resource [usage attribution](https://docs.microsoft.com/azure/marketplace/azure-partner-customer-usage-attribution). This can also be sourced from the `ARM_PARTNER_ID` Environment Variable. Supported formats are `<guid>` / `pid-<guid>` (GUIDs [registered](https://docs.microsoft.com/azure/marketplace/azure-partner-customer-usage-attribution#other-use-cases) in Partner Center) and `pid-<guid>-partnercenter` (for published [commercial marketplace Azure apps](https://docs.microsoft.com/azure/marketplace/azure-partner-customer-usage-attribution#commercial-marketplace-azure-apps)).

* `auxiliary_tenant_ids` - (Optional) Contains a list of (up to 3) other Tenant IDs used for cross-tenant and multi-tenancy scenarios with multiple AzureRM provider definitions. The list of `auxiliary_tenant_ids` in a given AzureRM provider definition contains the other, remote Tenants and should not include its own `subscription_id` (or `ARM_SUBSCRIPTION_ID` Environment Variable).

* `resource_provider_registrations` - (Optional) Specifies a pre-determined set of [Azure Resource Providers](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types) to automatically register when initializing the AzureRM Provider. Allowed values for this property are `core`, `extended`, `all`, or `none`. This can also be sourced from the `ARM_RESOURCE_PROVIDER_REGISTRATIONS` environment variable. For more information about which resource providers each set contains, see the [Resource Provider Registrations](#resource-provider-registrations) section below.

* `resource_providers_to_register` - (Optional) A list of arbitrary [Azure Resource Providers](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types) to automatically register when initializing the AzureRM Provider. Can be used in combination with the `resource_provider_registrations` property. For more information, see the [Resource Provider Registrations](#resource-provider-registrations) section below.

-> By default, Terraform will attempt to register any Resource Providers that it supports, even if they're not used in your configurations, to be able to display more helpful error messages. If you're running in an environment with restricted permissions, or wish to manage Resource Provider Registration outside of Terraform you may wish to disable this by setting `resource_provider_registrations` to `none`; however, please note that the error messages returned from Azure may be confusing as a result.

* `storage_use_azuread` - (Optional) Should the AzureRM Provider use AzureAD to connect to the Storage Blob & Queue APIs, rather than the SharedKey from the Storage Account? This can also be sourced from the `ARM_STORAGE_USE_AZUREAD` Environment Variable. Defaults to `false`.

~> **Note:** This requires that the User/Service Principal being used has the associated `Storage` roles - which are added to new Contributor/Owner role-assignments, but **have not** been backported by Azure to existing role-assignments.

~> **Note:** The Files Storage API does not support authenticating via AzureAD and will continue to use a SharedKey when AAD authentication is enabled.

It's also possible to use multiple Provider blocks within a single Terraform configuration, for example, to work with resources across multiple Subscriptions - more information can be found [in the documentation for Providers](https://www.terraform.io/docs/configuration/providers.html#multiple-provider-instances).

## Features -- `features{}`

* allows
  * configuring the Azure Provider's resources' behaviour
* see [here](guides/features-block.html.markdown)

## Resource Provider Registrations

Before each plan or apply operation, the AzureRM Provider attempts to ensure that necessary Azure Resource Providers are registered. This process enables the necessary APIs and services for the provider to work with Azure. By default, the provider will attempt to register a small set of resource providers, which provides coverage for the most common resource types that are supported by the provider.

You can customise this behavior as needed by setting the `resource_provider_registrations` provider property, or the `resource_providers_to_register` provider property, or both of these. The `resource_provider_registrations` property indicates a pre-determined set of resource providers to automatically register, and are:

* `core` - a small set of resource providers for essential services including Compute, Networking and Storage.
* `extended` - a larger set that provides coverage for the most common supported resources.
* `all` - a set of resource providers that enables every resource in the provider to be used.
* `none` - with this setting, the provider will not attempt to register any resource providers.

To view the latest specific Azure Resource Providers included in each set, please refer to [this page](https://github.com/hashicorp/terraform-provider-azurerm/blob/main/internal/resourceproviders/required.go) on GitHub.

In addition to, or in place of, the sets described above, you can also configure the AzureRM Provider to register specific Azure Resource Providers, by setting the `resource_providers_to_register` provider property. This should be a list of strings, containing the exact names of Azure Resource Providers to register. For a list of all resource providers, please refer to [official Azure documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types).

-> **Note on Permissions** The User, Service Principal or Managed Identity running Terraform should have permissions to register [Azure Resource Providers](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types). If the principal running Terraform has insufficient permissions to register Resource Providers then we recommend setting the property [`resource_provider_registrations`](#resource_provider_registrations) to `none` in the provider block to prevent auto-registration.
