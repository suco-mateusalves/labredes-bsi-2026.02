# Relatório Técnico: Acesso Remoto SSH e Diagnóstico de Rede no Ubuntu Server

## 1. Identificação

- **Nome completo:** Mateus Alves dos Santos
- **Matrícula:** 2023002055
- **Curso:** Sistemas de Informação
- **Turma:** 2026.2
- **Data:** 16/09/2026
- **Título da prática:** Aula Prática 05: Acesso Remoto SSH via Redirecionamento de Portas no VirtualBox e Diagnóstico de Rede

---

## 2. Objetivo

A atividade teve como objetivo praticar o diagnóstico de rede no Ubuntu Server e compreender o funcionamento do acesso remoto por meio do protocolo **SSH (Secure Shell)**.

A prática também envolveu a identificação das interfaces de rede, endereço IP, gateway, tabela de rotas e caminho percorrido pelos pacotes até um endereço externo, utilizando ferramentas como `netplan`, `ifconfig`, `route` e `traceroute`.

Em seguida, foi configurado o **redirecionamento de portas (Port Forwarding)** do VirtualBox, permitindo que uma conexão realizada no computador hospedeiro Windows, através da porta `5222`, fosse encaminhada para a porta `22` do Ubuntu Server, onde o serviço SSH estava disponível.

Por fim, foram utilizados comandos no Windows e no Ubuntu Server para verificar o estado das conexões TCP e confirmar o estabelecimento de uma sessão SSH remota.

---

## 3. Ambiente

A atividade foi realizada na mesma máquina virtual utilizada nas aulas anteriores.

O ambiente utilizado foi:

- **Sistema operacional:** Ubuntu Server 26.04;
- **Virtualizador:** Oracle VM VirtualBox;
- **Sistema operacional do hospedeiro:** Windows;
- **Usuário administrativo:** `administrador`;
- **Processador virtual:** 1 vCPU;
- **Memória RAM:** 2048 MB;
- **Disco virtual:** 32 GB;
- **Modo de rede:** NAT;
- **Interface de rede da VM:** `enp0s3`;
- **Endereço IP da VM:** `10.0.2.15`;
- **Gateway da VM:** `10.0.2.2`;
- **Porta SSH do servidor:** `22`;
- **Porta utilizada no hospedeiro:** `5222`.

A máquina virtual permaneceu com **2048 MB de memória RAM**, configuração que já havia sido adotada nas atividades anteriores devido aos problemas de memória encontrados durante a instalação do Ubuntu Server.

No modo NAT, a máquina virtual possui acesso à rede externa por meio da infraestrutura virtualizada do VirtualBox. Para permitir que o computador hospedeiro estabeleça uma conexão SSH com a máquina virtual, foi utilizado o recurso de redirecionamento de portas.

A configuração utilizada foi:

```text
IP do Hospedeiro: 127.0.0.1
Porta do Hospedeiro: 5222
IP do Convidado: 10.0.2.15
Porta do Convidado: 22
Protocolo: TCP
```

---

## 4. Procedimento

### 4.1. Verificação do estado da rede com `netplan`

Inicialmente foi verificado o estado das interfaces de rede da máquina virtual utilizando:

```bash
netplan status
```

O comando foi utilizado para verificar a configuração e o estado atual da interface de rede utilizada pelo Ubuntu Server.

![Status da rede com netplan](./Evidências/01.png)

---

### 4.2. Verificação do OpenSSH Server

Em seguida foi verificado se o pacote responsável pelo servidor SSH estava instalado:

```bash
dpkg -l | grep openssh-server
```

A utilização desse comando permitiu confirmar a presença do pacote `openssh-server` no sistema.

O SSH utiliza a porta TCP `22` como porta padrão para receber conexões.

![Verificação do OpenSSH Server](./Evidências/02.png)

---

### 4.3. Atualização dos repositórios

Antes da instalação das ferramentas de diagnóstico, foi realizada a atualização da lista de pacotes:

```bash
sudo apt update
```

Esse procedimento atualiza as informações disponíveis nos repositórios configurados no Ubuntu Server.

![Atualização dos repositórios](./Evidências/03.png)

---

### 4.4. Instalação das ferramentas de diagnóstico

Foram instalados os pacotes `net-tools` e `traceroute`:

```bash
sudo apt install -y net-tools traceroute
```

O pacote `net-tools` disponibiliza ferramentas tradicionais de administração de rede, como `ifconfig` e `route`.

Já o pacote `traceroute` disponibiliza o comando utilizado para identificar os saltos realizados pelos pacotes até determinado destino.

![Instalação do net-tools e traceroute](./Evidências/04.png)

---

### 4.5. Verificação das interfaces de rede com `ifconfig`

Após a instalação das ferramentas, foi executado:

```bash
ifconfig
```

O comando permitiu visualizar as interfaces de rede disponíveis no Ubuntu Server.

Entre as interfaces apresentadas estava a `enp0s3`, utilizada para a comunicação da máquina virtual com a rede disponibilizada pelo VirtualBox.

Também foi possível identificar o endereço IPv4 atribuído à interface, além da interface de loopback `lo`.

![Resultado do ifconfig](./Evidências/05.png)

---

### 4.6. Verificação da tabela de rotas

A tabela de roteamento da máquina virtual foi consultada utilizando:

```bash
route -n
```

O comando apresentou as rotas configuradas no sistema em formato numérico.

Foi possível identificar a rota padrão utilizada pela máquina virtual e o gateway `10.0.2.2`, fornecido pelo ambiente NAT do VirtualBox.

![Tabela de rotas](./Evidências/06.png)

---

### 4.7. Rastreamento do caminho até o destino

Para verificar o caminho percorrido pelos pacotes até um endereço externo, foi utilizado:

```bash
traceroute 8.8.8.8
```

O endereço `8.8.8.8` foi utilizado como destino para o teste.

A execução do comando permitiu observar os saltos realizados pelos pacotes. O primeiro salto corresponde ao gateway virtual utilizado pela máquina virtual.

![Resultado do traceroute](./Evidências/07.png)

---

### 4.8. Verificação das sessões ativas

Antes de estabelecer a conexão SSH, foi executado:

```bash
w
```

O comando permite visualizar informações sobre os usuários atualmente conectados ao sistema e suas respectivas sessões.

Nesse momento foi verificada a sessão existente diretamente no console da máquina virtual.

![Sessões ativas antes do SSH](./Evidências/08.png)

---

### 4.9. Verificação da configuração de rede no Windows

Após os testes realizados no Ubuntu Server, foram realizadas verificações no computador hospedeiro Windows.

Foi executado:

```powershell
ipconfig /all
```

O comando apresentou as configurações das interfaces de rede disponíveis no Windows, incluindo endereços IP, máscara, gateway e outras informações relacionadas à configuração de rede.

![Configuração de rede do Windows](./Evidências/09.png)

---

### 4.10. Verificação da porta 5222 antes do redirecionamento

Antes da configuração do Port Forwarding no VirtualBox, foi verificado se existia alguma conexão utilizando a porta `5222`:

```powershell
netstat -an | findstr 5222
```

A consulta foi realizada para estabelecer uma situação inicial antes da criação da regra de redirecionamento.

![Netstat antes do Port Forwarding](./Evidências/10.png)

---

### 4.11. Configuração do redirecionamento de portas no VirtualBox

Em seguida foi configurado o redirecionamento de portas no VirtualBox.

A configuração foi realizada nas opções de rede da máquina virtual, no **Adaptador 1**, utilizando a opção de **Redirecionamento de Portas**.

Foi criada uma regra para o serviço SSH com os seguintes parâmetros:

```text
Nome: SSH
Protocolo: TCP
IP do Hospedeiro: 127.0.0.1
Porta do Hospedeiro: 5222
IP do Convidado: 10.0.2.15
Porta do Convidado: 22
```

A finalidade da regra é fazer com que uma conexão direcionada para:

```text
127.0.0.1:5222
```

seja encaminhada pelo VirtualBox para:

```text
10.0.2.15:22
```

Ou seja, a porta `5222` utilizada no Windows funciona como uma porta de entrada para o serviço SSH disponível na porta `22` do Ubuntu Server.

![Configuração do Port Forwarding SSH](./Evidências/11.png)

---

### 4.12. Verificação da porta após o redirecionamento

Após a criação da regra no VirtualBox, o comando foi executado novamente no Windows:

```powershell
netstat -an | findstr 5222
```

A verificação teve como objetivo confirmar que a porta configurada para o redirecionamento estava sendo utilizada pelo VirtualBox.

A presença da porta em estado `LISTENING` indica que existe um serviço aguardando conexões naquele endereço e porta.

![Netstat após o Port Forwarding](./Evidências/12.png)

---

### 4.13. Estabelecimento da conexão SSH

Com o redirecionamento configurado, foi realizada uma conexão SSH a partir do Windows utilizando:

```powershell
ssh -p 5222 administrador@127.0.0.1
```

Nesse comando:

- `ssh` inicia o cliente SSH;
- `-p 5222` informa a porta utilizada no computador hospedeiro;
- `administrador` corresponde ao usuário do Ubuntu Server;
- `127.0.0.1` corresponde ao próprio computador hospedeiro.

O VirtualBox recebe a conexão na porta `5222` e realiza o encaminhamento para a porta `22` da máquina virtual.

![Conexão SSH com o Ubuntu Server](./Evidências/13.png)

---

### 4.14. Verificação da conexão TCP estabelecida

Com a sessão SSH aberta, foi realizada uma nova consulta no Windows:

```powershell
netstat -an | findstr 5222
```

Nesse momento foi possível verificar, além da porta em estado `LISTENING`, a existência de uma conexão TCP em estado `ESTABLISHED`.

O estado `ESTABLISHED` indica que a conexão TCP entre o cliente e o serviço SSH foi efetivamente estabelecida.

![Conexão SSH em estado ESTABLISHED](./Evidências/14.png)

---

### 4.15. Verificação da sessão SSH com `w`

Por fim, já dentro da sessão SSH, foi executado novamente:

```bash
w
```

O comando permitiu verificar as sessões atualmente conectadas ao Ubuntu Server.

Além da sessão existente no console da máquina virtual, foi possível identificar a nova sessão criada pelo acesso SSH, associada ao terminal `pts/0`.

![Sessão SSH identificada pelo comando w](./Evidências/15.png)

---

## 5. Testes e Evidências

Os testes realizados permitiram verificar tanto a configuração da rede do Ubuntu Server quanto o funcionamento do redirecionamento de portas e do acesso remoto por SSH.

Foram realizadas as seguintes validações:

- verificação do estado da interface de rede com `netplan status`;
- confirmação da instalação do `openssh-server`;
- atualização dos repositórios do Ubuntu;
- instalação das ferramentas `net-tools` e `traceroute`;
- identificação das interfaces com `ifconfig`;
- identificação da rota padrão com `route -n`;
- rastreamento de pacotes com `traceroute`;
- verificação das sessões ativas com `w`;
- identificação das configurações de rede do Windows com `ipconfig /all`;
- verificação da porta `5222` antes da configuração do redirecionamento;
- configuração da regra de Port Forwarding no VirtualBox;
- verificação da porta `5222` após a configuração;
- estabelecimento da conexão SSH;
- identificação da conexão TCP em estado `ESTABLISHED`;
- identificação da sessão SSH através do comando `w`.

As evidências foram organizadas na pasta `Evidências` e numeradas de acordo com a ordem de execução da atividade.

---

## 6. Problemas, Ajustes e Soluções

### 6.1. Memória da máquina virtual

Assim como nas atividades anteriores, a máquina virtual utilizada nesta prática permaneceu com **2048 MB de memória RAM**.

A configuração de 512 MB apresentada originalmente no ambiente do laboratório não foi utilizada, pois durante a instalação do Ubuntu Server nas atividades anteriores foram encontrados problemas relacionados à quantidade insuficiente de memória.

Por esse motivo, a configuração de 2048 MB foi mantida nas atividades seguintes para garantir a estabilidade do ambiente.

---

### 6.2. [Outros problemas encontrados durante a execução]

[Este espaço será preenchido após a análise das capturas da execução da Aula 05, caso tenha ocorrido algum erro, comportamento diferente do roteiro ou necessidade de ajuste.]

---

## 7. Conclusão

A atividade permitiu compreender, de forma prática, o funcionamento da configuração de rede de uma máquina virtual Ubuntu Server executada no VirtualBox utilizando o modo NAT.

Por meio dos comandos `netplan status`, `ifconfig` e `route -n`, foi possível identificar a interface de rede, o endereço IP atribuído à máquina virtual e a rota utilizada para comunicação com outras redes.

A utilização do `traceroute` permitiu observar o caminho percorrido pelos pacotes até um endereço externo, enquanto o comando `w` foi utilizado para identificar as sessões de usuários conectadas ao servidor.

Na segunda parte da atividade foi configurado o redirecionamento de portas do VirtualBox. A regra criada encaminhou conexões destinadas à porta `5222` do computador hospedeiro para a porta `22` do Ubuntu Server, onde o serviço SSH estava disponível.

Os testes realizados com `netstat` permitiram acompanhar o funcionamento dessa configuração, inicialmente verificando a situação da porta antes do redirecionamento e, posteriormente, identificando a porta em estado `LISTENING` e a conexão em estado `ESTABLISHED` após o estabelecimento da sessão SSH.

Por fim, a utilização do comando `w` dentro da sessão remota permitiu identificar o terminal `pts/0`, confirmando que uma nova sessão havia sido estabelecida por meio do SSH.

Dessa forma, a prática possibilitou relacionar conceitos de endereçamento IP, roteamento, NAT, portas TCP, redirecionamento de portas e acesso remoto, permitindo observar na prática como esses componentes trabalham em conjunto para possibilitar a comunicação entre o computador hospedeiro e uma máquina virtual.

