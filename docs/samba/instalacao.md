# Configurando o Samba como Controlador de Domínio no Alpine Linux

Este guia cobre a instalação e configuração do Samba para atuar como um controlador de domínio do Active Directory no Alpine Linux.

## Requisitos

- Alpine Linux instalado e atualizado.
- Acesso root ou privilégios `sudo`.
- Conectividade de rede adequada.

## Passo 1: Atualizar os pacotes

Antes de iniciar, atualize os pacotes do sistema:

```sh
apk update
apk upgrade
```

## Passo 2: Instalar o Samba e Dependências

Instale o Samba e as dependências necessárias para configurá-lo como controlador de domínio do AD:

```sh
apk add samba samba-dc samba-winbind krb5 krb5-server krb5-libs krb5-client openldap-clients
```

## Passo 3: Configurar o Kerberos

O Samba como controlador de domínio AD requer a configuração do Kerberos. Edite o arquivo `/etc/krb5.conf`:

```ini
[libdefaults]
    default_realm = EXEMPLO.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    EXEMPLO.COM = {
        kdc = dc.exemplo.com
        admin_server = dc.exemplo.com
    }

[domain_realm]
    .exemplo.com = EXEMPLO.COM
    exemplo.com = EXEMPLO.COM
```

Substitua `EXEMPLO.COM` pelo nome do seu domínio.

## Passo 4: Provisionar o Controlador de Domínio

Antes de iniciar o Samba como um controlador de domínio, você precisa provisioná-lo com o comando `samba-tool`. Isso configura o Samba como um Controlador de Domínio (DC) do AD. Execute o seguinte comando:

```sh
samba-tool domain provision --use-rfc2307 --interactive
```

Durante o provisionamento, você será solicitado a fornecer várias informações:

- **Realm**: O nome do domínio, como `EXEMPLO.COM`.
- **Domain**: O nome do domínio, como `EXEMPLO`.
- **Administrator password**: Defina a senha para o usuário `Administrator`.

Se preferir executar o provisionamento sem interação, você pode fornecer todas as opções de uma vez:

```sh
samba-tool domain provision \
  --realm=EXEMPLO.COM \
  --domain=EXEMPLO \
  --adminpass=SuaSenhaForte \
  --server-role=dc \
  --use-rfc2307
```

### Explicação das opções:
- `--realm`: O nome completo do domínio no formato `EXEMPLO.COM`.
- `--domain`: O nome curto do domínio.
- `--adminpass`: A senha para o administrador do domínio.
- `--server-role`: Define o Samba como controlador de domínio (`dc`).
- `--use-rfc2307`: Ativa o suporte para esquemas Unix em AD (RFC2307).

## Passo 5: Configurar DNS

O Samba pode atuar como servidor DNS. Para habilitar o DNS, edite o arquivo `/etc/samba/smb.conf`, que foi gerado automaticamente durante o provisionamento. O arquivo deve conter as seguintes configurações:

```ini
[global]
    workgroup = EXEMPLO
    realm = EXEMPLO.COM
    netbios name = DC1
    server role = active directory domain controller
    dns forwarder = 8.8.8.8

[sysvol]
    path = /var/lib/samba/sysvol
    read only = no

[netlogon]
    path = /var/lib/samba/sysvol/exemplo.com/scripts
    read only = no
```

A opção `dns forwarder` define o encaminhador de DNS. No exemplo, estamos utilizando o DNS público do Google (`8.8.8.8`), mas você pode substituir por outro de sua escolha.

## Passo 6: Iniciar o Samba

Agora, inicie o Samba como controlador de domínio:

```sh
rc-service samba start
```

Para garantir que o Samba e os serviços relacionados sejam iniciados automaticamente na inicialização do sistema:

```sh
rc-update add samba
```

## Passo 7: Testar a Configuração do Samba

Verifique se o Samba está funcionando corretamente:

```sh
samba-tool dns query localhost exemplo.com @ ALL
```

Esse comando deve retornar todas as zonas DNS configuradas.

Também é uma boa prática verificar se os serviços `smbd`, `nmbd` e `winbind` estão rodando:

```sh
rc-service samba status
```

## Passo 8: Verificar o DNS

Verifique se o servidor DNS está funcionando corretamente. Você pode testar a resolução de nomes com o `nslookup`:

```sh
nslookup dc1.exemplo.com
```

## Passo 9: Configurar as Regras de Firewall (Opcional)

Se você estiver usando um firewall, certifique-se de que as portas usadas pelo Samba estão abertas. As principais portas são:

- **TCP/UDP 53**: DNS
- **TCP/UDP 88**: Kerberos
- **TCP/UDP 135**: RPC
- **TCP/UDP 139 e 445**: SMB
- **UDP 389**: LDAP
- **TCP/UDP 464**: Kerberos password changes
- **TCP 636**: LDAP sobre SSL (LDAPS)

## Passo 10: Conectar um Cliente ao Controlador de Domínio

Agora que o Samba está configurado como controlador de domínio, você pode conectar um cliente Windows ao domínio. Siga estas etapas em um cliente Windows:

1. Abra o "Painel de Controle".
2. Vá em "Sistema e Segurança" > "Sistema".
3. Clique em "Alterar configurações" em "Nome do computador, domínio e configurações de grupo de trabalho".
4. Clique em "Alterar".
5. Selecione "Domínio" e insira o nome do domínio (por exemplo, `EXEMPLO`).
6. Insira as credenciais do administrador do Samba quando solicitado.

## Conclusão

Você configurou o Samba no Alpine Linux como um controlador de domínio do Active Directory. A partir daqui, você pode gerenciar seu domínio, adicionar máquinas e configurar permissões conforme necessário.