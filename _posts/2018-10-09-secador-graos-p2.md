---
layout: post
title: Secador de Grãos - Parte 2
excerpt_separator: ==
---

### Introdução

Vamos agora apresentar a segunda etapa do projeto. Iremos tratar aqui do código utilizado para controlar o secador de grãos. De início é bom informar que o código foi feito em linguagem C para controladores Atmega que é mais complexa do que a linguagem usada pelo Arduino. A programação será explicada e qualquer dúvida que surgir, um email pode ser enviado. 

<!--break-->

### Código-Fonte

Antes de tudo, vale lembrar que apesar de usar muitas simplificações e parecer complexo, o leitor não deve se intimidar. Iremos deixar alguns links com explicações mais completas e a disponibilidade de tirar dúvidas que surgirem pelo email. O código abaixo está dividido em trechos de configurações do controlador.

#### Conversor A/D

A primeira coisa a ser feita é a leitura dos sensores analógicos LDR e NTC que controlam a curva de secagem. Entretando, o microcontrolador não possui capacidade de armazenar o sinal analógico por completo dadas suas características de amplitude infinita inerente a esse tipo de sinal. É necessário então amostrar a entrada para que o controlador possa tratar devidamente os valores discretos de cada leitura. Para fazer isso é usado o conversor A/D interno do Atmega. Abaixo está o código que habilita e realiza leituras dos sinais.

<script src="https://gist.github.com/matheus-pessoa16/8b4f29d28510bd5ae49d6b3c0ac9cb7d.js"></script>

A função 'readSensor(uint8_t sensor)' recebe como parâmetro o número do canal A/D em que o sensor está conectado. Especificamente para esse projeto, a leitura foi limitada apenas a dois canais, o zero e o um.
No canal zero está o sensor de luminosidade e no canal um o sensor de temperatura. O converso A/D está configurado para a resolução de 10 bits e 125 KHz. 

#### Interrupção e PWM

Uma interrupção nada mais é do que uma espécie de chamada de função super prioritária que interrompe o fluxo natural do código salvando o contexto (variáveis, valores e posição do código sendo executada) e desviando para a região da memória que possui o código de tratamento da interrupção. No código abaixo, sempre que o contador 2 contar além do seu limite superior, chamado de overflow, a interrupção irá realizar a manipulação adequada. 

<script src="https://gist.github.com/matheus-pessoa16/439a40e078cf3137afd9be4a49c84cbb.js"></script>

A interrupção de tempo é responsável por calcular os valores da curva de secagem e por atualizar o PWM do motor e dos LEDs que indicam o nível de leitura dos sensores. A chamada da função ISR determina como o tratamento de uma interrupção do tipo overflow do Timer2 vai ser realizado. Existem outros parâmetros que estão disponíveis [aqui] (http://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf) na página 49. A interrupção é executada automaticamente a cada overflow do timer sem a necessidade de ser chamada em nenhum local.
A contagem do tempo juntamente com a execução da curva só iniciarão ao pressionar o botão de ligar. O botão, após iniciada a secagem, fica desabilitado durante todo o processo e é reabilitado no fim.
#### Configuração da Comunicação Serial

A última seção do código trata da configuração da comunicação serial UART. Esse é um padrão de comunicação serial que usa um RX para receber os dados e um TX para transmissão. Como já dito, a comunicação é serial, ou seja, é feito o envio bit a bit, um por vez. No Atmega, existem alguns registradores que são configurados e que são mostrados abaixo.

<script src="https://gist.github.com/matheus-pessoa16/3461660aeb93bf2aabf395b47dd83a07.js"></script>

Aqui, basta chamar a função initUSART() na função main e, sempre que quiser enviar ou receber dado via serial, chamar a respectiva função de escrita ou leitura. No projeto os dados são enviados via serial para um transmissor bluetooth que os transmite para o aplicativo Android que gera gráficos.

Essas são as três partes estruturais do código do projeto. Uma seção para leitura dos dados, outra para tratamento das informações e uma última que realiza comunicação. Como é possível notar, diferente do Arduino em que temos uma função 'setup' e uma função 'loop', no código em C puro seguimos o padrão de ter uma função 'main' e dentro um laço infinito que executa as funções.

Na próxima postagem, que será a última, mostraremos o código e o sistema em funcionamento. 
O código completo está [aqui](https://github.com/matheus-pessoa16/Sistemas-Embarcados/blob/master/Arduino/Secador%20de%20Gr%C3%A3os/secador_graos.c).



Até lá.

Matheus Pessoa


### Referências

[Comunicação serial](https://www.robocore.net/tutoriais/comparacao-entre-protocolos-de-comunicacao-serial.html)

[Comunicação serial](http://newtoncbraga.com.br/index.php/telecom-artigos/1709-tel006.html)

[Interrupções](http://www.avr-tutorials.com/interrupts/about-avr-8-bit-microcontrollers-interrupts)