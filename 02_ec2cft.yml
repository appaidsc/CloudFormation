AWSTemplateFormatVersion: '2010-09-09'
Description: Create an EC2 instance with a specific name

Parameters:
  InstanceName:
    Type: String
    Description: Name for the EC2 instance

Resources:
  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c7217cdde317cfec  # Amazon Linux 2 AMI (us-east-1)
      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      Tags:
        - Key: Name
          Value: !Ref InstanceName

Outputs:
  InstanceID:
    Description: The Instance ID
    Value: !Ref EC2Instance

  PublicIP:
    Description: Public IP of the Instance
    Value: !GetAtt EC2Instance.PublicIp
