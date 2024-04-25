# CUSTOM ROLES IN GOOGLE CLOUD

Cloud IAM also provides the ability to create customized Cloud IAM roles. You create a custom role by combining one or more of the available Cloud IAM permissions.




## Prepare to create a custom role

 Before you create a custom role, you might want to know:  
 * What permissions can be applied to a resource 
 * What a role's metadata is
 * What roles are grantable on a resource 



##  What permissions can be applied to a resource 
To get the list of permissions available to a resource run:
#
    gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID

## What is the metadata
To view the role metadata, use command below, replacing [ROLE_NAME] with the role. For example: `roles/viewer` or `roles/editor`:

#
    gcloud iam roles describe [ROLE_NAME]

##  View the grantable roles on resources
Use the gcloud iam list-grantable-roles command to return a list of all roles that can be applied to a given resource.

Execute the following gcloud command to list grantable roles from your project:
#
    gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
## How to create a custom role
* To create a custom role, a caller must possess `iam.roles.create` permission. By default, **the owner of a project or an organization ** has this permission and can create and manage custom roles.
* Users who are not owners, including organization admins, must be assigned either the **Organization Role Administrator role**, or the **IAM Role Administrator role**.

Use the `gcloud iam roles create` command to create new custom roles in two ways:

* Provide a YAML file that contains the role definition
* Specify the role definition using flags

When creating a custom role, you must specify whether it applies to the organization level or project level by using the --organization [ORGANIZATION_ID] or --project [PROJECT_ID] flags.



## Creating a custom role at the project level

#### YAML FILE METHOD SYNTAX
```
title: [ROLE_TITLE]
description: [ROLE_DESCRIPTION]
stage: [LAUNCH_STAGE]
includedPermissions:
- [PERMISSION_1]
- [PERMISSION_2]
```

Each of the placeholder values is described below:

[ROLE_TITLE] is a friendly title for the role, such as Role Viewer.
[ROLE_DESCRIPTION] is a short description about the role, such as My custom role description.
[LAUNCH_STAGE] indicates the stage of a role in the launch lifecycle, such as ALPHA, BETA, or GA.
`includedPermissions` specifies the list of one or more permissions to include in the custom role, such as `iam.roles.get`.

#### Steps
1. Create a yaml file by running the following command:
#
    nano role-definition.yaml

2. Add the following to the file
```
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
```

3. Then save and close the file by pressing **CTRL+X**,** Y** and then **ENTER**.

4. Run the following command (Note the project id is specified):

#
    gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \--file role-definition.yaml

#### USING FLAGS
Run the following command:
#
    gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \--title "Role Viewer" --description "Custom role description." \--permissions compute.instances.get,compute.instances.list --stage ALPHA

#### Work with the custom roles
1. List roles specifying project or organization. To list deleted roles specify `--show-deleted` flag:
#
    gcloud iam roles list --project $DEVSHELL_PROJECT_ID

2. List pre-defined roles:
#
    gcloud iam roles list


    

## Updating an existing custom role
if two owners for a project try to make conflicting changes to a role at the same time, some changes could fail. Cloud IAM solves this problem using an `etag` property in custom roles.
An update can be made to the custom role using the yaml method and the flags method as well:

#### YAML
1. Get the current definition for the role by executing the following gcloud command, replacing [ROLE_ID] with editor (for instance).

#
    gcloud iam roles describe [ROLE_ID] --project $DEVSHELL_PROJECT_ID
2. Copy the output to use to create a new YAML file in the next steps.
3. Create a new-role-definition.yaml file with your editor
4. Paste under `includedPermissions`:
```
- storage.buckets.get
- storage.buckets.list
```
5. See sample of what the yaml file looks like:
```
description: Edit access for App Versions
etag: BwVxIAbRq_I=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
- storage.buckets.get
- storage.buckets.list
name: projects/qwiklabs-gcp-03-ebf313700a52/roles/editor
stage: ALPHA
title: Role Editor
```
6. Save and close editor. Execute the following command:
#
    gcloud iam roles update [ROLE_ID] --project $DEVSHELL_PROJECT_ID \--file new-role-definition.yaml


#### USING FLAGS
Using flags, use `--add-permissions` or `--remove-permissions` or `--permissions [PERMISSIONS]` (with the commands separated by comma)

#
    gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \--add-permissions storage.buckets.get,storage.buckets.list

## Disable a custom role:
The following command disables a **viewer** role
#
    gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \--stage DISABLED

## Delete a custom role
#
    gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID

## Restore a deleted role 
This can be done within 7 days:
#
    gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
