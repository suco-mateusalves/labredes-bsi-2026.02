# Relatório Técnico: Introdução à Virtualização e Instalação do Ubuntu Server 26.04

## 1. Identificação

- **Nome completo:** Mateus Alves dos Santos
- **Curso:** Sistemas de Informação
- **Data:** 12/08/2026
- **Título da prática:** Aula Prática 01: Introdução à Virtualização e Instalação do Ubuntu Server 26.04

---

## 2. Objetivo

A atividade teve como objetivo compreender os conceitos fundamentais de virtualização por meio da criação e configuração de uma máquina virtual no Oracle VM VirtualBox e da instalação do Ubuntu Server 26.04.

A prática contemplou a preparação do hardware virtual, configuração da rede, instalação do sistema operacional, criação de um layout de armazenamento personalizado com LVM, criação do usuário administrativo, instalação do OpenSSH Server e validação do ambiente após a instalação.

Ao final, foram utilizados comandos para verificar a conectividade de rede, o armazenamento disponível e o funcionamento dos repositórios de pacotes.

---

## 3. Ambiente

A prática foi executada utilizando o Oracle VM VirtualBox como plataforma de virtualização.

A máquina virtual foi configurada com os seguintes recursos:

| Configuração | Valor |
|---|---|
| **Nome da VM** | `ubuntu_server` |
| **Sistema operacional** | Ubuntu Server 26.04 |
| **Arquitetura** | 64-bit |
| **Processador** | 1 vCPU |
| **Memória RAM inicialmente prevista** | 512 MB |
| **Memória RAM utilizada após ajuste** | 2048 MB |
| **Armazenamento** | 32 GB |
| **Formato do disco** | VDI |
| **Alocação** | Dinamicamente alocado |
| **Rede** | NAT |
| **Interface de rede** | `enp0s3` |

A memória inicialmente prevista para a atividade era de 512 MB. Durante a execução, entretanto, foi necessário aumentar a memória da máquina virtual para 2048 MB devido a um erro de falta de memória durante a inicialização do instalador. O ocorrido é detalhado na seção **6. Problemas e Soluções**.

---

## 4. Procedimento

### 4.1. Criação e configuração da máquina virtual

Inicialmente, foi criada uma nova máquina virtual no Oracle VM VirtualBox com o nome `ubuntu_server`.

Durante a criação foram definidos o sistema operacional Ubuntu 64-bit, a quantidade de processadores, a memória RAM e o disco virtual de 32 GB em formato VDI com alocação dinâmica.

A imagem ISO do Ubuntu Server 26.04 foi selecionada como mídia de instalação.

![Nome, sistema operacional e imagem ISO da VM](./Evidências/01.01-configuracao-vm.png)

![Configuração de memória e processador](./Evidências/01.02-configuracao-vm.png)

![Configuração do disco virtual](./Evidências/01.03-configuracao-vm.png)

Após a configuração, a máquina virtual ficou disponível no VirtualBox para inicialização.

![Máquina virtual criada no VirtualBox](./Evidências/02-maquina-virtual.png)

---

### 4.2. Inicialização do instalador

A máquina virtual foi inicializada utilizando a imagem ISO do Ubuntu Server.

Na tela de boot foi selecionada a opção de inicialização do instalador do Ubuntu Server.

![Inicialização do instalador do Ubuntu Server](./Evidências/03.01-instalacao-ubuntu.png)

---

### 4.3. Seleção do idioma

O idioma **English** foi utilizado durante o processo de instalação.

![Seleção do idioma do instalador](./Evidências/03.02-instalacao-ubuntu.png)

---

### 4.4. Configuração do teclado

O layout do teclado foi configurado como **Portuguese (Brazil)**.

![Configuração do teclado](./Evidências/03.03-instalacao-ubuntu.png)

---

### 4.5. Seleção do tipo de instalação

Foi selecionada a opção padrão **Ubuntu Server**, mantendo a instalação completa do ambiente de servidor.

![Seleção do tipo de instalação](./Evidências/03.04-instalacao-ubuntu.png)

---

### 4.6. Configuração da rede

O instalador identificou a interface `enp0s3`, que recebeu configuração automaticamente por DHCP através da rede NAT fornecida pelo VirtualBox.

![Configuração da interface de rede](./Evidências/03.05-instalacao-ubuntu.png)

---

### 4.7. Configuração do repositório de pacotes

Foi mantido o mirror de pacotes do Ubuntu apresentado pelo instalador. A comunicação com o repositório foi validada durante o próprio processo de instalação.

![Configuração do mirror de pacotes](./Evidências/03.06-instalacao-ubuntu.png)

---

### 4.8. Configuração personalizada do armazenamento

Para o armazenamento foi utilizado o modo **Custom storage layout**, permitindo configurar manualmente as partições e volumes.

![Seleção do layout personalizado de armazenamento](./Evidências/04.01-particionamento.png)

O disco virtual de aproximadamente 32 GB foi selecionado para receber a estrutura de armazenamento.

![Seleção do disco virtual](./Evidências/04.02-particionamento.png)

Foi criada uma partição de aproximadamente 1 GB, formatada em `ext4` e montada em:

```text
/boot
```

![Criação da partição /boot](./Evidências/04.03-particionamento.png)

No espaço restante foi configurado o LVM, utilizando o grupo de volumes:

```text
ubuntu-vg
```

Foram criados os volumes lógicos:

- `ubuntu-lv`, com aproximadamente 29 GB, formatado em `ext4` e montado em `/`;
- `swap-lv`, com aproximadamente 2 GB, destinado à área de swap.

A estrutura final ficou organizada da seguinte forma:

| Recurso | Tamanho aproximado | Utilização |
|---|---:|---|
| `/boot` | 1 GB | arquivos de inicialização |
| `ubuntu-lv` | 29 GB | sistema de arquivos raiz `/` |
| `swap-lv` | 2 GB | área de swap |

![Estrutura final do armazenamento](./Evidências/04.04-particionamento.png)

---

### 4.9. Configuração do usuário administrativo

Durante a configuração do perfil foi criado o usuário administrativo do servidor:

- **Nome:** `Administrador`;
- **Nome do servidor:** `ubuntu_server`;
- **Usuário:** `administrador`.

A senha definida durante a instalação não é reproduzida neste relatório.

![Configuração do usuário administrador](./Evidências/05-usuario-administrador.png)

---

### 4.10. Instalação do OpenSSH Server

Foi selecionada a instalação do **OpenSSH Server**, preparando o sistema para permitir acesso remoto seguro por SSH.

![Configuração do OpenSSH Server](./Evidências/06-openssh.png)

---

## 5. Testes e Validação

Após a conclusão da instalação e reinicialização do sistema, foram realizados testes para verificar o funcionamento do ambiente.

### 5.1. Atualização dos repositórios

Foi executado:

```bash
sudo apt-get update
```

O comando atualizou as informações sobre os pacotes disponíveis nos repositórios e confirmou que a máquina virtual possuía conectividade suficiente para acessar os servidores configurados.

![Execução do sudo apt-get update](./Evidências/07-apt-update.png)

---

### 5.2. Verificação da configuração de rede

Foi utilizado o comando:

```bash
ip addr
```

A saída permitiu verificar as interfaces de rede disponíveis e os endereços IP associados à máquina virtual, incluindo a interface `enp0s3`.

![Resultado do comando ip addr](./Evidências/08-ip-addr.png)

---

### 5.3. Verificação do armazenamento

Para conferir os sistemas de arquivos montados e o espaço disponível, foi executado:

```bash
df -h
```

O resultado permitiu validar o armazenamento configurado durante a instalação.

![Resultado do comando df -h](./Evidências/09-df-h.png)

---

## 6. Problemas e Soluções

### 6.1. Memória insuficiente durante a inicialização do instalador

**Problema:**  
A máquina virtual foi inicialmente configurada com **512 MB de memória RAM**, conforme a especificação original da atividade. Entretanto, durante a inicialização do instalador do Ubuntu Server, ocorreu um erro relacionado à falta de memória, impedindo a continuidade normal da instalação.

**Causa:**  
A quantidade de memória atribuída à máquina virtual não foi suficiente para que o instalador fosse carregado e executado corretamente naquele ambiente.

**Solução:**  
A máquina virtual foi desligada e a memória base foi aumentada de **512 MB para 2048 MB (2 GB)** nas configurações do VirtualBox. Após a alteração, o instalador conseguiu iniciar normalmente e a atividade pôde prosseguir.

A configuração final de memória utilizada pode ser observada na evidência abaixo:

![Configuração da VM com 2048 MB de memória](./Evidências/01.02-configuracao-vm.png)

Essa ocorrência demonstrou que, além de seguir uma especificação inicial, é necessário avaliar os recursos efetivamente exigidos pelo sistema operacional durante a execução.

---

## 7. Conclusão

A atividade permitiu compreender, na prática, o processo de criação de uma máquina virtual e de instalação de um sistema operacional voltado para servidores.

Foram configurados os principais recursos de hardware virtual, a rede, o usuário administrativo, o OpenSSH Server e um esquema de armazenamento personalizado utilizando LVM. A criação da partição `/boot`, do volume lógico raiz e da área de swap permitiu observar uma forma mais flexível de organização do armazenamento em sistemas Linux.

Os comandos `sudo apt-get update`, `ip addr` e `df -h` foram utilizados para validar o funcionamento dos repositórios, da rede e do armazenamento após a instalação.

Também foi necessário diagnosticar e corrigir um problema de insuficiência de memória. O aumento de 512 MB para 2048 MB permitiu concluir a instalação e evidenciou a importância de dimensionar adequadamente os recursos atribuídos a uma máquina virtual.

Ao final da prática, o Ubuntu Server encontrava-se instalado e funcional, deixando o ambiente preparado para as atividades seguintes da disciplina.
