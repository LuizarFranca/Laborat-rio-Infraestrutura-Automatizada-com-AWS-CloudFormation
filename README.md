# Laborat-rio-Infraestrutura-Automatizada-com-AWS-CloudFormation
Implementação de uma infraestrutura automatizada usando AWS CloudFormation

Objetivo
Aprender a criar e gerenciar infraestrutura como código (IaC) na AWS utilizando CloudFormation, implementando recursos básicos (VPC, sub-redes, EC2, Security Groups) de forma automatizada e reprodutível.

Pré-requisitos
  -Conta AWS ativa (preferencialmente conta de estudos ou sandbox).
   - Conhecimento básico sobre:
    -Conceitos de rede (VPC, sub-redes, IPs, rotas).
    -EC2 (instâncias, tipos, pares de chaves).
    -CloudFormation (conceito de templates YAML/JSON).
 -AWS CLI instalada e configurada.
 -Editor de texto (VS Code ou Cloud9).

Visão geral do laboratório
Recursos a serem criados
 1. template CloudFormation irá provisionar:
 1.VPC com CIDR 10.0.0.0/16
 2.Sub-rede pública 10.0.1.0/24
 3.Internet Gateway e tabela de rotas configurada
 4.Security Group permitindo acesso SSH (porta 22) e HTTP (porta 80)
 5.Instância EC2 (Amazon Linux 2) acessível via Internet
 6.Outputs com:
  -IP público da instância
  -ID da VPC


Passo a Passo
#1.Criar o Template CloudFormation
Crie um arquivo chamado infra-automatica.yaml com o conteúdo abaixo:

AWSTemplateFormatVersion: '2010-09-09'
Description: Infraestrutura Automatizada - Exemplo de Laboratório CloudFormation

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: LabVPC

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: PublicSubnet

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: LabIGW

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  DefaultRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permite SSH e HTTP
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: LabSG

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0 # Amazon Linux 2 - ajustar conforme região
      InstanceType: t2.micro
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds: [!Ref SecurityGroup]
      KeyName: MyKeyPair
      Tags:
        - Key: Name
          Value: LabInstance

Outputs:
  InstancePublicIP:
    Description: IP público da instância
    Value: !GetAtt EC2Instance.PublicIp
  VPCId:
    Description: ID da VPC criada
    Value: !Ref VPC


#2.Implantar o Stack
No terminal (com AWS CLI configurada):

aws cloudformation create-stack \
  --stack-name LabCloudFormation \
  --template-body file://infra-automatica.yaml \
  --capabilities CAPABILITY_IAM

Acompanhe o progresso pelo Console AWS → CloudFormation → Stacks.

#3. Validar os recursos
Após o status CREATE_COMPLETE:
  -Acesse o EC2 Dashboard → veja a instância em execução.
  -Verifique os Outputs da Stack para encontrar o IP público.
  -Teste o acesso via SSH:
  
  bash
  ssh ec2-user@<IP_PUBLICO>

- (Opcional) Instale um servidor web:
  
bash
 sudo yum install httpd -y
sudo systemctl start httpd


#Desafio Extra

Modifique o template para incluir:
  -Uma sub-rede privada.
  -Uma instância adicional em rede privada com acesso via bastion host.
  -Um S3 Bucket para armazenar logs.

  
