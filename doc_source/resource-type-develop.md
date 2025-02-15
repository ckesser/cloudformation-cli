# Developing resource types for use in AWS CloudFormation Templates<a name="resource-type-develop"></a>

Once you've modeled your resource type, and validated its schema, the next step is to develop the resource\. Developing the resource consists of two main steps:
+ Implementing the appropriate event handlers for your resource\.
+ Testing the resource locally to ensure it works as expected\.

## Implementing resource handlers<a name="resource-type-develop-implement-handlers"></a>

When you `generate` your resource package, the CloudFormation CLI stubs out empty handler functions, each of which each corresponds to a specific event in the resource lifecycle\. You add logic to these handlers to control what happens to your resource type at each stage of its lifecycle\.
+ `create`: CloudFormation invokes this handler when the resource is initially created during stack create operations\.
+ `read`: CloudFormation invokes this handler as part of a stack update operation when detailed information about the resource's current state is required\.
+ `update`: CloudFormation invokes this handler when the resource is updated as part of a stack update operation\.
+ `delete`: CloudFormation invokes this handler when the resource is deleted, either when the resource is deleted from the stack as part of a stack update operation, or the stack itself is deleted\.
+ `list`: CloudFormation invokes this handler when summary information about multiple resources of this resource type is required\.

You can only specify a single handler for each event\.

Which handlers you implement for a resource determine what provisioning actions CloudFormation takes with respect to the resource during various stack operations:
+ If the resource type contains both `create` and `update` handlers, CloudFormation invokes the appropriate handler during stack create and update operation\.
+ If the resource type does not contain an `update` handler, CloudFormation cannot update the resource during stack update operations, and will instead replace it\. CloudFormation invokes the `create` handler to creates a new resource, then deletes the old resource by invoking the `delete` handler\.
+ If the resource type does not contain `create`, `read`, and `delete` handlers, CloudFormation cannot actually provision the resource\.

Use the resource schema to specify which handlers you have implemented\. If you choose not to implement a specific handler, remove it from the `handlers` section of the resource schema\.

### Accessing AWS APIs from a resource type<a name="resource-type-develop-executionrole"></a>

If your resource type calls AWS APIs in any of its handlers, you must create an *[IAM execution role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)* that includes the necessary permissions to call those AWS APIs, and provision that execution role in your account\. CloudFormation then assumes that execution role to provide your resource type with the appropriate credentials\.

When you call `generate`, the CloudFormation CLI automatically generates an execution role template, `resource-role.yaml`, as part of generating the code files for the resource type package\. This template is based on the permissions specified for each handler in the `[handlers](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/resource-type-schema.html#schema-properties-handlers)` section of the [Resource type schema](https://docs.aws.amazon.com/cloudformation-cli/latest/userguide/resource-type-schema.html)\. When you use `submit` to register the resource type, the CloudFormation CLI attempts to create or update an execution role based on the template, and then passes this execution role to CloudFormation as part of the registration\.

For more information on the permissions available per AWS service, see [Actions, Resources, and Condition Keys for AWS Services](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html) in the *[AWS Identity and Access Management User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)*\.

## Monitoring runtime logging for resource types<a name="resource-type-develop-log"></a>

When you register a resource type using `cfn submit`, CloudFormation creates a CloudWatch log group for the resource type in your account\. This enables you to access the logs for your resource to help you diagnose any faults\. The log group is named according to the following pattern:

`/my-resource-type-stack-ResourceHandler-string`

Where:
+ *my\-resource\-type* is the three\-part resource type name
+ *string* is a unique string generated by CloudFormation

Now, when you execute stack operations for stacks that contain the resource type, CloudFormation delivers log events emitted by the resource type to this log group\.