# Documentação
Documentação Monitoramento de SSL via Zabbix integrado com o envio de alertas para o telegram


## Configurações no Servidor Zabbix

**1)** Logue via SSH no servidor Zabbix e adicionar este script abaixo na pasta `/home/zabbix/` (caso não tenha essa pasta é só cria-la) e troque o nome do arquivo para `checkssl.sh`

**Script:**

    data=`echo | openssl s_client -servername $1 -connect $1:443 2>/dev/null | openssl x509 -noout -enddate | sed -e 's#notAfter=##'`
    
    ssldate=`date -d "${data}" '+%s'`
    
    nowdate=`date '+%s'`
    
    diff="$((${ssldate}-${nowdate}))"
    
    echo $((${diff}/86400))

**2)** Em seguida, dê a permissões de execução:

`$ sudo chmod a+x checkssl.sh`

**3)** Para testar o Script você pode colocar um site qualquer que tenha SSL. Por exemplo:

`$ ./checkssl.sh github.com`  
`$ ./checkssl.sh google.com`  

O comando deve retornar um número indicando quantos dias faltam para o certificado SSL expirar.

**4)** Configuração relacionada ao Zabbix.

Em seguida, preciso editar o arquivo de configuração do Agente Zabbix.

`$ sudo nano /etc/zabbix/zabbix_agentd.conf`

E definir `EnableRemoteCommands=1`

![enter image description here](https://miro.medium.com/max/693/1*kwQwXtwtJKRtrty2ryIf_Q.png)

**4.1)** Saia e salve as alterações.

**4.2)** Reinicie o processo do agente Zabbix.

`$ sudo service zabbix-agent restart`


## Configurações no Zabbix Interface.

**1)** Adicionando um *HOST* em um *HOST GROUP*:

**Passo 1:** Utilizando um usuário administrador da conta Zabbix Clique em *Configuration* > *Host Groups* > E Selecione um grupo de sua escolha no caso da print a seguir foi selecionado o grupo *"Zabbix Servers"*

**Passo 2:** Clique em *Hosts* ao lado de *Zabbix Servers*

![enter image description here](https://i.imgur.com/RLxa0jO.png)

**Passo 3:** Após clicar em *Hosts* do grupo *Zabbix Servers* Clique em *Create Host* no canto superior direito.
![enter image description here](https://i.imgur.com/PEt8xsf.png)

**Passo 4:** Configurações do *Host*:

> **4.1:** De um nome ao *Host*.
> 
> **4.2:** Selecione o grupo que deseja colocar esse *Host* neste caso *Zabbix Servers*
> 
> **4.3:** Defina o IP do *"Agent"*, (Neste caso foi colocado o próprio IP do Servidor *Zabbix*. 
> 
> **4.4:** *(Opcional)* - Adicione uma descrição.

![enter image description here](https://i.imgur.com/yHJnZ1K.png) 

**2)** Após isso será gerado um *host* como esse:
![enter image description here](https://i.imgur.com/tFMTUns.png)


## Criando item dentro de *host:*

**Passo 1:** Clique em item ao lado do host que você criou:
![enter image description here](https://i.imgur.com/9XBnR5H.png)

**Passo 2:** Clique em *Create item* no canto superior direito:
![enter image description here](https://i.imgur.com/7BYpRAk.png)

**Passo 3:** Defina as configurações do item:

> **3.1:** Defina um nome para o item. (Lembre-se de trocar a url do site que deseja monitorar)
> 
> **3.2:** *Type* será *Zabbix agent*.
> 
> **3.3:** Utilize esta *key*: `system.run[/home/zabbix/checkssl.sh arcasolutions.com]` Substituindo a url para a qual deseja monitorar.
> 
> **3.4:** Defina o *Host Interface* para o IP do *host group* 
> 
> **3.5:** Defina *Type information* como *Numeric (float)*
> 
> **3.6:** em *Update interval* defina o tempo em que deseja que o script seja rodado para atualização no Zabbix, no exemplo do print a seguir foi colocado 1m, mas pode ser definido para uma atualização diária a cada 24h.
> 
> **3.7:** O restante das configurações podem ser mantidas como padrão.
> 
> **3.8:** Clique em update, para adicionar o item configurado.

![enter image description here](https://i.imgur.com/FA6dCS3.png)

**Passo 4:** Confira se o monitoramento está em funcionamento clicando em `*Monitoring > Latest Data*` após isso encontre seu item que foi criado anteriormente e veja se em *"Last Value"* está contando os dias em que falta para o vencimento do SSL.

![enter image description here](https://i.imgur.com/I6lOjoJ.png)

**5)** Pronto após seu item criado, configurado e funcionando, vamos passar para a criação de *triggers*.


## Configurando uma *Trigger* para gerar alertas:

**Passo 1:** Clique em *Triggers* que fica ao lado do *host* criado anteriormente:
![enter image description here](https://i.imgur.com/eBH04S2.png)

**Passo 2:** Clique no botão *Create Trigger* no canto superior direito da tela:
![enter image description here](https://i.imgur.com/jEATpVk.png)

**Passo 3:** Configurações da *Trigger*:

> **3.1:** De um nome a *Trigger*
> 
> **3.2:** Defina o grau de severidade do problema

    Observação: O grau de severidade é muito importante para que os alertas não se tornem banalizados por equipes que acompanham os alertas, neste caso não trate todos os alertas como *"Disaster"* por exemplo. Pois as equipes que acompanham podem acabar banalizando esses alertas por tratar tudo como *"Disaster"* neste caso é interessante colocar o grau de severidade de acordo com o impacto que esse alerta trás para o ambiente de trabalho.

> **3.3:** Em *Expression* clique no botão "Add" > Ao aparecer a tela de *"Condition"* clique em *"SELECT"* e Selecione seu *"Host"* > Após isso selecione o seu item, como mostra na **terceira** imagem.
> 
> **3.4:** Ainda na tela de *"Condition"* selecione a "*Function*" que deve ser igual a da segunda imagem abaixo `last() - Last (most recent) T valeu`
> em *"Result"* coloque o tempo que o alerta deve lhe avisar sobre o tempo de vencimento do SSL.

![enter image description here](https://i.imgur.com/J60RIz8.png)

![enter image description here](https://i.imgur.com/LVECXVQ.png)

![enter image description here](https://i.imgur.com/Jthzt3P.png)


**Passo 4:** Após finalizar todas essas configurações clique no botão "*Update*" e pronto, sua "*Trigger"* está pronta e vai gerar alertas quando chegar no dia configurado por você.

![enter image description here](https://i.imgur.com/pyDwgC6.png)

> **Teste:** Caso você queira testar esse alerta você pode definir um valor para a trigger do dia atual de vencimento do SSL é só checar no item quantos dias faltam e depois do sinal de "=" você coloca essa data por exemplo: Caso esteja faltando 20 dias para o vencimento é só deixar a expressão configurada assim: `last(/MonitoramentoSSL/system.run[/home/zabbix/checkssl.sh arcasolutions.com])=20` após fazer isso e clicar em update entrando na aba `*Monitoring >Problems*` na interface do zabbix é possível ver o alerta que foi criado.

## Configurando o recebimento de alertas no Telegram:

**Passo 1:** Para iniciar este processo é necessário entrar no Telegram de preferencia via WEB e iniciar uma conversa com 2 bot's, são eles o "BotFather" que vai lhe passar um token para que o zabbix entre em comunicação com o telegram e o "IDBot" que vai lhe repassar o seu ID de conexão. Assim como mostra as imagens abaixo:

**1 -** Pesquise o nome do bot na lupa de pesquisa do telegram:

![enter image description here](https://i.imgur.com/czjmKNW.png)

**2 - ** Clique no bot e inicie uma conversa clicando em *"Start/Restart Bot"*:

![enter image description here](https://i.imgur.com/zORDNtZ.png)

>**2.1** Após isso digite o comando `/newbot` no chat de conversa do "*BotFather"*
>
> **2.2:** Digite o nome que você vai dar ao Chat em que o bot vai enviar os alertas.
> 
> **2.3:** Digite o nome que você deseja dar ao Bot (É necessário que tenha `_bot` no final).
> 
> **2.4:** Após isso pegue o token que o BOT gerou e guarde, pois vamos usa-lo no zabbix.

![enter image description here](https://i.imgur.com/vRwFzhL.png)

**3)** O próximo passo é pegar o ID do seu telegram ou grupo para adicionar no zabbix como forma de comunicação também. para isso utilizaremos o "*IDBot"* como citado anteriormente. Assim como no "*BotFather"* procure o bot na lupa de pesquisa do telegram após isso é necessário clicar em *"Start/Restar Bot"*.

![enter image description here](https://i.imgur.com/Q7TjTqx.png)


![enter image description here](https://i.imgur.com/rwFmZhN.png)

> **3.1:** Digite `/getid` para o bot retornar o seu ID na mensagem abaixo (No print não foi mostrado por questão de segurança).
> **3.2:** Copie esse ID e guarde assim como o *Token* pois será utilizado também no Zabbix.

![enter image description here](https://i.imgur.com/fXa5ySH.png)

**4)** Após realizar esses procedimento inicie uma conversa no telegram com o bot que você criou:

![enter image description here](https://i.imgur.com/7COVFPt.png)


![enter image description here](https://i.imgur.com/DCtpmwO.png)

Envie qualquer mensagem no chat, apenas para iniciar uma conversa com o bot criado.

## Configurações de alertas enviados para o telegram no Zabbix.

**Passo 1:** Clique em `administrarion > media types > telegram`

![enter image description here](https://i.imgur.com/O05PeQx.png)

**Passo 2:** Clone o *type media* do telegram com o nome que você desejar para o monitoramento.

![enter image description here](https://i.imgur.com/MOqaFrC.png)

**Passo 3:** Renomeie o clone que você criou e adicione aquele token em "*Parameters"* que o "BotFather" lhe enviou no chat:

![enter image description here](https://i.imgur.com/vpcHInI.png)

**Passo 3:** Clique em *UPDATE* no final da página.

![enter image description here](https://i.imgur.com/0eKjgr7.png)

**Passo 4:** Realize o teste clicando no botão "*test"* para verificar se a mensagem foi enviada para seu telegram.

![enter image description here](https://i.imgur.com/blFlAm2.png)

**4.1:** Após clicar em "*test"* a tela de configurações vai abrir, adicione o assunto da mensagem de teste, em *"To"* adicone o seu ID que foi pego com o *"IDBot"* no telegram e adicione seu token novamente:

![enter image description here](https://i.imgur.com/xOTdmsV.png)

**Passo 5:** Clique em `Configuration > Action> Trigger Actions`

![enter image description here](https://i.imgur.com/7LMrWB5.png)

**Passo 6:** Clique em *"Create Action"* no canto superior direito:

![enter image description here](https://i.imgur.com/w2qC3qD.png)

**Passo 7:** Defina um nome para a *Action* e depois clique em "*Add"*:

![enter image description here](https://i.imgur.com/e4RjOHc.png)

**Passo 8:** Marque a caixa de seleção do host que foi criado por você anteriormente e clique em select:

![enter image description here](https://i.imgur.com/JSeapL9.png)

**Passo 9:** Clique em *"Operations"* e depois clique em *"Add"*:

![enter image description here](https://i.imgur.com/iqSm98t.png)

**Passo 10:** Adicione o usuário que você quer que envie a mensagem, pode ser tanto o user admin ou seu usuário e clique em select:

![enter image description here](https://i.imgur.com/TK00OPn.png)

**Passo 11:** Em *"Send only to"* coloque o tipo de media que você clonou, renomeou e configurou. Em *"Subject"* fica a seu critério porém vou deixar um padrão de variáveis que puxa as informações do Zabbix.

        Subject: 
    
	    {TRIGGER.STATUS}: {EVENT.NAME}
    
        Message: 
        
        **Data : {EVENT.DATE} Horário :**{EVENT.TIME}
        *Hostname : {HOST.NAME} IP : * {HOST.IP}
        *Item de Monitoramento :* {EVENT.NAME}
        *Descrição do Incidente :* {TRIGGER.DESCRIPTION}
        *Descrição do servidor :* {HOST.DESCRIPTION}
        *Consumo / Status :* {ITEM.NAME1} {ITEM.VALUE1}
        *Evento ativo á :* {EVENT.AGE}
        *Original event ID:* {EVENT.ID}

![enter image description here](https://i.imgur.com/urWVsae.png)

**Passo 12:** Clique em *"Update"* nas duas telas para adicionar a regra de mensagem.

![enter image description here](https://i.imgur.com/kMccAFE.png)

![enter image description here](https://i.imgur.com/cVSdZSr.png)

**Passo 13:** Para finalizar as configurações é necessário configurar o usuário que vai enviar o alerta para o telegram. Clique em `Administration > User` e selecione o usuário que você configurou na mensagem.

![enter image description here](https://i.imgur.com/3Vfi5hc.png)

![enter image description here](https://i.imgur.com/GQ9XbTa.png)

**Passo 14:** Clique em media e depois clique em add.

![enter image description here](https://i.imgur.com/L906Na6.png)

**Passo 15:** Selecione o tipo de media que será aquele que foi clonado e renomeado por você e "*Send To"* coloque o seu ID do telegram e clique em *"Update"*

![enter image description here](https://i.imgur.com/JXoVt30.png)

**Passo 16:** Para finalizar clique em *update* na tela anterior:

![enter image description here](https://i.imgur.com/BO7kGxA.png)

## Documentos complementares:

Documentação do Zabbix:

https://www.zabbix.com/documentation/current/pt






