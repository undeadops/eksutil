# Listing Pods Sample

## Building

Setting this up is rather easy. Make sure you are in the `lambda/listpods` directory, build the
function using the `make` command.

```
$ make
```

This creates a zip package of the function which can be deployed to AWS Lambda. 

## Configuration

There are few environment variables that will configure how this function works.

Variable Name | Description
--------------|------------
CLUSTER_NAME | Name of the Amazon EKS cluster.
ENV | (Optional) Setting this variable to `DEBUG` to enable debug log output.

You would also need to give the Lambda execution role permissions in Amazon EKS cluster. Refer to
this [User Guide](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html) for detailed
instructions.

1. Edit the `aws-auth` ConfigMap of your cluster.
```
$ kubectl -n kube-system edit configmap/aws-auth
```
2. Add your Lambda execution role to the config
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  mapRoles: |
    - rolearn: arn:aws:iam::<AWS Account ID>:role/devel-worker-nodes-NodeInstanceRole-74RF4UBDUKL6
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - rolearn: arn:aws:iam::<AWS Account ID>:role/<your lambda execution role>
      username: admin
      groups:
        - system:masters
```

For your Lambda execution role, you will need permissions to describe EKS cluster. Add the following
statement to the IAM role.

```
{
    "Effect": "Allow",
    "Action": [
        "eks:DescribeCluster"
    ],
    "Resource": "*"
}
```

You may want to be more restrictive by specifying only the arn of your EKS cluster for resource
field.

Once these are configured, you can test your function. Good luck!



