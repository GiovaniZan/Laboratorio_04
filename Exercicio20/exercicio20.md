# Exercício 20

## Tendo como base o projeto “ gpio_int ”,responda:



1. As interrupções são geradas quando o botão SW1 é pressionado ou quando é solto ? Qual
   configuração em qual registrador determina isso?

   > As interrupções ocorrem quando o botão é pressionado. Como o botão é conectado via uma lógica negativa, a interrupção é ajustada para ocorrer na borda de descida. O bit correspondente da porta é zerado no registrador GPIO Interrupt Event, denotado no programa de  GPIO_IEV

2. Como ocorre a troca de informação entre a ISR “ GPIOJ_Handler ” e o laço do programa
   principal?
   
   > O registrador R11 foi reservado para a interrupção, isto é, é alterado apenas os eventos de interrupção da porta GPIO_ J.
   
3. O que acontece se for utilizado o registrador R12 no programa no lugar do R11? Por que?

> Neste caso o programa não funciona pois na entrada da interrupção, o hardware automaticamente faz o PUSH dos registradores R0, R3, R12, LR, PC e xPSR e realiza o POP na saída da rotina de tratamento. Se alterarmos o R12 dentro do tratamento da interrupção, ele retomará o valor original no retorno para a execução normal.

4. Modifique o programa do projeto “ gpio_int ” para habilitar interrupções aos dois botões
   do kit, de forma que qualquer deles, quando pressionado, incremente a contagem binária
   apresentada nos LEDs do kit

- Quais modificações são necessárias na sub rotina "Button_int_conf" e na "ISR GPIOJ_Handler" ?

> 
>
> Na rotina de configuração de interrupção devemos acrescentar o bit da porta PJ1
>
> `Button_int_conf:`
>         `//MOV R2, #00000001b ; bit do PJ0`
>         `MOV R2, #00000011b ; bit do PJ0 e PJ1`
>         `LDR R1, =GPIO_PORTJ_BASE`
>
> 
>
> Na rotina de tratamento da interrupção devemos acrescentar o bit da porta PJ1 para sinalizar o tratamento deste botão.
>
> `GPIOJ_Handler:`
> `//        MOV R0, #00000001b ; ACK do bit 0`
>         `MOV R0, #00000011b ; ACK do bit 0`

5. Modifique novamente o programa do projeto "gpio_int", de forma que quando o botão
   SW1 for pressionado a contagem binária nos LEDs do kit seja incrementada e quando o
   botão SW2 for pressionado a contagem seja decrementada
   
   - Como é possível identificar a causa da interrupção (botão SW1 ou SW2 ) na ISR GPIOJ_Handler?

> Para fazer com que o outro botão também dispare a iterrupção, devemos modificar os bits de configuração na rotina
>
> 
>
> ```
> Button_int_conf:
> //        MOV R2, #00000001b ; bit do PJ0
>      MOV R2, #00000011b ; bit do PJ0 e PJ1
>      LDR R1, =GPIO_PORTJ_BASE
> // o restante foca inalterado.
> ```
>
> 
>
> Agora ambos botões da porta J disparam a mesma interrupção de porta J.
>
> Podemos identificar o pino responsável pelo disparo da interrupção da porta através do registrador GPIORIS -- GPIO Raw Interrupt Status. Este registrador possui um offset 0x414. 
>
> Na rotina de tratamento da interrupção:
>
> ```
> ; GPIOJ_Handler: Interrupt Service Routine for port GPIO J
> ; Utiliza R11 para se comunicar com o programa principal
> GPIOJ_Handler:
> 
>      LDR R0, [R1, #GPIO_RIS] ; lê o status da interrupção
> 
>      TST R0, #1
>      ITE EQ
>      ADDEQ R11, R11, #1 ; tratamento
>      SUBNE R11, R11, #1
> 
> //        MOV R0, #00000001b ; ACK do bit 0
>      MOV R0, #00000011b ; ACK do bit 0
>      LDR R1, =GPIO_PORTJ_BASE
>      STR R0, [R1, #GPIO_ICR]
> 
>      BX LR ; retorno da ISR
> 
> ```
>
> Modificamos o padrão de bit para limpar os flags de interrupção da porta, para incluir o bit do outro botão.
>
> Em seguida, lemos o registrador GPIO_RIS e testamos para verificar qual dos botões foi acionado, e atuamos de acordo (somando se for o bit 0 e subtraindo se não for o bit zero). Esta implementação trata o evento improvável de dois botões simultâneos como se fosse o botão ligado ao pino 2.



6. Como implementar uma estratégia de debounce dos botões quando utilizando interrupções?

- Observação importante: rotinas de serviço de interrupção devem ter processamento muito
  rápido interrupções no programa principal devem ser breves!

  > Podemos manter um registrador de uso geral como um vetor de bits de controle do programa. Dentro deste registrador podemos alocar alguns bits para determinar que houve modificação do padrão de estado das teclas, alguns bits podem ser utilizados para passar a informação de qual o pino que causou a interrupção, e mesmo se a borda foi de subida ou descida. Como a interrupção deve ser a mais breve possível, tendo a apenas sinalizar que houve interrupção de entrada e executar o tratamento do estado dos botões em uma subrotina, chamada do loop principal de acordo com o flag setado na interrupção. 
  >
  > Podemos fazer o tratamento de debounce dos botões na rotina de tratamento, chamada a partir do loop principal do programa apenas se houver interrupção (mas fora do tratamento da interrupção). Podemos implementar um contador baseado em alguma constante de tempo (eventualmente também disparada por interrupção), e verificar além do botão, se a interrupção ocorreu na borda de subida ou descida ( colocando esta opção na configuração de interrupção da porta J, ou modificando a interrupção para ocorrer na outra borda... ). Particularmente prefiro fazer a interrupção em ambas as bordas, simplesmente sinalizando para o programa que ocorreu uma alteração qualquer na porta dos botões. Reservo um bit para determinar que o debounce está sendo processado e faço o procedimento de debounce - determinação se a borda foi de subida ou descida dentro de uma subrotina, executada [(sempre que houver uma interrupção de botão, OU o debounce ainda não foi resolvido) E flag que marca o tempo foi disparado ]
  >
  > 







