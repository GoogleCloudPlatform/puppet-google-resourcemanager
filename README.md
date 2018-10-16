# Google Cloud Resource Manager Puppet Module

[![Puppet Forge](http://img.shields.io/puppetforge/v/google/gresourcemanager.svg)](https://forge.puppetlabs.com/google/gresourcemanager)

#### Table of Contents

1. [Module Description - What the module does and why it is useful](
    #module-description)
2. [Setup - The basics of getting started with Google Cloud Resource Manager](#setup)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](
   #reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

## Module Description

This Puppet module manages the resource of Google Cloud Resource Manager.
You can manage its resources using standard Puppet DSL and the module will,
under the hood, ensure the state described will be reflected in the Google
Cloud Platform resources.

## Setup

To install this module on your Puppet Master (or Puppet Client/Agent), use the
Puppet module installer:

    puppet module install google-gresourcemanager

Optionally you can install support to _all_ Google Cloud Platform products at
once by installing our "bundle" [`google-cloud`][bundle-forge] module:

    puppet module install google-cloud

Since this module depends on the `googleauth` and `google-api-client` gems,
you will also need to install those, with

    /opt/puppetlabs/puppet/bin/gem install googleauth google-api-client

If you prefer, you could also add the following to your puppet manifest:

		package { [
				'googleauth',
				'google-api-client',
			]:
				ensure   => present,
				provider => puppet_gem,
		}

## Usage

### Credentials

All Google Cloud Platform modules use an unified authentication mechanism,
provided by the [`google-gauth`][] module. Don't worry, it is automatically
installed when you install this module.

```puppet
gauth_credential { 'mycred':
  path     => $cred_path, # e.g. '/home/nelsonjr/my_account.json'
  provider => serviceaccount,
  scopes   => [
    'https://www.googleapis.com/auth/cloud-platform',
  ],
}
```

Please refer to the [`google-gauth`][] module for further requirements, i.e.
required gems.

### Examples

#### `gresourcemanager_project`

```puppet
gresourcemanager_project { 'My Sample Project':
  ensure     => present,
  id         => "test-project-${project_suffix}",
  credential => 'mycred',
}

```


### Classes

#### Public classes

* [`gresourcemanager_project`][]:
    Represents a GCP Project. A project is a container for ACLs, APIs, App
    Engine Apps, VMs, and other Google Cloud Platform resources.

### About output only properties

Some fields are output-only. It means you cannot set them because they are
provided by the Google Cloud Platform. Yet they are still useful to ensure the
value the API is assigning (or has assigned in the past) is still the value you
expect.

For example in a DNS the name servers are assigned by the Google Cloud DNS
service. Checking these values once created is useful to make sure your upstream
and/or root DNS masters are in sync.  Or if you decide to use the object ID,
e.g. the VM unique ID, for billing purposes. If the VM gets deleted and
recreated it will have a different ID, despite the name being the same. If that
detail is important to you you can verify that the ID of the object did not
change by asserting it in the manifest.

### Parameters

#### `gresourcemanager_project`

Represents a GCP Project. A project is a container for ACLs, APIs, App
Engine Apps, VMs, and other Google Cloud Platform resources.


#### Example

```puppet
gresourcemanager_project { 'My Sample Project':
  ensure     => present,
  id         => "test-project-${project_suffix}",
  credential => 'mycred',
}

```

#### Reference

```puppet
gresourcemanager_project { 'id-of-resource':
  create_time     => time,
  id              => string,
  labels          => keyvaluepairs,
  lifecycle_state => 'LIFECYCLE_STATE_UNSPECIFIED', 'ACTIVE', 'DELETE_REQUESTED' or 'DELETE_IN_PROGRESS',
  name            => string,
  number          => integer,
  parent          => {
    id   => string,
    type => string,
  },
  project         => string,
  credential      => reference to gauth_credential,
}
```

##### `name`

  The user-assigned display name of the Project. It must be 4 to 30
  characters. Allowed characters are: lowercase and uppercase letters,
  numbers, hyphen, single-quote, double-quote, space, and exclamation
  point.

##### `labels`

  The labels associated with this Project.
  Label keys must be between 1 and 63 characters long and must conform
  to the following regular expression: `[a-z]([-a-z0-9]*[a-z0-9])?`.
  Label values must be between 0 and 63 characters long and must
  conform to the regular expression `([a-z]([-a-z0-9]*[a-z0-9])?)?`.
  No more than 256 labels can be associated with a given resource.
  Clients should store labels in a representation such as JSON that
  does not depend on specific characters being disallowed

##### `parent`

  A parent organization

##### parent/type
  Must be organization.

##### parent/id
  Id of the organization

##### `id`

Required.  The unique, user-assigned ID of the Project. It must be 6 to 30
  lowercase letters, digits, or hyphens. It must start with a letter.
  Trailing hyphens are prohibited.


##### Output-only properties

* `number`: Output only.
  Number uniquely identifying the project.

* `lifecycle_state`: Output only.
  The Project lifecycle state.

* `create_time`: Output only.
  Time of creation


## Limitations

This module has been tested on:

* RedHat 6, 7
* CentOS 6, 7
* Debian 7, 8
* Ubuntu 12.04, 14.04, 16.04, 16.10
* SLES 11-sp4, 12-sp2
* openSUSE 13
* Windows Server 2008 R2, 2012 R2, 2012 R2 Core, 2016 R2, 2016 R2 Core

Testing on other platforms has been minimal and cannot be guaranteed.

## Development

### Automatically Generated Files

Some files in this package are automatically generated by
[Magic Modules][magic-modules].

We use a code compiler to produce this module in order to avoid repetitive tasks
and improve code quality. This means all Google Cloud Platform Puppet modules
use the same underlying authentication, logic, test generation, style checks,
etc.

Learn more about the way to change autogenerated files by reading the
[CONTRIBUTING.md][] file.

### Contributing

Contributions to this library are always welcome and highly encouraged.

See [CONTRIBUTING.md][] for more information on how to get
started.

### Running tests

This project contains tests for [rspec][], [rspec-puppet][] and [rubocop][] to
verify functionality. For detailed information on using these tools, please see
their respective documentation.

#### Testing quickstart: Ruby > 2.0.0

```
gem install bundler
bundle install
bundle exec rspec
bundle exec rubocop
```

#### Debugging Tests

In case you need to debug tests in this module you can set the following
variables to increase verbose output:

Variable                | Side Effect
------------------------|---------------------------------------------------
`PUPPET_HTTP_VERBOSE=1` | Prints network access information by Puppet provier.
`PUPPET_HTTP_DEBUG=1`   | Prints the payload of network calls being made.
`GOOGLE_HTTP_VERBOSE=1` | Prints debug related to the network calls being made.
`GOOGLE_HTTP_DEBUG=1`   | Prints the payload of network calls being made.

During test runs (using [rspec][]) you can also set:

Variable                | Side Effect
------------------------|---------------------------------------------------
`RSPEC_DEBUG=1`         | Prints debug related to the tests being run.
`RSPEC_HTTP_VERBOSE=1`  | Prints network expectations and access.

[magic-modules]: https://github.com/GoogleCloudPlatform/magic-modules
[CONTRIBUTING.md]: CONTRIBUTING.md
[bundle-forge]: https://forge.puppet.com/google/cloud
[`google-gauth`]: https://github.com/GoogleCloudPlatform/puppet-google-auth
[rspec]: http://rspec.info/
[rspec-puppet]: http://rspec-puppet.com/
[rubocop]: https://rubocop.readthedocs.io/en/latest/
[`gresourcemanager_project`]: #gresourcemanager_project
