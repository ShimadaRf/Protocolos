# Header IPv4 -
## Visão Geral -
O cabeçalho IPv4 é a parte frontal, essencial e compacta de cada datagrama que trafega pela internet, agindo como o envelope que guia a informação da origem ao destino. Sua estrutura básica é de 20 bytes e contém os dados cruciais que definem como o pacote deve ser roteado. Os primeiros campos incluem a versão, sempre 4 para este protocolo, e o IHL (Internet Header Length), que informa o tamanho total do cabeçalho. Em seguida, o campo tipo de serviço permite que roteadores priorizem o tráfego conforme a necessidade. O comprimento total indica o tamanho completo do pacote, enquanto os campos de identificação, flags e fragment offset são dedicados à fragmentação, permitindo que pacotes grandes sejam divididos para passar por redes com limitações de tamanho e, depois, remontados corretamente no destino. O TTL (time to live) é um contador de hops que evita que pacotes circulem em loops infinitos, ele é decrementado em cada roteador e o pacote é descartado ao chegar a zero. O campo protocolo é vital, pois informa qual protocolo da camada de transporte (como TCP ou UDP) está contido na área de dados, e o Checksum é usado para garantir a integridade do próprio cabeçalho. Por fim, temos os campos de endereços IP de origem e destino, cada um com 32 bits, que definem quem enviou e quem deve receber o pacote.  
O header segue o padrão de formação abaixo:
  
<img src="imagens\IPV4\headeripv4.png" alt="Header IPv4" width="700">  
   
Este conjunto de informações não apenas facilita o roteamento, mas também possui uma importância central na cibersegurança. A manipulação dos campos do cabeçalho IPv4 é um vetor comum para ataques de negação de serviço e evasão de sistemas de segurança. Por exemplo, a falsificação do endereço de origem (IP Spoofing) permite que um atacante se mascare ou realize ataques de amplificação, sobrecarregando uma vítima com tráfego refletido, é difícil imaginar um cenário onde um bom pentester não conheça a fundo os headers dos protocolos. Os campos de fragmentação podem ser explorados para contornar firewalls e IDS que não são capazes de reassemblar corretamente os pacotes para inspeção, permitindo que payloads maliciosos passem despercebidos. Por outro lado, a inspeção rigorosa desses mesmos campos é a base de todas as defesas de rede, incluindo regras de firewall baseadas no endereço de origem/destino e a análise do campo protocolo por sistemas IDS para identificar tráfego anômalo ou malicioso, tornando o cabeçalho IPv4 o principal ponto de controle e monitoramento no tráfego de rede.
A ideia desse tópico é explorar a fundo os campos do header IP e trazer a importância deles para a segurança. Abaixo temos algumas vulnerabilidades que podem ser exploradas, diretamente ou indiretamente, do header estudado.

1. Spoofing  
Os campos mais explorados são os de endereçamento. A principal ameaça é o IP Spoofing, onde o atacante falsifica o endereço IP de origem. Isso é usado para ocultar a identidade do agressor ou para lançar ataques de reflexão e amplificação, onde as respostas de servidores legítimos são direcionadas ao endereço falsificado da vítima, sobrecarregando-a. O endereço IP de destino, embora geralmente legítimo, pode ser alvo em ataques man-in-the-middle  locais, onde o roteamento é manipulado para desviar o pacote.
2. Vulnerabilidades de controle e qualidade  
O campo TTL, embora protetor contra loops infinitos, pode ser usado ofensivamente. Ao observar os valores de TTL em pacotes de resposta, um atacante pode realizar a engenharia reversa da topologia da rede, inferindo o número de hops até o alvo. O campo protocolo (que indica se o pacote é TCP, UDP, etc.) é crucial para firewalls, mas um valor inválido ou inesperado pode ser enviado para tentar causar ineficiências ou falhas em sistemas legados. Já o campo type of service pode ser manipulado para tentar forçar o tráfego malicioso a receber uma alta prioridade (QoS) na rede, prejudicando a latência do tráfego legítimo.
3. Riscos de Integridade e Malformação  
O header checksum, que visa garantir a integridade do cabeçalho, é limitado, um atacante pode facilmente recalcular o checksum para um cabeçalho que contenha informações ofensivas, pois ele não verifica a integridade do payload. Além disso, a manipulação de campos de controle de tamanho pode causar negação de serviço, valores malformados para a versão (diferente de 4), o IHL, ou o total length podem fazer com que sistemas, especialmente os mais antigos, falhem, congelem ou tentem ler fora dos limites da memória, causando buffer overflows.
4. Ameaças da Fragmentação  
Os campos de fragmentação (identification, flags e fragment offset) representam um grande risco de evasão. Eles são explorados em ataques como teardrop e com o uso de fragmentos sobrepostos (Overlapping Fragments). Nestes ataques, o invasor envia pedaços de dados que, quando remontados, sobrecarregam o sistema de destino ou, pior, resultam em pacotes que contornam a inspeção de firewalls e IDS. Por exemplo, um fragmento pode ser intencionalmente pequeno, contendo apenas o início do cabeçalho TCP/UDP, forçando o firewall a inspecionar apenas informações incompletas e permitindo que o payload malicioso passe despercebido.
5. Exploração das Opções  
O campo options, embora menos comum, é vulnerável. Se a opção source routing estiver habilitada, um atacante pode especificá-la para forçar o pacote a seguir um caminho específico na rede, ignorando as regras de roteamento padrão e potencialmente acessando segmentos de rede protegidos.

## Version -
O version é o primeiro campo do cabeçalho de um datagrama IP, ele tem 4 bits em seu campo, e possui uma função crítica na arquitetura de rede, ele determina a sintaxe e a semântica do restante do cabeçalho do datagrama. Especificamente, o valor contido no campo version indica a versão do protocolo IP à qual o datagrama em questão adere, e essa informação é fundamental para qualquer nó de rede (roteadores, firewalls e o hospedeiro destinatário) que receba o pacote, pois a estrutura e o significado dos campos subsequentes no cabeçalho variam drasticamente entre as versões. Para o IPv4, o campo version contém o valor binário 0100 (decimal 4), o que instrui o dispositivo a interpretar o datagrama de acordo com o formato estabelecido pela RFC 791 e suas extensões, enquanto para o IPv6, o campo contém o valor binário 0110 (decimal 6), exigindo a interpretação da estrutura simplificada e modificada definida pela RFC 8200. Ao ser o primeiro campo a ser examinado, o version permite que os dispositivos de processamento de pacotes despachem o datagrama para o módulo de software ou hardware apropriado para análise e encaminhamento, o que é um componente chave do mecanismo de fast-path ou forwarding dentro dos roteadores, minimizando a latência e garantindo a interoperabilidade. A existência deste campo na posição inicial viabiliza a transição gradual e a coexistência de múltiplas versões do IP na infraestrutura global da Internet, permitindo que roteadores inspecionem o campo e decidam se o datagrama deve ser encaminhado (potencialmente via tunnel), processado nativamente ou descartado se a versão não for suportada, assegurando a evolução contínua da rede sem interrupção.

Embora o campo de versão no cabeçalho IPv4 ocupe apenas 4 bits e pareça ter uma função estática, ele desempenha um papel tático interessante na cibersegurança quando manipulado ou analisado profundamente. Do ponto de vista defensivo, esse campo atua como o primeiro filtro de sanidade em qualquer dispositivo de rede. Como a especificação exige que o valor seja estritamente 4, firewalls e sistemas de detecção de intrusão (IDS) utilizam essa verificação como uma forma de triagem rápida. Se um pacote chega com qualquer valor diferente, ele pode ser descartado imediatamente no nível do hardware ou do driver, economizando ciclos da CPU que seriam gastos analisando o restante do cabeçalho ou a carga útil. Portanto, a defesa utiliza a imutabilidade desse campo para garantir a eficiência e detectar anomalias. Um alerta de "versão de IP inválida" em um monitoramento é **quase** sempre um indicador de alta fidelidade de que alguém está utilizando ferramentas de injeção de pacotes customizados na rede, pois sistemas operacionais legítimos não cometem esse erro naturalmente.

Na perspectiva ofensiva, a manipulação do campo version pode ser utilizada para testar a robustez dos analisadores de pacotes (parsers) ou para tentar evasão de segurança. Atacantes podem empregar técnicas de fuzzing, enviando pacotes com versões inválidas (como 0, 5 ou 15) para tentar causar uma negação de serviço (DoS) em equipamentos legados ou mal programados, que podem entrar em falha ao tentar processar um valor inesperado em vez de descartá-lo corretamente. Além disso, existe a técnica de desincronização de interpretação, onde um atacante tenta enganar um firewall para que ele ignore o pacote (achando, por exemplo, que é um tráfego IPv6 malformado), enquanto o sistema alvo vulnerável processa o pacote malicioso.

Ainda no cenário ofensivo e de reconhecimento, o campo serve para técnicas de fingerprinting ativo e canais encobertos. No fingerprinting, o atacante envia propositalmente um pacote com versão incorreta para analisar como o alvo responde, um sistema Windows pode simplesmente silenciar, enquanto um roteador Cisco ou um servidor Linux podem enviar uma mensagem de erro ICMP específica. Essa diferença de comportamento ajuda o atacante a mapear qual sistema operacional está rodando do outro lado sem precisar de um scan agressivo. Já em canais encobertos, dois dispositivos comprometidos dentro de uma rede podem se comunicar usando valores de versão inválidos como sinais de comando e controle, como a maioria dos dispositivos de rede descarta esses pacotes sem registrá-los profundamente nos logs de aplicação, essa comunicação pode passar despercebida por administradores que não estejam analisando o tráfego bruto no nível do cabo.

**Obs:** RFC significa Request for Comments. Os padrões mencionados no texto, RFC 791 e RFC 8200 são, basicamente, o nome do documento oficial que define o IPv4 e o IPv6, respectivamente. Eles agem para padronizar e criar uma "constituição" do protocolo que precisa ser seguida por todos que desejam utilizá-las. Sendo assim, a gente pode ver os protocolos IPv4 e IPv6 como produtos desse manual.

## IHL -
O campo Internet Header Length está localizado nos quatro bits menos significativos¹ do primeiro byte do cabeçalho IPv4, compartilhando o octeto inicial com o campo version. Esse campo desempenha uma função arquitetural crítica, ele define a fronteira estrutural entre os metadados de roteamento e do payload do pacote. Diferente de contadores de tamanho convencionais que operam em bytes, o IHL utiliza uma unidade de medida baseada em "palavras" de 32 bits (ou 4 bytes). Essa escolha de design reflete a necessidade histórica e computacional de alinhamento de memória, garantindo que o processamento do cabeçalho seja otimizado pela CPU.

A aritmética do IHL impõe limites rígidos ao protocolo. Como o valor é interpretado multiplicando-se o binário por 4, o menor valor funcional aceitável é 5 (0101), o que corresponde aos 20 bytes obrigatórios de um cabeçalho IPv4 padrão sem opções extras. No outro extremo, sendo um campo de 4 bits, seu valor máximo é 15 (1111), permitindo um cabeçalho de até 60 bytes. A diferença entre o mínimo (20 bytes) e o máximo (60 bytes) é reservada para o campo options e seu respectivo padding.

Do ponto de vista da cibersegurança e da engenharia de redes, o IHL atua como um offset vital para o sistema operacional. Ele instrui a pilha TCP/IP exatamente onde iniciar a leitura dos protocolos de camada superior (como TCP ou UDP). Qualquer inconsistência neste valor gera vetores de risco: um IHL menor que 5 classifica o pacote como malformado, resultando em descarte imediato ou erros de processamento; já um IHL inflado artificialmente, sem a presença real de opções, pode indicar uma tentativa de covert channel (canal encoberto), onde um atacante utiliza o espaço excedente do cabeçalho para trafegar dados ocultos ou comandos maliciosos, evadindo inspeções superficiais de firewall que focam apenas no payload.

    1. Bits menos significativos não significam menos importantes. São apenas bits que têm o menor valor numérico dentro de uma sequência binária. Eles ficam localizados na extremidade direita do número. Eles são frequentemente abreviados como LSB - Least Significant Bits

## Type of Service -
O campo ToS, que fica no segundo byte do cabeçalho, passou por algumas mudanças e não é o mesmo da versão original do header IPv4. Ele foi concebido na RFC 791 em 1981 e foi desenhado com o propósito de fornecer, aos roteadores, instruções sobre como priorizar e tratar datagramas específicos, permitindo uma gestão de tráfego que fosse além do simples "melhor esforço". Em sua concepção clássica, a estrutura dividia-se em três bits iniciais dedicados à precedência, quatro bits para três flags de serviço e dois bits reservados¹. A ideia era permitir que as aplicações especificassem suas necessidades, assim o roteador conseguiria ajudar cada uma de acordo com sua especificidade. Por exemplo, uma sessão de telnet solicitaria um baixo atraso, enquanto uma transferência FTP solicitaria uma alta vazão.

Na versão de 1992, a RFC 1349 introduziu a flag de custo (C). O objetivo era permitir que o tráfego fosse roteado através de caminhos de menor custo monetário, alterando a estrutura de flags para quatro bits (DTRC) e deixando apenas o último bit do byte como reservado. É fundamental compreender que, embora a RFC 1349, tenha sido substituída por normas posteriores, muitas pilhas TCP/IP legadas e sistemas industriais ainda operam sob essa lógica, criando um cenário heterogêneo onde equipamentos antigos interpretam bits de uma maneira, enquanto a infraestrutura moderna lê de outra forma.

**Visão Clássica**  
[ PPP | D | T | R | C | 0 ]
* **Precedence (PPP) - 3 bits**
    * 000 (0) - Routine
    * 001 (1) - Priority
    * 010 (2) - Immediate
    * 011 (3) - Flash
    * 100 (4) - Flash Override
    * 101 (5) - CRITIC/ECP 
    * 110 (6) - Internetwork Control
    * 111 (7) - Network Control
* **ToS (DTR) - 4 bits**
    * D (Delay) - Minimizar atraso
        * SSH, Telnet
    * T (Throughput) - Maximizar vazão
        * FTP
    * R (Reliability) - Maximizar confiabilidade
        * SNMP, DNS zone transfers
    * C (Cost) - Minimizar custo monetário
        * Foi adicionado na RFC 1349
* **Reserved - 1 bit**
    * MBZ (Must be Zero)

Na era moderna da engenharia de tráfego, a IETF declarou a obsolescência das definições anteriores através da RFC 2474 e da RFC 3168, implementando o modelo de Serviços Diferenciados (DiffServ). Nesta nova arquitetura, o antigo campo ToS foi renomeado para DS Field. Por isso não há um padrão muito claro quando pesquisamos sobre IPv4, inclusive na foto usada no começo desse documento, temos o campo ToS ainda como ToS e, ao pesquisar header IPv4, veremos que é fácil encontrar as duas nomenclaturas no campo. Acredito que essa diferença não tem um peso muito grande quando se estuda superficialmente e apenas de maneira teórica sobre o assunto, mas cada mínima vírgula fora do lugar pode ser usada por aqueles com olhos atentos e conhecimento. Os primeiros seis bits, agora chamados de Differentiated Services Code Point (DSCP), definem o comportamento salto-a-salto (PHB) que o pacote deve receber, permitindo uma granularidade de classes de serviço muito superior à antiga precedence. Os dois bits restantes, anteriormente desperdiçados ou reservados, foram reaproveitados para a notificação explícita de congestionamento (ECN), um mecanismo vital que permite aos roteadores sinalizarem congestionamento iminente marcando pacotes em vez de apenas descartá-los, possibilitando que as pontas da comunicação ajustem suas janelas de transmissão proativamente.

**Visão Moderna**
[ DSCP | ECN ]
* **DSCP (Differentiated Services Code Point) - 6 bits**
    * Default (CS0) - 000000
    * EF (Expedited Forwarding) - 101110
    * AF (Assured Forwarding) -
    * CS (Class Selector) - CS1 a CS7 (ex. CS6 110000) 
* **ECN (Explicit Congestion Notification) - 2 bits**
    * 00 - Non-ECT
        * Não suporta ECN
    * 01 - ECT(1)
        * ECN-Capable Transport
    * 10 - ECT(0)
        * ECN-Capable Transport
    * 11 - CE
        * Congestion Experienced. Marcado pelo roteador quando a fila está cheia. Dessa forma, o receptor vê isso e avisa ao remetente para diminuir a velocidade. (Esse aviso é feito via flag TCP CWR/ECE)

Agora a parte mais atrativa, olhando pelo lado da segurança ofensiva, a complexidade histórica desse campo abre vetores de ataque sofisticados, muitas vezes negligenciados por firewalls que não realizam inspeção profunda nesses bits. O uso mais sutil envolve a criação de canais encobertos, onde um atacante pode manipular os seis bits do campo DSCP ou as antigas flags da RFC 1349 para exfiltrar dados ou enviar comandos de controle bit a bit, invisíveis a logs de aplicação convencionais. Além disso, a discrepância na implementação de stacks TCP/IP permite o reconhecimento passivo de sistemas operacionais (a gente também chama isso de OS Fingerprinting), tendo em vista que diferentes sistemas definem valores ToS padrão distintos. Por exemplo, resposta com precedência elevada podem denunciar roteadores de borda específicos ou sistemas legados. 

Ainda do ponto de vista ofensivo, existe o risco de ataques de DoS via amplificação de QoS. Um atacante pode facilmente forjar pacotes com marcações de alta prioridade, fazendo com que o seu tráfego malicioso fure a fila dos roteadores, degradando ou bloqueando o tráfego legítimo sem necessariamente saturar a largura de banda total do link. Também, em ambientes mistos, onde coexistem implementações da RFC 1349 e DiffServ, a manipulação do bit de custo pode induzir um roteamento assimétrico, onde o tráfego de ataque segue um caminho físico diferente da resposta, potencialmente evadindo sensores de segurança posicionados em rotas principais.

Tenho uma facilidade maior em olhar pelo lado de quem ataca, mas é de vital importância sempre dar atenção aos dois lados da moeda. Afinal, como especialista de cibersegurança eu desejo conseguir evitar esse tipo de ataque onde eu atue. Desse modo, a mitigação dos ataques vindos desse campo vão exigir de nós uma postura de confiança zero em relação às marcações de QoS vindas de redes externas. A prática recomendada de traffic scrubbing² envolve a reescrita ou o zeramento do campo DS em todos os pacotes que entram pelo perímetro da rede, neutralizando canais encobertos e impedindo o abuso de prioridade. Paralelamente, a monitoria de anomalias no campo ECN pode servir como um sistema de alerta precoce para ataques volumétricos, indicando saturação de buffers antes mesmo que ocorra perda de pacotes significativa. Portanto, o domínio sobre o byte ToS não é só uma questão de engenharia de tráfego, mas um componente essencial na análise forense de rede e no desenvolvimento de mecanismos robustos de defesa.

Dissecar e mostrar como cada um dos possíveis ataques mencionados acima não é o foco do documento atual, mas para dar um ponta pé inicial naqueles que não conseguem ver isso acontecendo na prática. Vamos explorar um pouco mais o ataque DoS comentado acima.

<img src="imagens\IPV4\DSCPEFd.png" alt="Header IPv4" width="700"> 
<img src="imagens\IPV4\DSCPEFs.png" alt="Header IPv4" width="700"> 

No exemplo acima forjei um pacote com DSCP EF, representado pelo hex 0xB8. O comando usado foi "ping -c 3 -Q 184 192.168.1.109", mas existem diversas ferramentas que permitem esse tipo de manipulação de pacote (hping3). Olhando para esse pacote de maneira destrutiva e para cenários hipotéticos (ou nem tanto) vejamos como isso funciona no mundo real. Roteadores corporativos e de provedores configuram uma fila especial chamada LLQ (Low Latency Queue) ou priority queue. Essa fila é reservada para tráfego sensível a atraso, como VoIP ou vídeo em tempo real. A regra do roteador é, "se houver algo na fila EF, envie imediatamente e 'pare' todo o resto". Ou seja, se você iniciar um flood (como um UDP flood ou ICMP flood) setado com 0xB8 (EF), você não precisa saturar o link inteiro para causar danos. Você só precisa saturar a largura de banda reservada para a fila de prioridade :D. Dessa forma, o roteador ficará tão ocupado tentando despachar seus pacotes "da fila preferencial" que o tráfego de voz legítimo começará a picotar ou cair, e o tráfego normal ficará em starvation (ou seja, faminto, já que o roteador nunca vai ter tempo livre para atendê-lo). É um DoS cirúrgico que paralisa a operação crítica com menos volume de dados. Se você ficou curioso para saber como eu cheguei no número 184, pesquise sobre dscp values table, saiba de que maneira o valor decimal e o valor binário conversam para chegar no número usado para manipular esse campo.

    1. Esses bits reservados que encontramos em diversos lugares quando estamos estudando arquitetura de redes dizem respeito a padrões de segurança e engenharia. Na criação de estruturas tão complexas como a que estamos vendo, era, e ainda é, impossível prever com exatidão nossa necessidade anos a frente. A tecnologia evolui exponencialmente e esses espaços são deixados para que, caso haja a necessidade, alterações possam ser feitas sem quebrar a internet já existente. É muito mais fácil modificar a utilidade de um bit que já está lá do que criar uma estrutura nova e colocar todo o mundo nesse padrão. Se você, assim como eu, não se contenta com pouca informação, recomendo que estude mais a fundo o conceito de retrocompatibilidade!
    2. É, essencialmente, o processo de higienização de pacotes de rede. É uma técnica defensiva onde o tráfego que entra é inspecionado e toma um banho para entrar na rede limpinho. Esse tipo de processo tem como objetivo remover dados maliciosos, malformados ou não conformes, antes que esse tráfego atinja o destino final. Isso pode ser burlado com técnicas de tunelamento :D.

## Total Length - 
O campo total length ocupa os 16 bits subsequentes ao campo ToS, este valor dita ao kernel do sistema operacional exatamente onde o datagrama termina na memória. Ele vai nos dizer o tamanho total do datagrama IP, incluindo o próprio cabeçalho quanto os dados transportados. Por ele ocupar 16 bits dentro do cabeçalho, com essa quantidade ele consegue representar valores entre 0 e 65.535. Vale lembrar que, embora os bits permitam um valor 0, um pacote IP válido nunca pode ser menor que o seu próprio cabeçalho. Esse limite não é arbitrário, ele vem da projeção inicial do IPv4, veio de uma época de recursos restritos e precisava de uma forma compacta e eficiente de descrever o tamanho do pacote.

No campo ofensivo, a manipulação desse campo é usada para a criação de canais encobertos e evasão de detecção. Existe uma discrepância inerente entre a camada 2 (enlace) e a camada 3 (rede). O padrão ethernet exige um tamanho mínimo de quadro, muitas vezes forçando a inserção de padding no final do quadro se o pacote IP for muito pequeno. É crucial compreender que, para a pilha TCP/IP, qualquer dado que exista fisicamente no fio além do byte indicado pelo total length é irrelevante.

Um atacante pode explorar essa mecânica criando um pacote onde o total length declarado no cabeçalho IP é intencionalmente menor do que o tamanho real da carga útil transmitida no frame ethernet. Por exemplo, se um atacante envia um frame contendo 100 bytes de dados, mas define o total length do IP como 40 bytes, o sistema operacional da vítima, obedecendo aos padrões RFC, processará os 40 bytes e descartará os 60 bytes restantes como lixo de padding da camada de enlace. No entanto, esses 60 bytes "invisíveis" trafegaram pela rede e passaram por firewalls que inspecionam apenas até o limite do total length. Um malware ou implant na maquina de destino, operando no modo raw ou promíscuo, pode ler o quadro bruto da camada 2 e extrair esses dados ocultos, estabelecendo um canal de comando e controle que quase imperceptível.

O total length é vital no cálculo de remontagem de fragmentos. Já houveram ataques, históricos do meu ponto de vista, como o teardrop ou o bonk, que manipulavam os campos total length e o fragment offset de forma que os fragmentos se sobrepusessem na memória do alvo. Se o SO não tiver uma verificação de limites ao somar o offset com o total length de um fragmento recebido, ele pode sobrescrever estruturas de dados críticas do kernel, causando um kernel panic ou permitindo execução de código arbitrário.

Do outro lado, podemos usar o campo total length para uma análise comportamental. Certos tipos de tráfego possuem impressões digitais de tamanho muito específicas. Por exemplo, um ataque de buffer overflow contra um serviço vulnerável geralmente requer um payload grande para conter o shellcode e o NOP sled¹. Um IDS pode ser configurado para alertar sobre pacotes destinados a portas sensíveis que possuam um total length próximo ao MTU máximo, o que é atípico para simples comandos de controle. Da mesma forma, em ataques de amplificação DDoS, a resposta contendo o total length anormalmente grande em comparação à consulta é o principal indicador de anomalia que permite aos sistemas de mitigação filtrar o tráfego.

Vale ressaltar a diferença do campo total length com outros campos ou conceitos relacionados.
1. Total Length e IHL ->  
Como já foi dito, o total length nos dá o tamanho do pacote inteiro enquanto o IHL nos dá o tamanho do cabeçalho. Para extrair os dados, o sistema operacional realiza a seguinte operação aritmética:   
    $$Tamanho_{Payload} = TotalLength - (IHL \times 4)$$
    A multiplicação do IHL se dá, como já vimos, pelo fato de ele contar palavras de 32 bits, enquanto o Total Length conta bytes.
2. Total Length e MTU ->  
A MTU é uma restrição da camada de enlace (esse conceito será mais explorado nos próximos campos).
    * Se $TotalLength \le MTU$: O pacote é encapsulado em um quadro e transmitido.
    * Se $TotalLength > MTU$: O roteador deve fragmentar o pacote.

Um comportamento, talvez não tão intuitivo, para se observar é o processo de fragmentação. Quando um pacote original com um tamanho maior que o MTU é fragmentado para se adequar, o campo total length não é copiado do original. Ele é recalculado para cada novo fragmento. Cada fragmento é um pacote IP independente com seu próprio cabeçalho. O host de destino é quem vai usar o total length de cada fragmento recebido, subtrair o cabeçalho, e usar o fragment offset para colocar os dados no buffer de memória correto para recomposição.

    1. Um NOP sled é uma sequência de instruções no operation, ou seja, que não realizam nenhuma ação útil. Ela é usada para aumentar a probabilidade de sucesso de uma exploração de vulnerabilidade. A sequência NOP cria um tipo de caminho de escorregamento que, independentemente de onde o fluxo de execução "pousar" dentro do sled, eventualmente o guiará para o payload infectado localizado no final.

## Identification - 
O campo possui 16 bits e a sua criação fundamentou-se na necessidade de desambiguação. Em um fluxo contínuo de dados entre uma origem e um destino, o identification serve como a assinatura digital efêmera que distingue um pacote específico de seus antecessores e sucessores imediatos, permitindo que o receptor compreenda a individualidade de cada unidade de transmissão.

Historicamente, a especificação original da RFC 791 impunha uma rigidez semântica a este campo, exigindo que ele fosse único para cada datagrama enviado dentro de um intervalo de tempo determinado pelo maximum segment lifetime. Isso transformava o ID em uma variável de estado crítico mantida pelo kernel do sistema operacional. Para cumprir essa diretriz, as primeiras implementações de pilhas de rede adotaram contadores globais e incrementais. Nessa abordagem, o sistema operacional mantinha uma variável estática na memória que era incrementada linearmente a cada novo pacote gerado, independentemente de destino ou do protocolo encapsulado. Embora computacionalmente eficiente, essa metodologia transformou o campo identification em um canal lateral de vazamento de informação, expondo a taxa de transmissão interna do host e permitindo vetores de ataques sofisticados, como o idle scan, baseados puramente na previsibilidade aritmética deste campo.

A evolução das redes para larguras de banda na ordem de gigabits e terabits por segundo expôs a fragilidade matemática do campo. Com apenas 16 bits, o ciclo de vida do contador sofre um esgotamento acelerado conhecido como wrap-around. Em enlaces de alta velocidade, é possível exaurir todo o espaço de IDs e reiniciar a contagem em frações de segundo, criando um cenário de colisão onde dois datagramas distintos, emitidos em momentos diferentes, mas próximos, acabam compartilhando o mesmo identificador numérico. Esse fenômeno compromete a integridade da camada de rede, pois o protocolo perde a capacidade de garantir a distinção entre pacotes antigos e pacotes novos, levando a potenciais corrupções de dados silenciosas.

Em resposta a essas limitações físicas e de segurança, a semântica do campo sofreu uma redefinição com a introdução da RFC 6864. Esta atualização normativa bifurcou o tratamento do identification baseando-se na natureza atômica do datagrama. Para datagramas atômicos¹, o campo identification foi oficialmente depreciado, perdendo sua obrigatoriedade de unicidade. Isso permitiu que sistemas operacionais modernos passassem a preencher este campo com zero ou valores constantes em fluxos TCP onde a fragmentação é proibida, economizando entropia² e ciclos de CPU. Contudo, para datagramas que ainda permitem fragmentação, a exigência de unicidade permanece, forçando o uso de algoritmos de geração, como geradores pseudoaleatórios ou contadores segregados por destino, numa tentativa de equilibrar a conformidade com o protocolo e a mitigação de reconhecimento de tráfego por terceiros.

Hoje o campo identification opera em um estado de dualidade, é um vestígio legado necessário para a mecânica básica do protocolo IP em redes antigas ou instáveis, mas simultaneamente é tratado como um risco de segurança e um gargalo de desempenho em redes modernas. Sua análise forense não revela apenas a identidade de um pacote, mas frequentemente expõe a versão e a configuração do sistema operacional subjacente, servindo como uma impressão digital do comportamento do kernel na gestão de seus contadores internos.

É importante ressaltar que o campo identification não opera isoladamente, ele forma uma espécie de trio com os campos subsequentes. A combinação dele com os campos flags e fragment offset é o que permite a reconstrução.

    1. São datagramas que não podem ser fragmentados. Como a fragmentação é proibida, o campo identification perde sua função primária de axiliar na reconstrução do pacote.
    2. No contexto de SO, refere-se à reserva de aleatoriedade disponível no kernel. Eu gosto de pensar nele como um tanque de combustível de aleatoriedade.

## Flags - 

O campo flags é um componente crítico de controle situado na segunda palavra de 32 bits do header. Ele inicia-se no bit 48 e termina no bit 50 em relação ao início do header, ou seja, só possui 3 bits. A função primordial do campo é gerenciar a fragmentação e a remontagem do pacote. Ele permite que o protocolo IP desacople o tamanho do datagrama da limitação física do meio (o MTU vai ficar bem mais claro no cálculo do fragment offset), permitindo que pacotes grandes atravessem links com MTUs pequenos, dividindo e marcando para reconstruir depois.

O primeiro bit é reservado e sempre 0. Na prática, qualquer pacote com esse bit setado como 1 deve ser considerado malformado e geralmente é descartado. O segundo bit é chamado de DF, ou don't fragment. Caso ele esteja 0, o roteador tem permissão para fragmentar o pacote se o tamanho exceder o MTU da interface de saída e, caso ele esteja como 1, o roteador é proibido de fragmentar o pacote. O terceiro bit é chamado de MF, ou more fragment. Se ele estiver setado como 1, isso indica que este não é o último fragmento do datagrama original e que existem mais dados pertencentes a ele chegando. O MF setado como 0 indica que este é o último fragmento, assim o receptor consegue calcular o tamanho total do datagrama e valida se todos os fragmentos necessários para completar a remontagem já foram recebidos. Se o pacote não for fragmentado o MF dele vem como 0 também.

Assim como os outros dois campos dessa linha, identification e fragment offset, o campo flags é inútil sem os outros. O ID vai garantir a unicidade, ou seja, se dois processos enviassem, por exemplo, pings ao mesmo tempo, o receptor misturaria os fragmentos do ping a com o ping b corrompendo os dados. O campo flags vai dizer quando a espera por novos pacotes deve finalizar. E o campo fragment offset, próximo da sequência, vai dizer onde cada bloco deve ser encaixado.

## Fragment Offset -
O campo Fragment Offset foi concebido juntamente com o próprio internet protocol (IP) na década de 1970. Esse conceito surgiu da necessidade de interoperabilidade entre redes com diferentes capacidades. No final dos anos 1970 e início dos anos 1980, o protocolo IP foi projetado para rodar sobre diversas tecnologias de camada de enlace. O problema é que cada uma dessas tecnologias impõe um maximum transmission unit (MTU), ou seja, um tamanho máximo de pacote. Isso nos levou a pensar no que fazer caso um datagrama gerado em uma rede com um MTU alto precisasse atravessar uma rede com um MTU baixo. Os arquitetos do IP, principalmente Jon Postel e Vint Cerf, decidiram que os roteadores intermediários teriam a responsabilidade de quebrar, em outras palavras, fragmentar o pacote em pedaços menores. Para permitir que o destino remontasse os pedaços na ordem correta, foi criado o campo fragment offset, ele indica a posição de início dos dados contidos naquele fragmento em relação ao payload do datagrama original.

O cabeçalho IPv4 foi projetado para ser o mais compacto possível, com apenas 20 bytes obrigatórios e o espaço destinado ao fragment offset no cabeçalho é de apenas 16 bits, sendo que 3 deles são reservados para as flags, já discutidas anteriormente, e os outros 13 para o fragment offset. Esse número extremamente baixo se dá pela grande escassez de recursos, estamos falando de uma época onde cada mínimo bit importava, as restrições de hardware eram extremas. Esse número extremamente baixo de bits acarretou em alguns problemas. O maior valor binário que o campo do fragment offset poderia representar seria de 2¹³ = 8191. O problema é que o tamanho máximo de um datagrama IPv4 é de 65.535 bytes, ou seja, como o fragment offset pode dizer onde cada pacote vai começar e terminar se o limite máximo dele é de 8191? A solução foi mais simples do que se pode imaginar, é só multiplicar por 8, ou seja, se o valor do campo offset é de 185, sabemos que esse pacote começa no byte 1480. Isso explica o porquê de o campo fragment offset não ser sequencial, ele não diz a ordem dos pacotes, mas, na verdade, a posição inicial dos bytes daquele payload, pensar nele como um mapa me ajudou na compreensão.

Para calcular como o fragment offset é determinado, o processo é bem simples e realmente depende de apenas três números: o tamanho total do payload, o tamanho do cabeçalho IP (geralmente 20 bytes no IPv4) e o MTU da rede. Pensemos no seguinte exemplo, se temos um payload de 5000 bytes, um cabeçalho de 20 bytes e um MTU de 1500 bytes, o primeiro passo é, de fato, remover o tamanho do cabeçalho da MTU para descobrir o tamanho máximo de dados reais que cada fragmento pode carregar. Isso nos dá 1500 - 20 = 1480 por fragmento.

O segredo aqui é que o fragment offset é medido em unidades de 8 bytes. Por isso, o tamanho dos dados que você está enviando em cada fragmento (os 1480 bytes) precisa ser um múltiplo de 8, o que 1480 é (1480 / 8 = 185).

Daí, é só dividir o payload total pelos fragmentos de 1480 bytes: 5000 dividido por 1480 resulta em três fragmentos completos e um resto.

1. O primeiro fragmento carrega os primeiros 1480 bytes. Seu deslocamento (offset) é 0 / 8 = 0.
2. O segundo fragmento começa no byte 1480. Seu deslocamento é 1480 / 8 = 185.
3. O Terceiro Fragmento começa no byte 2960 (1480 + 1480). Seu deslocamento é 2960 / 8 = 370.
4. O último fragmento começa no byte 4440 (2960 + 1480) e carrega os 560 bytes restantes. Seu deslocamento é 4440 / 8 = 555.

<img src="imagens\IPV4\calcfo1.png" alt="Calculo F.O." width="500">

O fragment offset é, portanto, o ponto de partida dos dados de cada fragmento, expresso em múltiplos de 8. Ele informa ao receptor exatamente onde o bloco de dados daquele pacote se encaixa no payload original de 5000 bytes para que ele possa ser reconstruído corretamente.

Apesar de sua engenhosidade, o mecanismo de fragmentação do IPv4 introduz uma complexidade indesejada, gerando sobrecarga de processamento nos roteadores e no host de destino. No contexto da cibersegurança, o fragment offset é fundamental em técnicas de evasão de IDS/IPS, como o tiny fragment attack e a sobreposição maliciosa de fragmentos, também conhecida como overlap. O manuseio inconsistente da remontagem em diferentes stacks TCP/IP é uma vulnerabilidade conhecida, sendo um dos motivos pelos quais o IPv6 praticamente eliminou a fragmentação intermediária, aderindo estritamente ao princípio end-to-end.

## Time to Live - 
O campo TTL é um contador de 8 bits. Ele atua como um mecanismo de segurança para limitar a vida útil de um datagrama na rede. A existência do TTL é crítica para evitar o problema de count-to-infinity em redes de comutação de pacotes¹. Sem esse campo, um erro na configuração das tabelas de roteamento faria com que pacotes circulassem indefinidamente entre roteadores, consumindo largura de banda e poder de processamento até saturar a rede. O que o TTL garante é que todos os pacotes tenham um morte caso se percam em caminhos diversos.

Originalmente, ou seja, na visão da RFC 791, o TTL foi definido como uma unidade de tempo em segundos. A especificação ditava que se um pacote ficasse num roteador por mais de 1 segundo, o campo deveria ser decrementado pelo tempo gasto. Mas hoje em dia ele já não funciona mais assim. Com a RFC 1812, a prática se baseia em decrementar o TTL em 1 unidade a cada hop. Hoje, o TTL é universalmente tratado como um contador de saltos, e não como um cronômetro.

O ciclo de vida do pacote pode ser resumido em 3 fases. A primeira é a inicialização, onde o sistema operacional (kernel) define o valor inicial do TTL ao criar o soquete bruto ou encapsular o datagrama. Esse valor é configurável via parâmetros de kernel. A segunda fase é o processamento em trânsito. Quando um roteador recebe um pacote, ele executa a seguinte lógica simplificada:

    - Verifica a integridade do cabeçalho (checksum)
    - Analisa o valor do TTL
    - TTL novo = TTL atual - 1
    - Recálculo do checksum

A terceira e última fase é a condição de parada. Se o roteador for menor ou igual a 0, ele deve ser descartado. O roteador não encaminha um pacote com TTL zero.

A interação do TTL não é isolada, ela depende intrinsecamente do protocolo auxiliar ICMP. Quando um pacote é descartado por exaustão de TTL, o roteador deve informar a origem para evitar buracos negros na rede. O roteador gera um pacote ICMP encapsulando o cabeçalho IP original junto com os primeiros 64 bits do payload original (para que a origem saiba qual conexão falhou) e o envia de volta ao IP de origem.

O campo TTL é amplamente usado por ferramentas para detecção de rota. Um exemplo é o traceroute. O traceroute não rastreia um pacote. Ele envia vários pacotes suicidas. Como o roteador descarta e envia uma resposta ICMP de volta para a origem, ele consegue montar um mapa do percurso que o pacote percorre.

Com os 8 bits de espaço, temos um TTL com um máximo teórico de 255. Esse valor é até considerado grande quando vemos que estudos de topologia da internet monstraram que o valor médio da internet raramente excede 30 hops. Como já foi dito, embora o valor do TTL possa ser alterado, ele é um ótimo indício de um SO pois esses seguem padrões pré-definidos de TTL. Linux, macOS/iOS e android têm um padrão de 64 no campo TTL enquanto o windows tem 128. Com isso conseguimos saber que, por exemplo, se um pacote chega na nossa maquina com um TTL de 109 é, provavelmente, advindo de um windows. 

    1. Arquitetura de comunicação digital onde os dados são segmentados em unidades discretas e independentes (pacotes), contendo informações de cabeçalho para roteamento e controle.

## Protocol - 
Esse campo é um identificador numérico de 8 bits que atua como o mecanismo primário de demultiplexação¹ entre a camada de rede e a camada de transporte. No modelo OSI ou TCP/IP, o encapsulamento aninha dados como uma boneca russa. Quando um datagrama IP chega ao seu destino e o cabeçalho IP é processado e removido, o sistema operacional precisa saber o módulo de protocolo apropriado para o qual deve entregar o payload restante. O campo protocol diz quais são os dados brutos que vêm a seguir. Sem este campo, a pilha IP não saberia distinguir se o payload é um segmento TCP, um datagrama UDP, ou só uma mensagem de controle.

Se o valor no campo protocol apontar para um número para o qual o host receptor não possui um handler registrado, o pacote é descartado e a pilha IP gera uma mensagem de erro ICMP de volta ao remetente. Nessa mensagem veremos provavelmente o código 2 para protocol unreachable. Esse mecanismo é fundamental para diagnósticos de rede, informando ao remetente que, embora o host esteja ativo, ele não fala a linguagem de transporte solicitada.

A IANA gerencia a lista de protocol numbers. Na tabela abaixo coloquei alguns dos protocolos organizados pelo seu valor decimal para conseguirmos ver os valores definidos pela IANA para alguns deles, mas no linux é só ir em /etc/protocols caso queira encontrar outro.

|Decimal|Hex|Palavra|Descrição|
|:----:|:----:|:----:|----|
|1|0x01|ICMP|Internet Control Message Protocol|
|2|0x02|IGMP|Internet Group Management Protocol. Gerenciamento de multicast. Alvo comum para ataques de DoS ou mapeamento de rede passivo|
|4|0x04|IP-in-IP|IP Encapsulation within IP. Tunelamento simples. Usado em redes legadas ou configuração de túnel manual|
|6|0x06|TCP|Transmission Control Protocol|
|17|0x11|UDP|User Datagram Protocol|
|41|0x29|IPv6|IPv6 Encapsulation. Não é erro. Ele só indica que o payload do pacote IPv4 é, na verdade, um pacote IPv6 (ex: túneis 6to4, ISATAP)|
|47|0x2F|GRE|Generic Routing Encapsulation. Protocolo de tunelamento não criptografado|
|50|0x32|ESP|Encapsulating Security Payload. IPSec (dados criptografados)|
|51|0x33|AH|Authentication Header. IPSec (apenas integridade/autenticação, sem criptografia)|
|89|0x59|OSPF|Open Shortest Path First. Protocolo de roteamento dinâmico interior gateway protocol|
|255|0xFF|Reserved|Reservado pela IANA. Se aparecer em um sniffer, é provável que seja tráfego gerado customizadamente|

O campo é um dos 5 pilares da tupla de filtragem em firewalls stateless (Os outros 4 são srcIP, dstIP, srcport, dstport). Regras comuns de segurança frequentemente bloqueiam tudo, liberando apenas TCP e UDP. Para casos de pentest onde não se tem acesso prévio as configurações do firewall, vale estudar a rotina da empresa e entender as necessidades. Bloquear indiscriminadamente protocolos não TCP/UDP impede o funcionamento, por exemplo, do ICMP. Isso quebra algumas ferramentas que dependem de mensagens vindas do ICMP ou até mesmo o path mtu discovery, causando conexões que travam misteriosamente ou lentidão. Isso pode causar black holing de conexões. Conhecer o cenário da empresa te faz entender as necessidades de uso dela e possivelmente encontrar brechas nessa maneira de agir de alguns especialistas de cibersegurança que bloqueiam tudo e vão liberando de acordo com as necessidades. Uma técnica para contornar firewalls que filtram com base no campo protocol é o tunelamento, eles encapsulam protocolos dentro de outros. IP-over-UDP, por exemplo, protocolos como QUIC, WireGuard, VXLAN e OpenVPN encapsulam seus payloads dentro de um pacote UDP. Assim, um firewall analisando apenas o campo protocol verá UDP. Ele não consegue saber se dentro daquele UDP tem tráfego web, ou túnel VPN ou tráfego malicioso, a não ser que ele realize DPI (deep packet inspection) para analisar o payload da camada de aplicação.

    1. É o processo lógico onde o sistema operacional utiliza o identificador numérico (campo protocol) para separa o tráfego convergente recebido na camada de rede (IP) e encaminhá-lo ao módulo de software específico da camada superior para processamento.

## Header Checksum -
Esse campo é um componente de 16 bits, projetado com um único propósito, garantir a integridade do próprio cabeçalho durante seu trânsito pela rede. Ele funciona como um selo de verificação, permitindo que cada roteador no caminho valide rapidamente se o cabeçalho foi corrompido por erros de transmissão. É crucial entender que seu escopo é estritamente limitado ao cabeçalho, ele não oferece qualquer proteção para os dados do payload, uma responsabilidade delegada aos protocolos da camada de transporte, como TCP e UDP, que possuem seus próprios mecanismos de checksum.

O algoritmo de cálculo é uma soma de complemento de um (one's complement sum) de todas as palavras de 16 bits do cabeçalho. O processo é o seguinte:
1.  **Inicialização:** O campo header checksum é preenchido com zeros.
2.  **Soma:** O cabeçalho é tratado como uma sequência de inteiros de 16 bits, que são somados.
3.  **Wrap-around:** Se a soma exceder 16 bits, o bit de estouro (carry) é adicionado de volta ao resultado. Este passo é repetido até que não haja mais estouros.
4.  **Inversão:** O resultado final é invertido bit a bit (complemento de um) e inserido no campo.

Quando um roteador recebe o pacote, ele realiza o mesmo cálculo sobre o cabeçalho recebido (incluindo o valor do checksum). Se o cabeçalho estiver intacto, o resultado final da soma (após o wrap-around) será uma sequência de 16 bits todos iguais a 1 (0xFFFF em hexadecimal), o que confirma a integridade. Qualquer outro resultado indica corrupção, e o pacote é descartado.

Uma característica fundamental e intencional do design é a eficiência. Como o campo TTL é decrementado em cada hop, o checksum precisa ser recalculado a cada salto. O algoritmo de complemento de um permite um recálculo incremental extremamente rápido. Em vez de refazer a soma inteira, o roteador pode simplesmente subtrair o valor antigo do TTL, adicionar o novo valor e ajustar o checksum, economizando ciclos de CPU.

Do ponto de vista da cibersegurança, o header checksum é paradoxal. Por um lado, ele é um mecanismo de integridade fraco e facilmente contornável. Como o algoritmo é público e computacionalmente barato, qualquer atacante que modifique o cabeçalho (por exemplo, em um ataque de IP Spoofing) pode simplesmente recalcular o checksum correto para o cabeçalho forjado. Isso torna o checksum inútil para detectar modificações maliciosas intencionais. Um firewall ou IDS nunca pode confiar no checksum como prova de que o cabeçalho é autêntico ou benigno.

Por outro lado, a manipulação do checksum pode ser usada em ataques DoS contra equipamentos de rede. Roteadores e firewalls dependem do processamento rápido de pacotes. Um atacante pode inundar um dispositivo com pacotes que possuem checksums inválidos. Embora o dispositivo simplesmente descarte esses pacotes, o ato de verificar o checksum de cada um consome recursos da CPU. Em altas taxas de pacotes, isso pode sobrecarregar o processador do dispositivo, degradando sua performance para o tráfego legítimo, um ataque conhecido como checksum flood.

Além disso, o campo pode ser explorado para técnicas de fingerprinting. Diferentes sistemas operacionais e dispositivos de rede podem ter implementações de pilha TCP/IP que tratam pacotes com checksums inválidos de maneiras sutilmente distintas. Alguns podem descartar silenciosamente, outros podem registrar um erro, e outros ainda podem ter comportamentos anômalos. Um atacante pode enviar pacotes com checksums incorretos para um alvo e observar a resposta (ou a falta dela) para inferir informações sobre o sistema operacional ou o hardware subjacente.

## Source Address
## Destination Address
## Options
## Padding
Esse campo é frequentemente negligenciado em estudos superficiais, visto apenas como um espaço vazio ou algo assim. Inclusive, em um dos livros usados como fonte aqui, ele nem é mencionado. 

O padding é um campo de tamanho variável inserido no final do cabeçalho IPv4, especificamente após o campo de opções e antes do início do payload, como vemos na imagem que iniciou esse arquivo. A necessidade imperativa deste campo é puramente matemática e arquitetural, derivada da granularidade de processamento das CPUs de 32 bits da época em que o protocolo foi desenhado, na RFC 791. O processamento de memória é otimizado quando os dados estão alinhados em fronteiras de palavras específicas. 

Voltando um pouco, no IPv4, a unidade fundamental de medida para o tamanho do cabeçalho é a palavra de 32 bits. O campo IHL não conta bytes, ele conta palavras de 32 bits, ou seja:

$$\text{Tamanho do Cabeçalho} = IHL \times 32 \text{ bits}$$

Se o campo options tiver um tamanho que não resulte em um múltiplo perfeito de 32 bits, o cabeçalho terminaria, de certa forma, quebrado em relação à memória. O trabalho do padding é preencher essa lacuna para garantir que o cabeçalho sempre termine em uma fronteira de 32 bits, permitindo que o payload comece alinhado. O valor dos bits de preenchimento devem ser 0, como foram definidos pela RFC 791. O campo padding só existe se o campo de opções for utilizado, e o tamanho do padding varia de 0 a 3 bytes.

Gosto de ver o padding como um espaço de armazenamento não monitorado. Como a maioria dos firewalls simples apenas verifica cabeçalhos padrão e ignora conteúdo do padding (assumindo que são zeros), este campo se torna um vetor ideal para esteganografia de rede. Mas vou deixar para dissertar sobre isso no final desse campo. 

Além da ideia citada acima, já li sobre vazamento de memória e OS fingerprinting usando o campo padding. A ideia é, se o sistema operacional aloca memória para o pacote e preenche o cabeçalho, mas esquece de escrever zeros explicitamente no padding, os dados que estavam anteriormente naquela região da memória RAM (ou seja, lixo de memória/kernel heap) são enviados pela rede. Isso pode acabar vazando pedaços de senhas, nomes de usuários ou código interno do kernel. Padrões específicos de como o padding é aplicado podem servir como uma assinatura passiva para identificar sistemas operacionais antigos ou dispositivos embarcados vulneáveis.

Mas aqui pensamos em possibilidades onde a sanitização foi considerada trivial. A regra da RFC é bem restrita com relação ao padding, qualquer bit nesse campo que seja diferente de 0 é uma anomalia. Pessoalmente eu não tenho experiência com suricata IDS, mas li que é bem mais fácil do que imaginei escrever uma regra de inspeção de uma área do cabeçalho IP. Algo do tipo, se o IHL for maior que 5 (o que indica a presença de options/padding), o sistema deve verificar se os bytes finais do cabeçalho são nulos. Ou uma normalização forçada, o dispositivo intermediário de rede intercepta o pacote, analisa o cabeçalho IP e reescreve forçosamente o campo padding para tudo zero antes de encaminhar o pacote.

Voltando para a esteganografia que comentei. 

## Payload