# Exercício 19



- Considere a porta GPIO J, aquela em que
  estão ligados os botões do kit EK TM4C1294XL

1.  Consulte a Tabela 2 9 (p. 116) do manual do
   TM4C1294 e descubra o número da IRQ da
   porta GPIO J
2.  Examine a Seção 3.4 (p. 154) do manual e
   descubra quais registradores de controle
   do NVIC referem se à IRQ da porta GPIO J
3. Determine qual é o bit dos registradores de
   controle que refere se à IRQ da porta GPIO J
4. Examine a Seção 3.4 (p. 159) do manual e
   descubra qual registrador de prioridade do
   NVIC refere se à IRQ da porta GPIO J
5. Determine quais são os bits do registrador
   de prioridade que referem se à IRQ da porta
   GPIO J

1- A porta GPIOJ corresponde ao IRQ 67

2 - Registrador de controle: Register 6: Interrupt 64-95 Set Enable (EN2), offset 0x108

​     Register 10: Interrupt 64-95 Clear Enable (DIS2), offset 0x188

​     Register 14: Interrupt 64-95 Set Pending (PEND2), offset 0x208

​     Register 18: Interrupt 64-95 Clear Pending (UNPEND2), offset 0x288

​     Register 22: Interrupt 64-95 Active Bit (ACTIVE2), offset 0x308

 3 - o IRQ 67 corresponde ao bit 2 do registrador EN2

4 - O registrador de prioridade é 

 Register 40: Interrupt 64-67 Priority (PRI16), offset 0x440

5 - bits 29, 30 e 31 do registrador 40 definem a prioridade da interrupção da porta J