# Desafio 3: Primeira Stack com AWS CloudFormation - Santander Code Girls 2025

Este repositório contém meu **terceiro desafio do bootcamp Santander Code Girls 2025**, em parceria com a DIO.

O objetivo do desafio é aplicar na prática o uso do **AWS CloudFormation** para criar e gerenciar recursos de forma automatizada.

## Conceitos Fundamentais

### O que é AWS CloudFormation
- Serviço que **automatiza a criação, atualização e exclusão de recursos AWS** usando templates YAML ou JSON.
- Base do conceito **Infrastructure as Code (IaC)**: descreve a infraestrutura em arquivo e a AWS cria tudo automaticamente como uma **stack**.

### Template CloudFormation
- Arquivo YAML ou JSON que descreve recursos (EC2, S3, IAM, Security Groups, etc.), parâmetros, mapas, outputs e dependências.
- O CloudFormation lê o template e cria uma **stack** com os recursos definidos.

### Stack
- É a execução de um template: **conjunto de recursos** criados a partir daquele template.
- Pode ser criada, atualizada ou excluída de forma automática.

### Custos
- O uso do CloudFormation **não gera custos adicionais**, você paga apenas pelos recursos que a stack criar (EC2, S3, RDS, etc.).

## Exemplos de Templates

### Exemplo 1 — EC2 Simples
```yaml
Description: Criar um Amazon EC2 simples
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0ed9277fb7eb570c9
      InstanceType: t2.micro
      Tags:
        - Key: "Name"
          Value: "EC2"
```

### Exemplo 2 — EC2 com Apache
```yaml
Description: Instalar Servidor Apache
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0ed9277fb7eb570c9
      InstanceType: t2.micro
      Tags:
        - Key: "Name"
          Value: "Webserver-Apache"
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            yum install -y httpd.x86_64
            systemctl start httpd.service
            systemctl enable httpd.service
            echo "<h1>OLA AWS FOUNDATIONS do $(hostname -f)</h1>" > /var/www/html/index.html
```

### Exemplo 3 — Grupo de Segurança (Firewall)
```yaml
Description: Configurar Grupo de Segurança
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      ...
      SecurityGroups:
        - !Ref GrupoSeguranca

  GrupoSeguranca:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Acesso Liberado Porta 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
```

### Exemplo 4 — EC2, S3 e IAM
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation Template para criar EC2, IAM User e S3 Bucket

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro

Resources:
  S3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: S3-FOUNDATION

  IAMGroup:
    Type: 'AWS::IAM::Group'
    Properties:
      GroupName: GPO-ADMIN-LAB

  IAMUser:
    Type: 'AWS::IAM::User'
    Properties:
      UserName: alexsandro.lechner
      Groups:
        - !Ref IAMGroup

  EC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !FindInMap [UbuntuMap, !Ref "AWS::Region", UbuntuAMI]
      KeyName: your-key-pair-name
      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          apt-get update
          apt-get install -y python3-pip

  EC2SecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Habilitar acesso SSH
      VpcId: vpc-040a4ffd0374c4cf3
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0

Mappings:
  UbuntuMap:
    us-east-1:
      UbuntuAMI: ami-0c55b159cbfafe1f0
    us-west-2:
      UbuntuAMI: ami-0dba2cbf4e3c2e7b1

Outputs:
  InstanceId:
    Description: ID da instância EC2 criada
    Value: !Ref EC2Instance
  S3BucketName:
    Description: Nome do bucket S3 criado
    Value: !Ref S3Bucket
  IAMUserName:
    Description: Nome do usuário IAM criado
    Value: !Ref IAMUser
```
