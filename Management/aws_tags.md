# Assigning Tags to AWS Objects
When creating AWS objects, certain tags are required for billing, identification and management purposes. This illustrates which tags are required and why.

## The required Tags

| Tag | Use |
|--|--|
| Name | This tag is meant to provide a meaningful name to your AWS object. For the purposes of the Egar project, this is usually prefixed with 'EGAR-' (eg: EGAR-HOST-1) |
| Billing | This tag is used for billing, according to the type of object. For this reason, values in this tag should be the instance type (eg: 'ec2'). Cost Explorer breaks down billing according to the presence of this tag as well as the tag-value to establish what the billing is for specifically. (eg EGAR )
| Project | This tag is used to establish which hosts should be brought down overnight and started up the following morning. This is run via a Lambda function called through Amazon Cloud Watch, which schedules all EC2 instances with a 'project' tag with value 'egar' t be brought down at 8pm each night and up at 8am each morning.
| Owner | The name of the person who owns the object/instance. This can be the creator or the regular user.
| Purpose | A short description of the purpose of the instance (eg: 'Drone server' or 'Development machine for user X')
