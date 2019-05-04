---
title: "Bringing some more speed to cloudformation"
date: 2019-05-04T08:29:06-04:00
draft: false
---

After working extensively with AWS Cloudformation the last couple of months, I have noticed that template debugging and creation is not overly fast. To help ease development burden, I have created [cloudspeed](https://github.com/christopherdiehl/cloudspeed) which aims to bring some more speed to cloudformation by introducing automatic creation/deletion of templates in addition to event traces on creation failure. 

To help get started with cloudspeed, I have provided a high level overview of cloudformation and infrastructure as code to outline what the service does and why it is helpful. At the end, I'll outline what services exist to help validate templates and how cloudspeed fits in.

# Cloudformation

## What is CloudFormation

AWS documentation describes CloudFormation as:

> AWS CloudFormation provides a common language for you to describe and provision all the infrastructure resources in your cloud environment. CloudFormation allows you to use a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This file serves as the single source of truth for your cloud environment. [Source](https://aws.amazon.com/cloudformation/)

## Why CloudFormation/ Infrastructure as Code?

- Automation

  - Pros:
    - Ability to automatically redeploy infrastructure as changed.
    - Only need to change a few files. Depending on file type, could be vendor agnostic.
    - If infrastructure fails, can easily rebuild using stored code.
    - Ability to "roll-back" infrastructure
  - Cons:
    - If infrastructure starts to "drift", renders the infrastructure as code problematic and can lead to regressions.
    - "drift" is when an admin make manual modifications to infrastructure rather than modifying the code

- Source Control
  - Pros:
    - the ability to track changes to infrastructure via source code.
    - Additionally ability to edit infrastructure using a simple text editor.
  - Cons:
    - lack of UI which might have quality of life features

## CloudFormation

- Define infrastructure using JSON or YAML
- Custom cloudformation language types, [SAM](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md)
- Preview changes via [change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- Parameters provide a secure way to dynamically pass in environment variables

## Validating Templates

Templates can be amazing, but only when they are valid. If a template isn't usable by AWS, then it's pointless. Thankfully, AWS provides a way to validate template files using the `aws` cli tool's [validate-template](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/validate-template.html).

Unfortunately, `validate-template` only ensures the file is valid JSON or YAML. The only way I have found to ensure the template has valid template properties is to attempt to create the stack and debug via error messages, manually creating and deleting the stack each time. There is also an interesting stackoverflow [discussion](https://stackoverflow.com/questions/11854772/how-can-i-quickly-and-effectively-debug-cloudformation-templates) around this topic. In early 2018 AWS also released [cfn-python-lint](https://github.com/aws-cloudformation/cfn-python-lint) which attempts to provide validation for both CloudFormation template properties and values, via custom rules. Due to how quickly CloudFormation values and properties are updated, these rules can be out of date. Specifically, I've found the tool was unable to completely validate/lint parts of the new AWS Serverless template. To provide another option, I created [cloudspeed](https://github.com/christopherdiehl/cloudspeed) which spools up the infrastructure, returns any error messages, then deletes the stack. Both libraries have their specific use cases, and I would recommend using *cfn-lint* whenever possible to catch major mistakes and following up with *cloudspeed* to validate and iterate over the template to ensure it creates the infrastructure you desire.

## Helpful Links

- [Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
- [Infrastructure as Code](https://devops.stackexchange.com/questions/550/what-is-infrastructure-as-code)
- [Immutable Infrastructure](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure)
- [Validate Template](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/validate-template.html)
