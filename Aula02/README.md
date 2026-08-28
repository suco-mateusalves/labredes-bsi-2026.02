# Relatório Técnico: Administração de Usuários, Grupos e Permissões no Linux

## 1. Identificação

- **Nome completo:** Mateus Alves dos Santos
- **Curso:** Sistemas de Informação
- **Data:** 28/08/2026
- **Título da prática:** Aula Prática 02: Administração de Usuários, Grupos e Permissões no Linux

---

## 2. Objetivo

A atividade teve como objetivo praticar a administração de usuários e grupos no Ubuntu Server, bem como compreender o funcionamento das permissões POSIX aplicadas a arquivos e diretórios.

A prática envolveu a criação de contas de usuários, criação e composição de grupos, definição de proprietário e grupo de arquivos e diretórios, aplicação de permissões por meio do `chmod` e realização de testes com usuários autorizados e não autorizados.

Além das configurações básicas propostas, foi utilizado o bit especial **SetGID** no diretório do setor financeiro. Essa configuração garante que novos arquivos e subdiretórios criados dentro dele herdem o grupo `financeiro`, facilitando a colaboração entre os integrantes do setor e mantendo a consistência das permissões.

---

## 3. Ambiente

A atividade foi realizada na máquina virtual configurada na Aula 01, utilizando:

- **Sistema operacional:** Ubuntu Server;
- **Virtualizador:** Oracle VM VirtualBox;
- **Usuário administrativo:** `administrador`;
- **Diretórios de trabalho:** `/srv/projeto` e `/srv/financeiro`;
- **Usuários utilizados:** `fulano`, `cicrano`, `beltrano` e `novato`;
- **Grupos utilizados:** `devs` e `financeiro`.

Os comandos administrativos foram executados com `sudo`, enquanto os testes de acesso foram realizados diretamente nas contas dos usuários envolvidos.

---

## 4. Procedimento

### 4.1. Criação dos usuários

Inicialmente foram criadas as contas que seriam utilizadas durante os testes de permissões:

```bash
sudo adduser fulano
sudo adduser cicrano
sudo adduser beltrano
sudo adduser novato
```

Após a criação, as contas foram verificadas no sistema para confirmar que estavam disponíveis para as etapas seguintes.

![Criação e verificação dos usuários](./Aula02/Evidências/01-criacao-usuarios.png)

---

### 4.2. Criação e configuração do grupo `devs`

Foi criado o grupo `devs` para representar uma equipe com acesso compartilhado ao diretório do projeto:

```bash
sudo groupadd devs
```

Em seguida, `fulano`, `cicrano` e `beltrano` foram adicionados como grupos suplementares:

```bash
sudo usermod -aG devs fulano
sudo usermod -aG devs cicrano
sudo usermod -aG devs beltrano
```

A associação foi validada consultando o arquivo `/etc/group`:

```bash
grep "devs" /etc/group
```

A saída confirmou os três usuários como membros do grupo `devs`. O usuário `novato` foi mantido fora do grupo para ser utilizado posteriormente nos testes de bloqueio.

![Criação do grupo devs e associação dos usuários](./Aula02/Evidências/02%20%26%2003%20-criacao-grupo-devs.png)

---

### 4.3. Criação do diretório compartilhado `/srv/projeto`

Foi criado o diretório destinado ao compartilhamento de arquivos entre os integrantes do grupo `devs`:

```bash
sudo mkdir -p /srv/projeto
```

A configuração inicial foi consultada com:

```bash
ls -ld /srv/projeto
```

Em seguida, o usuário `administrador` foi definido como proprietário e o grupo `devs` como grupo associado:

```bash
sudo chown administrador /srv/projeto
sudo chgrp devs /srv/projeto
```

As permissões do diretório foram configuradas como `770`:

```bash
sudo chmod 770 /srv/projeto
```

Essa configuração corresponde a:

| Categoria | Permissão | Efeito |
|---|---|---|
| Proprietário | `rwx` | leitura, escrita e acesso |
| Grupo `devs` | `rwx` | leitura, escrita e acesso |
| Outros | `---` | nenhum acesso |

A saída de `ls -ld /srv/projeto` confirmou a permissão `drwxrwx---` e a associação ao usuário `administrador` e ao grupo `devs`.

---

### 4.4. Criação e configuração do arquivo `config_redes.txt`

Dentro do diretório compartilhado foi criado o arquivo `config_redes.txt` com um conteúdo inicial:

```bash
echo "Especificacao tecnica do roteador de borda" > /srv/projeto/config_redes.txt
```

Posteriormente, o grupo do arquivo foi alterado para `devs`:

```bash
sudo chgrp devs /srv/projeto/config_redes.txt
```

e suas permissões foram definidas como `660`:

```bash
sudo chmod 660 /srv/projeto/config_redes.txt
```

A permissão `660` concede leitura e escrita ao proprietário e ao grupo, sem conceder qualquer permissão aos demais usuários.

A configuração final foi verificada com:

```bash
ls -l /srv/projeto/config_redes.txt
```

O resultado `-rw-rw----` confirmou que o arquivo estava configurado conforme o objetivo da atividade.

![Criação do diretório, configuração de proprietário, grupo, permissões e arquivo compartilhado](./Aula02/Evidências/04%20%26%2005%20%26%2006%20%26%2007%20-diretorio-projeto-proprietario-grupo-permissao-770-arquivo-config-redes.png)

---

## 5. Testes e Validação do Diretório `/srv/projeto`

### 5.1. Teste com o usuário autorizado `fulano`

Para validar as permissões do grupo `devs`, foi iniciada uma sessão com o usuário `fulano`:

```bash
su - fulano
```

O usuário conseguiu acessar normalmente o diretório:

```bash
cd /srv/projeto
```

e listar o arquivo existente:

```bash
ls -l
```

Também foi realizado um teste de escrita:

```bash
echo "Revisado por Fulano" >> config_redes.txt
```

Por fim, o conteúdo foi consultado:

```bash
cat config_redes.txt
```

A presença das duas linhas no arquivo confirmou que `fulano`, como membro do grupo `devs`, possuía tanto permissão de leitura quanto de escrita.

![Acesso e escrita realizados pelo usuário fulano](./Aula02/Evidências/08%20%26%2009-fulano-acesso-fulano-escrita.png)

---

### 5.2. Teste com o usuário não autorizado `novato`

O usuário `novato`, que não pertence ao grupo `devs`, foi utilizado para validar a restrição de acesso.

Ao tentar acessar e listar o conteúdo de `/srv/projeto`, o sistema retornou **Permission denied**, comprovando que as permissões `770` do diretório estavam impedindo o acesso de usuários externos ao grupo.

![Bloqueio do usuário novato no diretório do projeto](./Aula02/Evidências/10%20%26%2011-novato-bloqueado-novato-ls-bloqueado.png)

---

## 6. Exercício Prático — Diretório do Setor Financeiro

### 6.1. Criação do grupo e associação dos usuários

Para o exercício de fixação foi criado o grupo:

```bash
sudo groupadd financeiro
```

Os usuários `cicrano` e `beltrano` foram adicionados ao grupo:

```bash
sudo usermod -aG financeiro cicrano
sudo usermod -aG financeiro beltrano
```

Dessa forma, apenas os integrantes definidos para o setor financeiro deveriam possuir acesso ao diretório correspondente.

---

### 6.2. Criação e propriedade do diretório

Foi criado o diretório:

```bash
sudo mkdir -p /srv/financeiro
```

O usuário `administrador` foi definido como proprietário:

```bash
sudo chown administrador /srv/financeiro
```

e o grupo associado foi alterado para `financeiro`:

```bash
sudo chgrp financeiro /srv/financeiro
```

---

### 6.3. Aplicação da permissão `2770` e do SetGID

No diretório `/srv/financeiro` foi utilizada a permissão:

```bash
sudo chmod 2770 /srv/financeiro
```

O primeiro algarismo, `2`, não representa uma permissão comum de leitura, escrita ou execução. Ele ativa o bit especial **SetGID** no diretório.

Assim, `2770` combina:

- `2` — ativa o **SetGID**;
- `7` — proprietário com `rwx`;
- `7` — grupo com `rwx`;
- `0` — outros sem qualquer permissão.

O mesmo bit especial também pode ser aplicado de forma simbólica com:

```bash
sudo chmod g+s /srv/financeiro
```

Portanto, `chmod 2770` já ativa o SetGID; o uso de `chmod g+s` representa explicitamente a mesma característica e pode ser utilizado para reforçar ou aplicar apenas esse bit sem alterar as demais permissões.

Em um diretório com SetGID, novos arquivos e subdiretórios criados por integrantes do grupo passam a **herdar o grupo do diretório pai**, em vez de depender apenas do grupo primário do usuário que os criou. Para um diretório compartilhado entre membros de um setor, esse comportamento evita inconsistências de grupo e facilita o trabalho colaborativo.

A configuração foi conferida com:

```bash
ls -ld /srv/financeiro
```

A presença de `s` na posição de execução do grupo, como em `drwxrws---`, indica que o SetGID está ativo.

Também foi utilizada a consulta:

```bash
grep "financeiro" /etc/group
```

que confirmou `cicrano` e `beltrano` como membros do grupo.

![Criação do grupo financeiro, diretório, permissão 2770 e SetGID](./Aula02/Evidências/12%20%26%2013-grupo-financeiro-diretorio-financeiro.png)

---

## 7. Testes e Validação do Diretório `/srv/financeiro`

### 7.1. Teste de leitura e escrita com `cicrano`

Foi iniciada uma sessão com o usuário `cicrano`, integrante do grupo `financeiro`.

Dentro do diretório compartilhado, o usuário criou o arquivo `relatorio.txt`:

```bash
echo "Relatorio Financeiro Q3" > /srv/financeiro/relatorio.txt
```

Em seguida, acrescentou uma nova linha:

```bash
echo "Atualizacao de dados" >> /srv/financeiro/relatorio.txt
```

Por fim, o conteúdo foi consultado:

```bash
cat /srv/financeiro/relatorio.txt
```

A exibição das duas linhas confirmou que `cicrano` possuía acesso de leitura e escrita ao diretório financeiro.

![Criação e alteração do relatório pelo usuário cicrano](./Aula02/Evidências/14-cicrano-financeiro.png)

---

### 7.2. Teste de bloqueio com `fulano`

O usuário `fulano` pertence ao grupo `devs`, mas não ao grupo `financeiro`.

Durante o teste, foram realizadas tentativas de listar o diretório e criar um arquivo em `/srv/financeiro`. Ambas foram bloqueadas pelo sistema com a mensagem **Permission denied**.

Também foi possível observar que `fulano` continuava tendo acesso ao diretório `/srv/projeto`, demonstrando que as permissões estavam separando corretamente os dois ambientes de trabalho.

![Bloqueio de fulano no setor financeiro e acesso preservado ao projeto](./Aula02/Evidências/15-fulano-financeiro-bloqueado.png)

---

### 7.3. Teste de bloqueio com `novato`

O usuário `novato`, que não pertence aos grupos `devs` nem `financeiro`, também foi utilizado para testar as restrições.

As tentativas de acessar `/srv/financeiro`, acessar `/srv/projeto` e escrever no arquivo do setor financeiro foram negadas pelo sistema.

Esse teste confirmou que usuários sem associação aos grupos autorizados não possuem acesso aos recursos protegidos.

![Bloqueio do usuário novato nos diretórios protegidos](./Aula02/Evidências/16-novato-financeiro-bloqueado.png)

---

## 8. Resultados

Ao final da atividade foram obtidos dois ambientes compartilhados com políticas distintas de acesso:

| Recurso | Grupo autorizado | Permissão | Resultado |
|---|---|---|---|
| `/srv/projeto` | `devs` | `770` | membros de `devs` possuem acesso completo |
| `config_redes.txt` | `devs` | `660` | membros de `devs` podem ler e escrever |
| `/srv/financeiro` | `financeiro` | `2770` | membros possuem acesso completo e SetGID ativo |
| `relatorio.txt` | herdado de `financeiro` | conforme criação | compartilhamento consistente pelo grupo |

Os testes comprovaram que:

- `fulano` conseguiu acessar e alterar arquivos de `/srv/projeto`;
- `novato` foi bloqueado no diretório do projeto;
- `cicrano` conseguiu criar e modificar arquivos em `/srv/financeiro`;
- `fulano` foi bloqueado no diretório financeiro;
- `novato` foi bloqueado nos ambientes protegidos;
- o diretório financeiro foi configurado com **SetGID**, garantindo a herança do grupo em novos recursos criados dentro dele.

---

## 9. Problemas, Ajustes e Soluções

Durante a atividade, uma atenção especial foi dada às permissões de diretórios compartilhados.

Em `/srv/projeto`, a permissão `770` foi suficiente para permitir acesso completo ao proprietário e ao grupo `devs`, mantendo os demais usuários sem acesso.

No exercício do setor financeiro, entretanto, foi adotada a permissão `2770` em vez de apenas `770`. A inclusão do bit SetGID torna a configuração mais adequada para um diretório colaborativo, pois novos arquivos e subdiretórios passam a manter o grupo `financeiro`.

A configuração também pode ser expressa simbolicamente com:

```bash
sudo chmod g+s /srv/financeiro
```

Esse comando não substitui a necessidade de definir as permissões comuns quando elas ainda não estiverem corretas; ele atua especificamente sobre o bit SetGID. No caso realizado, `chmod 2770` já reúne as permissões `770` e o SetGID em uma única operação.

Os testes com usuários pertencentes e não pertencentes aos grupos foram utilizados para confirmar que as alterações produziram o comportamento esperado.

---

## 10. Conclusão

A atividade permitiu compreender, na prática, como o Linux utiliza usuários, grupos, propriedade e permissões para controlar o acesso a arquivos e diretórios.

A configuração de `/srv/projeto` demonstrou o uso das permissões tradicionais, com `770` no diretório e `660` no arquivo compartilhado, permitindo que os integrantes do grupo `devs` trabalhassem sobre os mesmos recursos enquanto usuários externos permaneciam bloqueados.

O exercício do setor financeiro ampliou essa configuração por meio do uso do **SetGID**. A permissão `2770` manteve o acesso restrito ao proprietário e ao grupo `financeiro` e acrescentou a herança de grupo para novos arquivos e subdiretórios. O comando simbólico `chmod g+s` permitiu compreender também outra forma de representar esse bit especial.

Os testes com `fulano`, `cicrano` e `novato` demonstraram que as políticas configuradas estavam funcionando conforme esperado: usuários autorizados conseguiram ler e escrever nos recursos de seus grupos, enquanto usuários sem autorização receberam `Permission denied`.

Dessa forma, a prática não se limitou à execução dos comandos, mas demonstrou como permissões e grupos podem ser utilizados para organizar ambientes colaborativos com isolamento e controle de acesso em um servidor Linux.
