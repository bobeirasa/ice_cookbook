Description
===========

Application cookbook for installing the Netflix Ice AWS usage
and cost reporting application.

Requirements
============

Chef 11.4.0+ and Ohai 6.10+ for platform_family use.

This cookbook requires attributes be set based on the instructions for 
configuring the [Netflix Ice application](https://github.com/Netflix/ice).

You will also need to enable Amazon's programmatic billing access to 
receive detailed billing reports.

The following cookbooks are dependencies:

* apt (on ubuntu)
* openssl
* java
* tomcat
* nginx
* artifact (Riot Games)

## Platform:

Tested on 

* Ubuntu 12.04
* Centos 6.4

Other Debian and RHEL family distributions are assumed to work.

Attributes
==========

* `node['ice']['version']` - Ice version to download and install.  These 
versions are packaged and hosted by Medidata Solutions until we can get the 
Netflix Ice team to package and host official ice releases.  If you wish to 
install the latest stable version from [Netflix](https://github.com/netflix/ice#download-snapshot-builds) directly, provide `stable`. 
<i>Note: this option will always install the latest version even if its not
backwards compatible.</i>
* `node['ice']['checksum']` - Checksum for Ice WAR file.
* `node['ice']['war_url']` - HTTP URL for Ice WAR file.
* `node['ice']['force_redeploy']` - Will force a redeploy of the Ice WAR file.
* `node['ice']['company_name']` - Organization name that is displayed in the 
UI header.
* `node['ice']['processor']['enabled']` - Enables the Ice processor.
* `node['ice']['processor']['local_dir']` - Local work directory for the Ice
processor.
* `node['ice']['billing_aws_access_key_id']` - AWS access key id used for
accessing AWS detailed billing files from S3.
* `node['ice']['billing_secret_key']` - AWS secret key used for
accessing AWS detailed billing files from S3.
* `node['ice']['billing_s3_bucket_name']` - Name of the S3 bucket containing
the AWS detailed billing files.
* `node['ice']['billing_s3_bucket_prefix']` - Directory in the S3 billing bucket 
containing AWS detailed billing files.
* `node['ice']['work_s3_bucket_name']` - Name of the S3 bucket that Ice uses 
for processed AWS detailed billing files.  This bucket is shared between the Ice
processor and reader.
* `node['ice']['work_s3_bucket_prefix']` - Directory in the S3 work bucket 
containing the processed AWS detailed billing files.
* `node['ice']['reader']['enabled']` - Enables the Ice reader and installs the
Nginx reverse proxy.
* `node['ice']['reader']['local_dir']` - Local work directory for the Ice reader.
* `node['ice']['start_millis']` - Specify the start time in milliseconds for the 
processor to start processing billing files.
* `node['ice']['public_hostname']` - Optional. Fully qualified domain name used for 
configuring the Nginx reverse proxy on Ice readers/UI nodes.
* `node['ice']['accounts']` - Optional.  Hash mapping of AWS account names to 
account numbers.  This is used within Ice to give accounts human readable names 
in the UI.
* `node['ice']['nginx_port']` - Optional.  Nginx port configuration. Default: 80.
* `node['ice']['nginx_config']` - Optional.  Nginx site configuration chef 
template name.  Default: 'nginx_ice_site.erb'.
* `node['ice']['nginx_config_cookbook']` - Optional. Nginx custom configuration
cookbook.  Use this if you'd like to bypass the default ice cookbook nginx 
configuration and implement your own templates and recipes to configure Nginx for
ice.  Default: 'ice'.
* `node['ice']['custom_resource_tags']` - Optional.  Array of custom resource tags
to have ice process.  As described in the ice README you must explicitly enable these
custom tags in your billing statements.

## Usage

This recipe allows you to deploy Netflix Ice as a standalone node running both the
processor and reader or as seperate nodes running a processor and a reader which is the
deployment layout that the Netflix Ice team recommends.

Here is a sample role for creating an Ice processor node:
```YAML
chef_type:           role
default_attributes:
description:         
env_run_lists:
json_class:          Chef::Role
name:                ice-processor
override_attributes:
  ice:
    billing_aws_access_key_id:     YOURAWSKEYID
    billing_aws_secret_key:        YOURAWSSECRETKEY
    billing_s3_bucket_name:        ice-billing
    version:                       0.0.4
    war_url:                       https://s3.amazonaws.com/dl.imedidata.net/ice
    checksum:                      eb9e7503585553bdebf9d93016bcbe7dc033c21e2b1b2f0df0978ca2968df047
    skip_manifest_check:           false
    company_name:                  Company Name
    force_deploy:                  false
    processor:
      enabled: true
    reader:
      enabled: false
    start_millis:                  1357016400000
    work_s3_bucket_name:           ice-work
  tomcat:
    catalina_options: -Xmx1024M -Xms1024M
run_list:
  recipe[ice]
```

Here is a sample role for creating an Ice reader node:
```YAML
chef_type:           role
default_attributes:
description:         
env_run_lists:
json_class:          Chef::Role
name:                ice-reader
override_attributes:
  ice:
    billing_aws_access_key_id:     YOURAWSKEYID
    billing_aws_secret_key:        YOURAWSSECRETKEY
    billing_s3_bucket_name:        ice-billing
    version:                       0.0.4
    war_url:                       https://s3.amazonaws.com/dl.imedidata.net/ice
    checksum:                      eb9e7503585553bdebf9d93016bcbe7dc033c21e2b1b2f0df0978ca2968df047
    skip_manifest_check:           false
    company_name:                  Company Name
    force_deploy:                  false
    processor:
      enabled: false
    reader:
      enabled: true
    start_millis:                  1357016400000
    work_s3_bucket_name:           ice-work
  tomcat:
    catalina_options: -Xmx1024M -Xms1024M
run_list:
  recipe[ice]
```

## License and Author

* Author: [Ray Rodriguez](https://github.com/rayrod2030)
* Contributor: [akshah123](https://github.com/akshah123)
* Contributor: [Benton Roberts](https://github.com/benton)
* Contributor: [Harry Wilkinson](https://github.com/harryw)
* Contributor: [rampire](https://github.com/rampire)

Copyright 2013 Medidata Solutions Worldwide

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
