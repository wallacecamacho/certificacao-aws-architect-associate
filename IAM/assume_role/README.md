# Como assumo uma função do IAM usando a CLI da AWS?

_Última atualização: 2019-05-07_

Desejo assumir uma função do Amazon Identity and Access Management (IAM) usando a AWS Command Line Interface (AWS CLI). Como posso fazer isso?

## Artefatos
[**Arquivos de exemplos das configurações**](https://github.com/wallacecamacho/certificacao-aws-architect-associate/tree/master/IAM/assume_role/exemplos).

## Resolução

Siga estas instruções para assumir uma função do IAM usando a CLI da AWS. Neste exemplo, o usuário terá acesso somente leitura às instâncias do Amazon Elastic Compute Cloud (Amazon EC2) e permissão para assumir uma função do IAM.

**Crie um usuário do IAM que tenha permissões para assumir funções**

1. [Crie um usuário do IAM usando a CLI da AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_cliwpsapi):

**Nota: **Substitua** Bob** pelo seu nome de usuário do IAM.

```
aws iam create-user --user-name Bob
```

1. Crie a política do IAM que concede as permissões a **Bob** usando a AWS CLI. Você deve criar o arquivo JSON que define a política do IAM usando seu editor de texto favorito. Este exemplo usa o vim, que é comumente usado no Linux:

**Nota: **Substitua** example** por seu próprio nome da política, nome do usuário, função, nome do arquivo JSON, nome do perfil e chaves.

```
vim example-policy.json
```

1. O conteúdo do arquivo **example-policy.json** deve ser semelhante a este:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "iam:ListRoles",
                "sts:AssumeRole"
            ],
            "Resource": "*"
        }
    ]
}
```

Para obter mais informações sobre como criar políticas do IAM, consulte [Criando políticas do IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html), [Exemplo de políticas baseadas em identidade do IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html) e [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) .

**Crie a política do IAM**

1. Use o comando **aws iam create-policy**:

```
aws iam create-policy --policy-name example-policy --policy-document file://example-policy.json
```

O comando aws iam create-policy gera várias informações, incluindo o ARN (Amazon Resource Name) da política do IAM:

```
arn:aws:iam::123456789012:policy/example-policy
```

**Nota: **Substitua** 123456789012** por sua própria conta

1. Anote a ARN da política do IAM na saída e anexe a política a **Bob** usando o comando **attach-user-policy**. Verifique se o anexo está no lugar usando **list-attached-user-policies**:

```
aws iam attach-user-policy --user-name Bob --policy-arn "arn:aws:iam::123456789012:policy/example-policy"
aws iam list-attached-user-policies --user-name Bob
```

**Crie o arquivo JSON que define a relação de confiança da função IAM**

1. Crie o arquivo JSON que define o relacionamento de confiança:

```
vim example-role-trust-policy.json
```

2. O conteúdo do arquivo example-role-trust-policy.json deve ser semelhante a este:

```
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Principal": { "AWS": "arn:aws:iam::123456789012:root" },
        "Action": "sts:AssumeRole"
    }
}
```

Você também pode restringir a relação de confiança para que a função do IAM possa ser assumida apenas por usuários específicos do IAM. Você pode fazer isso especificando entidades semelhantes a **arn:aws:iam::123456789012:user/exemplo-nome-de-usuário**. Para obter mais informações, consulte [AWS JSON Policy Elements: Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html).

**Crie a função do IAM e anexe a política**

Crie uma função do IAM que possa ser assumida por **Bob** que tenha acesso somente leitura às instâncias do Amazon Relational Database Service (Amazon RDS). Como essa função do IAM é assumida por um usuário do IAM, você deve especificar uma entidade que permita que os usuários do IAM assumam essa função. Por exemplo, um princípio semelhante a **arn:aws:iam::123456789012:root** permite que todas as identidades do IAM da conta assumam essa função. Para obter mais informações, consulte [Criando uma função para delegar permissões a um usuário do IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html).

1. Crie a função IAM que tenha acesso somente leitura às instâncias do Amazon RDS DB. Anexe as políticas do IAM à sua função do IAM de acordo com seus requisitos de segurança.

O comando **aws iam create-role** cria a função IAM e define o relacionamento de confiança de acordo com o conteúdo do arquivo JSON. O comando **aws iam attach-role-policy** anexa a Política gerenciada da AWS **AmazonRDSReadOnlyAccess** à função. Você pode anexar políticas diferentes (políticas gerenciadas e políticas personalizadas) de acordo com seus requisitos de segurança. O comando **aws iam list-attached-role-policy** amostra as políticas do IAM anexadas à função do IAM **example-role**.

```
aws iam create-role --role-name example-role --assume-role-policy-document file://example-role-trust-policy.json
aws iam attach-role-policy --role-name example-role --policy-arn "arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess"
aws iam list-attached-role-policies --role-name example-role
```

2. Marque **Bob** para verificar o acesso somente leitura às instâncias do EC2 e se ele pode assumir a **função de exemplo**. Crie chaves de acesso para **Bob** com este comando:

```
aws iam create-access-key --user-name Bob
```

O comando da AWS CLI gera um ID da chave de acesso e uma chave de acesso secreta - observe essas chaves. Pode salvar o AccessKeyId e o SecretAccessKey para uso em outros exemplos.

**Configure as chaves de acesso**

1. Para configurar as teclas de acesso, use o perfil padrão ou um perfil específico. Para configurar o perfil padrão, execute **aws configure**. Para criar um novo perfil específico, execute:
   
``` 
aws configure --profile test
```
2. Neste exemplo, o perfil padrão está configurado:

```
aws configure
ID da chave de acesso da AWS [Nenhum]:ExampleAccessKeyID1
Chave de acesso secreto da AWS [Nenhuma]:ExampleSecretKey1
Nome da região padrão [Nenhum]:eu-west-1
Formato de saída padrão [Nenhum]:json
```

**Nota:** Para **Nome da região padrão**, especifique sua [Região da AWS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html).

**Verifique se seus comandos da AWS CLI são chamados e o acesso do usuário do IAM**

1. execute o comando **aws sts get-caller-identity**:

```
aws sts get-caller-identity
```

O comando **aws sts get-caller-identity** gera três informações, incluindo o ARN. Ele deve mostrar algo semelhante a **arn:aws:iam::123456789012:user/Bob**, que verifica se os comandos da AWS CLI são chamados como **Bob**.

1. Confirme se o usuário do IAM possui acesso somente leitura às instâncias do EC2 e nenhum acesso às instâncias do Amazon RDS DB executando estes comandos:

```
aws ec2 describe-instances --query "Reservations[*].Instances[*].[VpcId, InstanceId, ImageId, InstanceType]"
aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier, DBName, DBInstanceStatus, AvailabilityZone, DBInstanceClass]"
```

O comando **aws ec2 describe-instance** deve mostrar todas as instâncias do EC2 que estão na região eu-west-1. O comando **aws rds describe-db-instances** deve gerar uma mensagem de erro de acesso negado, porque **Bob** não tem acesso ao Amazon RDS.

**Assuma a função do IAM**

1. Obtenha o ARN da função executando este comando:

```
aws iam list-roles --query "Roles[?RoleName == 'example-role'].[RoleName, Arn]"
```

2. O comando lista funções do IAM, mas filtra a saída pelo nome da função. Para assumir a função do IAM, execute este comando:

```
aws sts assume-role --role-arn "arn:aws:iam::123456789012:role/example-role" --role-session-name AWSCLI-Session
```

O comando da CLI da AWS gera várias informações. Dentro do bloco de credenciais, você precisa de **AccessKeyId**, **SecretAccessKey** e **SessionToken**. Anote o registro de data e hora do campo de expiração. Está no fuso horário UTC e indica quando as credenciais temporárias da função IAM expiram. Se as credenciais temporárias expirarem, você deverá chamar a chamada da API **sts:AssumeRole** novamente.

**Nota:** Você pode aumentar a expiração máxima da duração da sessão para credenciais temporárias para funções do IAM usando o [DurationSeconds](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html#API_AssumeRoleWithSAML.html#API_AssumeRoleWithSAML_RequestParameters) parâmetro.

**Crie variáveis ​​de ambiente para assumir a função do IAM e verificar o acesso**

1. Crie três variáveis ​​de ambiente para assumir a função do IAM. Essas variáveis ​​de ambiente são preenchidas com esta saída:
```
export AWS_ACCESS_KEY_ID=ExampleAccessKeyID1
export AWS_SECRET_ACCESS_KEY=ExampleSecretKey1
export AWS_SESSION_TOKEN=ExampleSessionToken1
```
2. Compare as informações geradas digite o comando:
   
```
env | grep AWS
```

3. Verifique se você assumiu a função do IAM executando este comando:

```
aws sts get-caller-identity
```

O comando da CLI da AWS deve gerar o ARN como **arn:aws:sts::123456789012:papel-assumido/exemplo-papel/AWSCLI-Session** em vez de **arn:aws:iam::123456789012:user/Bob**, que verifica se você assumiu o **exemplo-papel**.

```
{
    "Arn": "arn:aws:sts::12345678912:assumed-role/example-role/AWSCLI-Session",
    "Account": "12345678912",
    "UserId": "AROAZMJJDJUPBDSA2V7WI:AWSCLI-Session"
}
```

3. Você criou uma função do IAM com acesso somente leitura às instâncias do Amazon RDS DB, mas sem acesso às instâncias do EC2. Verifique executando estes comandos:

```
aws ec2 describe-instances --query "Reservations[*].Instances[*].[VpcId InstanceId, ImageId, InstanceType]"
```

```
aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier, DBName, DBInstanceStatus, AvailabilityZone, DBInstanceClass]"
```

O comando **aws ec2 describe-instances** deve gerar uma mensagem de erro de acesso negado e o comando **aws describe-db-instances** deve retornar as instâncias do Amazon RDS DB. Isso verifica se as permissões atribuídas à função do IAM estão funcionando corretamente.

4. Para retornar ao usuário do IAM, remova as variáveis ​​de ambiente:

```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
aws sts get-caller-identity
```

O comando **unset **remove as variáveis ​​de ambiente e o comando** aws sts get-caller-identity** verifica se você retornou como **Bob**.

Você também pode usar uma função criando um perfil no arquivo **~/.aws/config**. Para obter mais informações, consulte [Assumindo uma função do IAM na AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html).

