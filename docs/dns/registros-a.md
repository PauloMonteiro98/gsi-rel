O `samba-tool` é utilizado para gerenciar vários aspectos de um servidor Samba, incluindo registros DNS. O comando `samba-tool dns` permite manipular registros DNS dentro de uma zona gerenciada pelo Samba como controlador de domínio (AD).

Os registros do tipo A associam um nome de host a um endereço IP IPv4. No exemplo a seguir, vamos simular a criação de 4 registros de DNS do tipo A, com nomes de cidades mineiras pouco conhecidas, e listar todos os registros da zona.

### Passo a Passo

#### Comando para criar registros DNS do tipo A:

```bash
samba-tool dns add <servidor-dns> <zona> <nome> A <ip> -U <usuário>
```

- `<servidor-dns>`: IP ou nome do servidor DNS (geralmente `127.0.0.1` para o servidor local).
- `<zona>`: Nome da zona DNS (exemplo: `minasgerais.lab`).
- `<nome>`: Nome do host que você quer associar a um IP (exemplo: uma cidade).
- `A`: Especifica que o tipo de registro é A (IPv4).
- `<ip>`: O endereço IP associado ao nome.
- `-U <usuário>`: Usuário autorizado (geralmente `administrator`).

#### Comando para listar registros DNS da zona:

```bash
samba-tool dns query 127.0.0.1 "minasgerais.lab" @ ALL -U administrator
```

Esse comando consulta todos os registros da zona `minasgerais.lab`.

### Exemplo de Simulação no Terminal

Aqui está uma simulação com cidades pouco conhecidas de Minas Gerais (como **Itamarandiba**, **Passa Vinte**, **Aracitaba**, **Caputira**) e seus IPs fictícios.

#### 1. Criar o registro DNS para Itamarandiba

```bash
$ samba-tool dns add 127.0.0.1 minasgerais.lab itamarandiba A 192.168.10.10 -U administrator
Password for [administrator]:
Record added successfully
```

#### 2. Criar o registro DNS para Passa Vinte

```bash
$ samba-tool dns add 127.0.0.1 minasgerais.lab passavinte A 192.168.10.20 -U administrator
Password for [administrator]:
Record added successfully
```

#### 3. Criar o registro DNS para Aracitaba

```bash
$ samba-tool dns add 127.0.0.1 minasgerais.lab aracitaba A 192.168.10.30 -U administrator
Password for [administrator]:
Record added successfully
```

#### 4. Criar o registro DNS para Caputira

```bash
$ samba-tool dns add 127.0.0.1 minasgerais.lab caputira A 192.168.10.40 -U administrator
Password for [administrator]:
Record added successfully
```

### Listar todos os registros da zona `minasgerais.lab`

Agora, para listar todos os registros da zona `minasgerais.lab`, usamos o seguinte comando:

```bash
$ samba-tool dns query 127.0.0.1 "minasgerais.lab" @ ALL -U administrator
Password for [administrator]:
  Name=, Records=0, Children=4
    A: itamarandiba, Address=192.168.10.10, Flags=600000f0 [DOMAIN_ENTRY, CACHE_ENTRY]
    A: passavinte, Address=192.168.10.20, Flags=600000f0 [DOMAIN_ENTRY, CACHE_ENTRY]
    A: aracitaba, Address=192.168.10.30, Flags=600000f0 [DOMAIN_ENTRY, CACHE_ENTRY]
    A: caputira, Address=192.168.10.40, Flags=600000f0 [DOMAIN_ENTRY, CACHE_ENTRY]
```

### Conclusão

Simulamos a criação de quatro registros DNS do tipo A utilizando o `samba-tool` e, em seguida, listamos todos os registros na zona `minasgerais.lab`. Esse processo pode ser repetido para adicionar mais registros conforme necessário no seu ambiente de DNS controlado pelo Samba.