Exercícios de fixação

1. Diferencie varredura de portas e enumeração de serviços em uma frase cada.

Varredura de portas: verifica quais portas de um dispositivo estão abertas, fechadas ou filtradas.
Enumeração de serviços: identifica quais serviços e, quando possível, quais versões estão funcionando nas portas abertas.

2. Por que um SYN scan é considerado mais "discreto" que um Connect scan?
Porque o SYN scan não completa totalmente a conexão TCP: ele envia um pacote SYN e analisa a resposta, enquanto o Connect scan realiza o three-way handshake completo, deixando mais registros nos logs do sistema.

3. O que significa uma porta aparecer como "filtered" em vez de "closed"?
Significa que o scanner não conseguiu determinar se a porta está aberta ou fechada, geralmente porque um firewall, roteador ou alguma regra de segurança está bloqueando ou descartando os pacotes.

4. Explique como o NAT limita o que uma varredura externa consegue enxergar.
O NAT esconde os endereços IP privados dos dispositivos da rede interna, fazendo com que uma varredura externa normalmente enxergue apenas o IP público do roteador e os serviços que foram explicitamente expostos ou redirecionados.

5. Cite duas informações que o banner grabbing pode revelar sobre um serviço.
O banner grabbing pode revelar, por exemplo, o nome do software/serviço utilizado e a sua versão, como Apache 2.4 ou OpenSSH 9.x.
