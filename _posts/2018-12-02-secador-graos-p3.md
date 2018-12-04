---
layout: post
title: Secador de Grãos - Parte 3
excerpt_separator: ==
---

### Introdução

Na terceira e última parte desse tutorial, será mostrado o funcionamento do projeto através de dois vídeos da execução. No primeiro o PWM de cada sensor é mostrado pelo LED verde indicador e ocorre uma variação das condições de luminosidade e de temperatura (essa, por ser menor, é pouco visível). Depois uma execução inteira da curva é mostrada.

<!--break-->

### Vídeo 1

O primeiro vídeo está abaixo e nele são mostrados os LEDs indicadores de nível de cada sensor, bem como o LED vermelho mais a esqueda que indica o PWM do motor e o último LED vermelho que indica se o sistema está ligado ou não. O botão de iniciar o processo está em pull up, o que significa que ele envia 5V o tempo todo para a porta do Arduino e quando pressionado, envia 0V.
São testadas variações tanto de luminosidade como de temperatura para ver o LED sendo alterado. A temperatura, como varia mais devagar não é tão perceptível, porém, na interface serial, é possível observar sua mudança.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kWEfS6okCko" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


### Vídeo 2

No segundo vídeo é mostrado o funcionamento do sistema com uma execução completa da curva de secagem. Quando ligado, o botão de iniciar o processo fica desabilitado e só retorna no fim da secagem. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/PyTDrAwF5z0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Finalizando

No projeto original está previsto o uso de um bluetooth que transmite os dados dos sensores e da curva para um aplicativo Android. Entretando, como o bluetooth que possuo não funcionou, resolvi remover essa parte do projeto e deixar apenas a comunicação serial comum funcionando. 
Essa aplicação foi muito interessante para consolidar os conhecimentos sobre o controlador Atmega328p e seus registradores usando a linguagem C. Foram explorados diversos recursos do microcontrolador e o conhecimento e os códigos podem ser reutilizados em outros projetos.

Até o próximo post.


Matheus Pessoa
