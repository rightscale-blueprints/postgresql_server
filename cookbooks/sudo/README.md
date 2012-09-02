DESCRIPTION
===========

This cookbook installs sudo and configures the /etc/sudoers file.

REQUIREMENTS
============

Requires that the platform has a package named sudo and the sudoers file is /etc/sudoers.

ATTRIBUTES
==========

The following attributes are set to blank arrays:

    node['authorization']['sudo']['groups']
    node['authorization']['sudo']['users']

They are passed into the sudoers template which iterates over the values to add sudo permission to the specified users and groups.

If you prefer to use passwordless sudo just set the following attribute to true:

    node['authorization']['sudo']['passwordless']

This attribute controls whether or not to include the /etc/sudoers.d
directory, it default to false. If you set it to true, the default
recipe will create the directory /etc/sudoers.d and put the
placeholder file README there

    node['authorization']['sudo']['include_sudoers_d']

USAGE
=====

You can create sudoer entries in two ways,

- populating the node['authorization']['sudo']  properties
- using the sudo lwrp

To use this cookbook, set the attributes above on the node via a role or the node object itself. In a role.rb:

    "authorization" => {
      "sudo" => {
        "groups" => ["admin", "wheel", "sysadmin"],
        "users" => ["jerry", "greg"],
        "passwordless" => true
      }
    }

In JSON (role.json or on the node object):

    "authorization": {
      "sudo": {
        "groups": [
          "admin",
          "wheel",
          "sysadmin"
        ],
        "users": [
          "jerry",
          "greg"
        ],
        "passwordless": true
      }
    }

Note that the template for the sudoers file has the group "sysadmin" with ALL:ALL permission, though the group by default does not exist.

sudo LWRP
=========

**Note** Sudo version 1.7.2 or newer is required to use the sudo LWRP
  as it relies on the "#includedir" directive introduced in version
  1.7.2. The recipe does not enforce installing the version. To use
  this LWRP, set `node['authorization']['sudo']['include_sudoers_d']`
  to `true`.

This is a fairly complex LWRP for managing sudoers fragment files in
/etc/sudoers.d. It has two modes, "natural" mode which mimics the
sudoers file interface and "template" mode where you supply a regular
erb template and hash of variables. For "template" mode, the sudo lwrp
simply ensures that resulting sudo fragment passes validation and has
the proper filesystem permissions.

In either mode, the sudo lwrp will render a sudoers fragment in
/etc/sudoers.d/

In the case that the sudoers fragment does not pass validation, this
lwrp will fail the chef-client run before the fragment can be copied
to /etc/sudoers.d. This prevents the corruption of your sudoers configuration.

Example of the default mode, "natural" mode

    sudo "tomcat" do
      user "%tomcat" # or a username
      runas "app_user" # or "app_user : tomcat"
      commands ["/etc/init.d/tomcat restart"] # array of commands, will be .join(",")
      host "ALL"
      nopasswd false # true prepends the runas_spec with NOPASSWD
    end


Example of template mode

    sudo "tomcat"
      # this template must exist in the calling cookbook
      template "restart_tomcat.erb"
      variables( :cmds => [ "/etc/init.d/tomcat restart" ] )
    end

In either case, the following file would be generated in /etc/sudoers.d/tomcat

     # this file was generated by chef
     %tomcat ALL=(app_user) /etc/init.d/tomcat restart

Description of all attributes

* :name -- name of the file to be created in /etc/sudoers.d/ ,
  defaults to the name you use for the resource. An exception will be
  thrown if th
* :user -- user to provide sudo privileges to
* :group -- same as user except "%" is prepended to the name in
case it is not already
* :commands -- an array of commands that the user/group can execute using
sudo, must use the full path for each command, otherwise the resulting
fragment will fail validation
* :nopasswd -- whether or not a password must be supplied when
invoking sudo
* :template -- a template file in the current cookbook (not the sudo
cookbook), currently must be an erb template
* :variables -- variables to use with the template

If you use the template attribute, all other attributes will be
ignored except for the variables attribute.

LICENSE AND AUTHOR
==================

Author:: Bryan W. Berry <bryan.berry@gmail.com>
Author:: Adam Jacob <adam@opscode.com>
Author:: Seth Chisamore <schisamo@opscode.com>

Copyright 2009-2011, Opscode, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.