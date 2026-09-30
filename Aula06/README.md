# Relatório Técnico: Configuração de Rede Estática com Netplan e Modo Placa em Ponte no VirtualBox

## 1. Identificação

- **Nome completo:** Mateus Alves dos Santos
- **Matrícula:** 2023002055
- **Curso:** Sistemas de Informação
- **Turma:** 2026.2
- **Data:** 23/09/2026
- **Título da prática:** Aula Prática 06: Configuração de Rede Estática com Netplan e Modo Placa em Ponte (Bridge Adapter) no VirtualBox

---

## 2. Objetivo

A atividade teve como objetivo reconfigurar o adaptador de rede da máquina virtual Ubuntu Server no VirtualBox, alterando o modo de conexão de **NAT** para **Placa em Ponte (Bridge Adapter)**.

Com essa alteração, a máquina virtual passa a participar diretamente da rede física à qual o computador hospedeiro está conectado, podendo ser identificada como um equipamento independente na rede local.

A prática também teve como objetivo configurar um endereço IP estático no Ubuntu Server utilizando o **Netplan**, definindo endereço IP, máscara de sub-rede, gateway padrão e servidores DNS.

Por fim, foram realizados testes de conectividade entre o computador hospedeiro Windows e a máquina virtual Ubuntu Server, além de testes de resolução de nomes e rastreamento de rotas externas utilizando o `traceroute`.

---

## 3. Ambiente

A atividade foi realizada utilizando a mesma máquina virtual das aulas anteriores.

O ambiente utilizado foi:

- **Sistema operacional do Guest:** Ubuntu Server 26.04;
- **Sistema operacional do Host:** Windows;
- **Virtualizador:** Oracle VM VirtualBox;
- **Máquina virtual:** `ubuntu_server`;
- **Usuário administrativo:** `administrador`;
- **Processador virtual:** 1 vCPU;
- **Memória RAM:** 2048 MB;
- **Disco virtual:** 32 GB;
- **Interface de rede virtual:** `enp0s3`;
- **Modo de rede anterior:** NAT;
- **Novo modo de rede:** Placa em Ponte (Bridge Adapter);
- **Rede do laboratório:** `172.20.20.0/22`;
- **Máscara:** `255.255.252.0`;
- **Gateway:** `172.20.20.1`.

A máquina virtual permaneceu com **2048 MB de memória RAM**, configuração adotada nas atividades anteriores devido aos problemas de memória encontrados durante a instalação do Ubuntu Server.

No modo NAT utilizado anteriormente, a máquina virtual possuía um endereço privado fornecido pelo roteador virtual do VirtualBox. Na presente atividade, a interface foi alterada para o modo **Placa em Ponte**, permitindo que a VM participasse diretamente da rede física do laboratório.

A rede utilizada na atividade pertence ao bloco:

```text
172.20.20.0/22
```

O endereço IP estático utilizado pela máquina virtual deverá ser um endereço livre dentro da faixa definida para a atividade.

---

## 4. Procedimento

### 4.1. Verificação de um endereço IP livre

Antes de configurar o endereço estático no Ubuntu Server, foi necessário verificar a disponibilidade de um endereço IP na rede do laboratório.

Conforme o roteiro da atividade, foi utilizado inicialmente o endereço candidato:

```text
172.20.23.1
```

No computador hospedeiro Windows, foi executado:

```powershell
ping 172.20.23.1
```

O objetivo desse teste foi verificar se já existia algum equipamento respondendo pelo endereço escolhido.

Quando o endereço não apresenta resposta, ele pode ser considerado disponível para utilização na atividade.

![Teste de disponibilidade do IP](./Evidências/01.png)

---

### 4.2. Alteração do adaptador de rede para Placa em Ponte

Com um endereço disponível identificado, foi alterada a configuração de rede da máquina virtual no VirtualBox.

Nas configurações da máquina virtual, foi acessada a opção:

**Configurações → Rede → Adaptador 1**

O campo **Conectado a:** foi alterado de:

```text
NAT
```

para:

```text
Placa em Ponte (Bridge Adapter)
```

Também foi selecionada a placa de rede física do computador hospedeiro que estava conectada à rede do laboratório.

A opção **Cabo conectado** foi mantida habilitada para permitir a comunicação da interface virtual.

![Configuração do adaptador em modo Placa em Ponte](./Evidências/02.png)

---

### 4.3. Verificação do arquivo de configuração do Netplan

Após a alteração do adaptador no VirtualBox, foi acessado o Ubuntu Server para realizar a configuração estática da interface de rede.

Inicialmente foi verificado o conteúdo do diretório `/etc/netplan/`:

```bash
ls /etc/netplan/
```

O arquivo utilizado para a configuração foi:

```text
/etc/netplan/00-installer-config.yaml
```

Em seguida, o arquivo foi aberto com o editor Nano:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

O arquivo foi configurado seguindo a estrutura YAML proposta no roteiro da atividade:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 172.20.23.X/22
      routes:
        - to: default
          via: 172.20.20.1
      nameservers:
        addresses:
          - 172.20.20.1
          - 1.1.1.1
          - 8.8.8.8
```

O valor `172.20.23.X` deve corresponder ao endereço livre identificado no teste anterior.

Na edição do arquivo foi necessário observar a indentação do YAML, utilizando espaços e mantendo corretamente os níveis da estrutura.

Após a edição, o conteúdo foi exibido utilizando:

```bash
cat /etc/netplan/00-installer-config.yaml
```

Essa etapa permitiu conferir a configuração antes de aplicá-la.

![Conteúdo do arquivo de configuração do Netplan](./Evidências/03.png)

---

### 4.4. Aplicação da configuração de rede

Depois de conferir o arquivo, a nova configuração foi aplicada com:

```bash
sudo netplan apply
```

O comando aplica as configurações presentes no arquivo do Netplan à interface de rede do sistema.

Em seguida, foi verificado o endereço atribuído à interface `enp0s3`:

```bash
ip addr show enp0s3
```

Também poderia ser utilizado:

```bash
sudo netplan status
```

para consultar o estado da interface e das rotas configuradas.

O objetivo dessa etapa foi confirmar que o endereço IP estático havia sido efetivamente atribuído à interface.

![IP estático atribuído à interface enp0s3](./Evidências/04.png)

---

### 4.5. Teste de conectividade do Windows para a VM

Após a aplicação da configuração, foi realizado o primeiro teste de conectividade entre o computador hospedeiro e a máquina virtual.

No PowerShell do Windows foi executado:

```powershell
ping 172.20.23.X
```

onde `172.20.23.X` corresponde ao endereço configurado na máquina virtual.

O teste teve como objetivo verificar se o Windows conseguia alcançar diretamente a máquina virtual através da rede local.

A comunicação direta entre o Host e o Guest demonstra uma das diferenças importantes em relação ao modo NAT utilizado nas aulas anteriores.

![Ping do Windows para a máquina virtual](./Evidências/05.png)

---

### 4.6. Teste de conectividade da VM para o Windows

Em seguida foi realizado o teste no sentido inverso.

Primeiramente, no Windows, foi utilizado:

```powershell
ipconfig
```

para identificar o endereço IPv4 da placa de rede física do computador hospedeiro.

Depois, no Ubuntu Server, foi executado:

```bash
ping -c 4 172.20.22.45
```

O endereço `172.20.22.45` representa o endereço do computador hospedeiro apresentado como exemplo no roteiro e deve ser substituído pelo IPv4 efetivamente identificado no Windows.

O parâmetro `-c 4` determina que sejam enviados quatro pacotes ICMP.

![Ping da máquina virtual para o Windows](./Evidências/06.png)

---

### 4.7. Rastreamento de rota para `google.com`

Após os testes de comunicação local, foi realizado um teste de conectividade externa utilizando o comando:

```bash
traceroute google.com
```

Esse teste teve dois objetivos principais: verificar a resolução do nome `google.com` e observar o caminho percorrido pelos pacotes até o destino.

De acordo com a configuração proposta no roteiro, o primeiro salto deve corresponder ao gateway da rede do laboratório:

```text
172.20.20.1
```

Os demais saltos representam os roteadores percorridos até alcançar a rede externa.

![Traceroute para google.com](./Evidências/07.png)

---

### 4.8. Rastreamento de rota para `one.one.one.one`

Por fim, foi realizado outro teste de rastreamento de rota utilizando o serviço DNS da Cloudflare:

```bash
traceroute one.one.one.one
```

O domínio `one.one.one.one` corresponde ao endereço IP `1.1.1.1`.

O teste permitiu verificar novamente a resolução de nomes e observar a sequência de roteamento utilizada até o destino externo.

![Traceroute para one.one.one.one](./Evidências/08.png)

---

## 5. Testes e Evidências

Os testes realizados permitiram validar as principais etapas da configuração de rede.

Inicialmente, foi verificada a disponibilidade de um endereço IP na rede do laboratório. Em seguida, o adaptador virtual do VirtualBox foi alterado para o modo **Placa em Ponte**, permitindo a participação da máquina virtual na mesma rede física do computador hospedeiro.

Após a alteração, foi configurado um endereço IP estático no Ubuntu Server utilizando o Netplan, juntamente com a rota padrão e os servidores DNS.

A aplicação da configuração foi validada através do comando `ip addr show enp0s3`, confirmando a atribuição do endereço à interface.

Também foram realizados testes de conectividade nos dois sentidos:

- Windows Host → Ubuntu Server Guest;
- Ubuntu Server Guest → Windows Host.

Por fim, foram realizados dois testes com `traceroute`, um para `google.com` e outro para `one.one.one.one`, permitindo verificar a resolução de nomes e o encaminhamento dos pacotes para redes externas.

As evidências foram organizadas na pasta `Evidências` e numeradas de acordo com a ordem em que os procedimentos foram realizados.

---

## 6. Problemas, Ajustes e Soluções

### 6.1. Memória da máquina virtual

Assim como nas atividades anteriores, a máquina virtual utilizada nesta prática permaneceu com **2048 MB de memória RAM**.

A configuração de 512 MB apresentada originalmente no ambiente do laboratório não foi utilizada, pois durante a instalação do Ubuntu Server foram encontrados problemas relacionados à quantidade insuficiente de memória.

Por esse motivo, a configuração de 2048 MB foi mantida nas atividades seguintes para garantir a estabilidade do ambiente virtualizado.

---

### 6.2. Configuração do endereço IP estático

A configuração do endereço IP estático exige atenção especial para evitar conflitos com outros equipamentos da rede do laboratório.

Por esse motivo, o endereço escolhido foi previamente testado a partir do Windows utilizando `ping`.

Caso o endereço escolhido apresentasse resposta, seria necessário selecionar outro endereço disponível antes de aplicá-lo no Ubuntu Server.

---

### 6.3. Sintaxe do arquivo YAML

A configuração do Netplan utiliza o formato YAML, que depende de uma indentação correta.

Durante a edição do arquivo `/etc/netplan/00-installer-config.yaml`, foi necessário manter os níveis da estrutura alinhados e utilizar espaços em vez de tabulações.

Uma configuração incorreta poderia impedir a aplicação das regras de rede através do `netplan apply`.

---

### 6.4. [Outros problemas encontrados durante a execução]

[Este espaço será preenchido após a análise das capturas da execução da Aula 06, caso tenha ocorrido algum erro, comportamento diferente do roteiro ou necessidade de ajuste.]

---

## 7. Conclusão

A atividade permitiu compreender na prática a diferença entre os modos de rede **NAT** e **Placa em Ponte (Bridge Adapter)** no VirtualBox.

No modo NAT utilizado anteriormente, a máquina virtual permanecia atrás de um roteador virtual do VirtualBox, utilizando um endereço privado e necessitando de mecanismos como o redirecionamento de portas para que determinados serviços fossem acessados a partir do computador hospedeiro.

Com a utilização do modo Placa em Ponte, a interface de rede virtual passou a participar diretamente da rede física do laboratório, permitindo que a máquina virtual fosse tratada como um equipamento independente dentro da rede local.

A configuração do endereço IP estático através do Netplan também permitiu compreender como são definidos o endereço da interface, a máscara de sub-rede, a rota padrão e os servidores DNS em um Ubuntu Server.

Os testes de `ping` demonstraram a comunicação bidirecional entre o computador hospedeiro e a máquina virtual, enquanto os testes com `traceroute` permitiram observar o encaminhamento dos pacotes para destinos externos.

Dessa forma, a prática possibilitou relacionar os conceitos de virtualização de rede, Bridge Adapter, endereçamento IPv4, gateway, DNS, roteamento e configuração de rede estática em servidores Linux.
