# Pingdom provider #

This currently only supports working with:
- Basic HTTP
- Ping checks.

## Build and install ##

### Dependencies ###

You should have a working Go environment setup.  If not check out the Go [getting started](http://golang.org/doc/install) guide.

This use [dep](https://github.com/golang/dep) for dependency management.  To fetch all dependencies run `dep ensure`.

### Build ###

```
go get bitbucket.org/devops_sysops/pingdom-provider
```

The binary will then be available at `~/go/bin/pingdom-provider`, unless you've set `$GOPATH`, in which case you'll find it at `$GOPATH/bin`

### Install ###

You will need to install the binary as a [terraform third party plugin](https://www.terraform.io/docs/configuration/providers.html#third-party-plugins).  Terraform will then pick up the binary from the local filesystem when you run `terraform init`.

```
ln -s ~/go/bin/pingdom-provider ~/.terraform.d/plugins/$(uname | tr '[:upper:]' '[:lower:]')_amd64/terraform-provider-pingdom_v$(date +%Y.%m.%d)
```

Updates can now be applied by re-running `go get bitbucket.org/devops_sysops/pingdom-provider`.

## Usage ##

**Basic Check**

```
variable "pingdom_user" {}
variable "pingdom_password" {}
variable "pingdom_api_key" {}
variable "pingdom_account_email" {} # Optional: only required for multi-user accounts

provider "pingdom" {
    user = "${var.pingdom_user}"
    password = "${var.pingdom_password}"
    api_key = "${var.pingdom_api_key}"
    account_email = "${var.pingdom_account_email}" # Optional: only required for multi-user accounts
}

resource "pingdom_check" "example" {
    type = "http"
    name = "my http check"
    host = "example.com"
    resolution = 5
}

resource "pingdom_check" "example_with_alert" {
    type = "http"
    name = "my http check"
    host = "example.com"
    resolution = 5
    sendnotificationwhendown = 2
    integrationids = [
      12345678,
      23456789
    ]
}

resource "pingdom_check" "ping_example" {
    type = "ping"
    name = "my ping check"
    host = "example.com"
    resolution = 1
}
```

Apply with:
```
 terraform apply \
    -var 'pingdom_user=YOUR_USERNAME' \
    -var 'pingdom_password=YOUR_PASSWORD' \
    -var 'pingdom_api_key=YOUR_API_KEY'
```

**Using attributes from other resources**

```
variable "heroku_email" {}
variable "heroku_api_key" {}

variable "pingdom_user" {}
variable "pingdom_password" {}
variable "pingdom_api_key" {}

provider "heroku" {
    email = "${var.heroku_email}"
    api_key = "${var.heroku_api_key}"
}

provider "pingdom" {
    user = "${var.pingdom_user}"
    password = "${var.pingdom_password}"
    api_key = "${var.pingdom_api_key}"
}

resource "heroku_app" "example" {
    name = "my-app"
    region = "us"
}

resource "pingdom_check" "example" {
    name = "my check"
    host = "${heroku_app.example.heroku_hostname}"
    resolution = 5
}
```