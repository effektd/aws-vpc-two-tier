# aws-vpc-two-tier

This is a standard two tier AWS VPC stack.

# Stack information

## Resources

Deploying this CloudFormation stack will provide you with;

- Two public subnets
- Two private subnets
- Internet Gateway
- NAT Gateway

## AWS Costs

The only real cost associated specifically to this CloudFormation stack is the NAT Gateway, please visit [this page](https://aws.amazon.com/vpc/pricing/) for more information on NAT Gateway pricing

## Deploying the stack

The below steps are to be followed when this is the first time deploying the stack;

- Open a terminal window on your local machine
- Ensure you have access to your AWS account and are currently set to your desired AWS region
- Navigate to this code repository
- Run the following command to deploy the CloudFormation stack
- `aws cloudformation create-stack --stack-name "<Insert-Stack-Name>" --template-body file://cfn-network-stack.yml`
    - Change `<Insert-Stack-Name>` to the name of which you wish to call this CloudFormation stack
- Wait for the stack to finish deployment
- You now have a two tier AWS VPC deployed

## Updating an existing stack

If you need to update the already deployed CloudFormation stack do the following;

- Open a terminal window on your local machine
- Ensure you have access to your AWS account and are currently set to your desired AWS region
- Navigate to this code repository
- Run the following command to update the CloudFormation stack
- `aws cloudformation update-stack --stack-name "<Insert-Stack-Name>" --template-body file://cfn-network-stack.yml`
    - Change `<Insert-Stack-Name>` to the name of which you called your CloudFormation stack when you first deployed it
- Wait for the stack to finish deployment
- You now have a two tier AWS VPC deployed

## Outputs

You can use the outputs provided by this CloudFormation stack in your other stacks.

For example you could use `!ImportValue "<Insert-Stack-Name>-PRIVATE-B` to retrieve the subnet id for the Private B subnet for use in a different CloudFormation stack.

# Customisation

You will likely want to customise this CloudFormation stack for your needs, the below will provide examples of some common adjustments.

## More subnets

If your region has more than two availability zones, you will likely want to create a public and private subnet in these additional zones.

Simply follow the alphabetical lettering for the following resources and use _odd_ octets for your public subnets and use _even_ octets for your private subnets

- Add the following in the `Parameters` section
```
  CIDRPublicC:
    Type: String 
    Default: 10.0.5.0/24
    Description: "The CIDR Range for this subnet"
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-8]))$

  CIDRPrivateC:
    Type: String 
    Default: 10.0.6.0/24
    Description: "The CIDR Range for this subnet"
    AllowedPattern: ^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\/(1[6-9]|2[0-8]))$
```

- Add the following in the `Resources` section
```
  PublicSubnetC:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref CIDRPublicC
      AvailabilityZone: !Select [ 2, !GetAZs ]
      Tags:
        - Key: Name
          Value: !Sub "${VPCName}-PUBLIC-C"

  PrivateSubnetC:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref CIDRPrivateC
      AvailabilityZone: !Select [ 2, !GetAZs ]
      Tags:
        - Key: Name
          Value: !Sub "${VPCName}-PRIVATE-C"

  PublicSubnetCRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnetC
      RouteTableId: !Ref PublicRouteTable

  PrivateSubnetCRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnetC
      RouteTableId: !Ref PrivateRouteTable
```
**NOTE:** Make sure you change the Availability zone select value in the `EC2::Subnet` resources to the next number up, as this example is creating a third subnet, we set `!Select [ 2, !GetAZs ]` which will retrieve the **third** availability zone.

- Add the following to the `Outputs` section
```
  PublicSubnetCID:
    Value: !Ref PublicSubnetC
    Export:
       Name: !Sub "${AWS::StackName}-PUBLIC-C"

  PrivateSubnetCID:
    Value: !Ref PrivateSubnetC
    Export:
       Name: !Sub "${AWS::StackName}-PRIVATE-C"
```

# Note from the author

You are welcome to use this code base as you see fit as per the [license](LICENSE).