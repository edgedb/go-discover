# Go Discover Nodes for Cloud Providers [![Build Status](https://travis-ci.org/hashicorp/go-discover.svg?branch=master)](https://travis-ci.org/hashicorp/go-discover) [![GoDoc](https://godoc.org/github.com/hashicorp/go-discover?status.svg)](https://godoc.org/github.com/hashicorp/go-discover)


`go-discover` is a Go (golang) library and command line tool to discover
ip addresses of nodes in cloud environments based on meta information
like tags provided by the environment.

The configuration for the providers is provided as a list of `key=val key=val
...` tuples. If either the key or the value contains a space (` `), a backslash
(`\`) or double quotes (`"`) then it needs to be quoted with double quotes.
Within a quoted string you can use the backslash to escape double quotes or the
backslash itself, e.g. `key=val "some key"="some value"`

Duplicate keys are reported as error and the provider is determined through the
`provider` key.

### Supported Providers

The following cloud providers have implementations in the go-discover/provider
sub packages. Additional providers can be added through the
[Register](https://godoc.org/github.com/hashicorp/go-discover#Register)
function.

 * Aliyun (Alibaba) Cloud [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/aliyun/aliyun_discover.go#L15-L28)
 * Amazon AWS [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/aws/aws_discover.go#L19-L33)
 * DigitalOcean [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/digitalocean/digitalocean_discover.go#L16-L24)
 * Google Cloud [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/gce/gce_discover.go#L17-L37)
 * Microsoft Azure [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/azure/azure_discover.go#L16-L37)
 * Openstack [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/os/os_discover.go#L23-L38)
 * Scaleway [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/scaleway/scaleway_discover.go#L14-L22)
 * SoftLayer [Config options](https://github.com/hashicorp/go-discover/blob/master/provider/softlayer/softlayer_discover.go#L16-L25)

HashiCorp maintains acceptance tests that regularly allocate and run tests with
real resources to verify the behavior of several of these providers. Those
currently are: Amazon AWS, Microsoft Azure, Google Cloud, and DigitalOcean.

### Config Example

```
# Aliyun (Alibaba) Cloud
provider=aliyun region=... tag_key=consul tag_value=... access_key_id=... access_key_secret=...

# Amazon AWS
provider=aws region=eu-west-1 tag_key=consul tag_value=... access_key_id=... secret_access_key=...

# DigitalOcean
provider=digitalocean region=... tag_name=... api_token=...

# Google Cloud
provider=gce project_name=... zone_pattern=eu-west-* tag_value=consul credentials_file=...

# Microsoft Azure
provider=azure tag_name=consul tag_value=... tenant_id=... client_id=... subscription_id=... secret_access_key=...

# Openstack
provider=os tag_key=consul tag_value=server username=... password=... auth_url=...

# Scaleway
provider=scaleway organization=my-org tag_name=consul-server token=... region=...

# SoftLayer
provider=softlayer datacenter=dal06 tag_value=consul username=... api_key=...

```

## Command Line Tool Usage

Install the command line tool with:

```
go get -u github.com/hashicorp/go-discover/cmd/discover
```

Then run it with:

```
$ discover addrs provider=aws region=eu-west-1 ...
```

## Library Usage

Install the library with:

```
go get -u github.com/hashicorp/go-discover
```

You can then either support discovery for all available providers
or only for some of them.

```go
// support discovery for all supported providers
d := discover.Discover{}

// support discovery for AWS and GCE only
d := discover.Discover{
	Providers : map[string]discover.Provider{
		"aws": discover.Providers["aws"],
		"gce": discover.Providers["gce"],
	}
}

// use ioutil.Discard for no log output
l := log.New(os.Stderr, "", log.LstdFlags)

cfg := "provider=aws region=eu-west-1 ..."
addrs, err := d.Addrs(cfg, l)
```

For complete API documentation, see
[GoDoc](https://godoc.org/github.com/hashicorp/go-discover). The configuration
for the supported providers is documented in the
[providers](https://godoc.org/github.com/hashicorp/go-discover/provider)
sub-package.

## Testing

Configuration tests can be run with Go:

```
$ go test ./...
```

By default tests that communicate with providers do not run unless credentials
are set for that provider. To run provider tests you must set the necessary
environment variables.

**Note: This will make real API calls to the account provided by the credentials.**

```
$ AWS_ACCESS_KEY_ID=... AWS_ACCESS_KEY_SECRET=... AWS_REGION=... go test -v ./provider/aws
```

This requires resources to exist that match those specified in tests
(eg instance tags in the case of AWS). To create these resources,
there are sets of [Terraform](https://www.terraform.io) configuration
in the `test/tf` directory for supported providers.

You must use the same account and access credentials above. The same
environment variables should be applicable and read by Terraform.

```
$ cd test/tf/aws
$ export AWS_ACCESS_KEY_ID=... AWS_ACCESS_KEY_SECRET=... AWS_REGION=...
$ terraform init
...
$ terraform apply
...
```

After Terraform successfully runs, you should be able to successfully
run the tests, assuming you have exported credentials into
your environment:

```
$ go test -v ./provider/aws
```

To destroy the resources you need to use Terraform again:

```
$ cd test/tf/aws
$ terraform destroy
...
```

**Note: There should be no requirements to create and test these resources other
than credentials and Terraform. This is to ensure tests can run in development
and CI environments consistently across all providers.**

## Retrieving Test Credentials

Below are instructions for retrieving credentials in order to run
tests for some of the providers.

<details>
  <summary>Google Cloud</summary>

1. Go to https://console.cloud.google.com/
1. IAM &amp; Admin / Settings:
    * Create Project, e.g. `discover`
    * Write down the `Project ID`, e.g. `discover-xxx`
1. Billing: Ensure that the project is linked to a billing account
1. API Manager / Dashboard: Enable the following APIs
    * Google Compute Engine API
1. IAM &amp; Admin / Service Accounts: Create Service Account
    * Service account name: `admin`
    * Roles:
        * `Project/Service Account Actor`
        * `Compute Engine/Compute Instance Admin (v1)`
        * `Compute Engine/Compute Security Admin`
    * Furnish a new private key: `yes`
    * Key type: `JSON`
1. The credentials file `discover-xxx.json` will have been downloaded
   automatically to your machine
1. Source the contents of the credentials file into the `GOOGLE_CREDENTIALS`
   environment variable

</details>

<details>
  <summary>Azure</summary>
See also the [Terraform provider documentation](https://www.terraform.io/docs/providers/azurerm/index.html#creating-credentials).

```shell
# Install Azure CLI (https://github.com/Azure/azure-cli)
curl -L https://aka.ms/InstallAzureCli | bash

# 1. Login
$ az login

# 2. Get SubscriptionID
$ az account list
[
  {
    "cloudName": "AzureCloud",
    "id": "subscription_id",
    "isDefault": true,
    "name": "Gratis versie",
    "state": "Enabled",
    "tenantId": "tenant_id",
    "user": {
      "name": "user@email.com",
      "type": "user"
    }
  }
]

# 3. Switch to subscription
$ az account set --subscription="subscription_id"

# 4. Create ClientID and Secret
$ az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/subscription_id"
{
  "appId": "client_id",
  "displayName": "azure-cli-2017-07-18-16-51-43",
  "name": "http://azure-cli-2017-07-18-16-51-43",
  "password": "client_secret",
  "tenant": "tenant_id"
}

# 5. Export the Credentials for the client
export ARM_CLIENT_ID=client_id
export ARM_CLIENT_SECRET=client_secret
export ARM_TENANT_ID=tenant_id
export ARM_SUBSCRIPTION_ID=subscription_id

# 6. Test the credentials
$ az vm list-sizes --location 'West Europe'
```
</details>
