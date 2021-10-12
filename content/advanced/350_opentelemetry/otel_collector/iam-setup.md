---
title: "IAM Setup"
date: 2021-5-01T09:00:00-00:00
weight: 41
draft: false
---

### Creating IAM permissions for OTEL Collector

Enable the IAM OIDC provider on our EKS Cluster
```bash
eksctl utils associate-iam-oidc-provider --region=$AWS_REGION \
    --cluster=$CLUSTER_NAME \
    --approve
export OIDC_PROVIDER=$(aws eks describe-cluster --name $CLUSTER_NAME --region=$AWS_REGION \
    --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///")
echo -e "OIDC_PROVIDER: $OIDC_PROVIDER"
```

Create IAM policy for the collector:
```bash
cat << EOF > AWSDistroOpenTelemetryPolicy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:PutLogEvents",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:DescribeLogStreams",
                "logs:DescribeLogGroups",
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords",
                "xray:GetSamplingRules",
                "xray:GetSamplingTargets",
                "xray:GetSamplingStatisticSummaries",
                "cloudwatch:PutMetricData",
                "ec2:DescribeVolumes",
                "ec2:DescribeTags",
                "ssm:GetParameters"
            ],
            "Resource": "*"
        }
    ]
}
EOF

aws iam create-policy --region=${AWS_REGION} \
    --policy-name AWSDistroOpenTelemetryPolicy \
    --policy-document file://AWSDistroOpenTelemetryPolicy.json

export ADOT_IAMPOLICY_ARN=$(aws iam list-policies --region=${AWS_REGION} | jq '.Policies | .[] | select(.PolicyName == "AWSDistroOpenTelemetryPolicy").Arn' --raw-output)
echo -e "Created ADOT IAM Policy: $ADOT_IAMPOLICY_ARN"
```

Create an IAM role for the collector:
```
read -r -d '' TRUST_RELATIONSHIP <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::${ACCOUNT_ID}:oidc-provider/${OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${OIDC_PROVIDER}:sub": "system:serviceaccount:aws-otel-eks:aws-otel-sa"
        }
      }
    }
  ]
}
EOF
echo "${TRUST_RELATIONSHIP}" > trust.json

aws iam create-role --role-name AWSDistroOpenTelemetryRole \
    --assume-role-policy-document file://trust.json \
    --description "IAM Role for ADOT"
```

Associate the IAM Policy (`AWSDistroOpenTelemetryPolicy`) with the IAM Role (`AWSDistroOpenTelemetryRole`):
```bash
aws iam attach-role-policy --role-name AWSDistroOpenTelemetryRole \
    --policy-arn=$ADOT_IAMPOLICY_ARN
```
