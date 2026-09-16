# Monitores

Os monitores são mecanismos de sincronização de alto nível desenvolvidos por Brinch Hansen e Hoare com o objetivo de tornar o desenvolvimento de aplicações concorrentes mais simples e menos propenso aos erros comuns no uso de semáforos. Estruturalmente, um monitor é um módulo ou tipo abstrato de dados que encapsula variáveis privadas, estruturas de dados internos e procedimentos de acesso. A característica fundamental dessa estrutura é que o compilador garante automaticamente a exclusão mútua entre os procedimentos declarados, assegurando que apenas um processo por vez esteja ativo executando o código dentro do monitor. Para gerenciar o bloqueio e a coordenação quando um processo não pode prosseguir, utilizam-se variáveis de condição. Por meio das operações de espera (`wait`) e sinalização (`signal`), um processo suspende sua execução até que outro processo emita um sinal liberando a condição esperada.

# Deadlock de Comunicação

Diferente do bloqueio tradicional por recursos físicos do computador, o deadlock de comunicação ocorre em sistemas distribuídos e redes onde múltiplos processos cooperativos sincronizam suas ações trocando mensagens. Nessa arquitetura, a situação de impasse surge quando um processo emissor envia uma mensagem de solicitação a um processo receptor e fica suspenso aguardando uma resposta. Se a mensagem enviada ou a resposta de confirmação for perdida no canal de comunicação, o processo emissor permanece aguardando indefinidamente, enquanto o receptor fica bloqueado esperando por uma nova requisição. Esse cenário representa uma anomalia na sincronização de cooperação entre os participantes. A técnica mais comum para prevenir e quebrar o deadlock de comunicação é o uso de temporizadores (*timeouts*), que forçam o reenvio automático das mensagens caso uma resposta válida não seja recebida dentro do tempo limite estabelecido.

# Livelock

O livelock é uma condição de concorrência na qual dois ou mais processos alteram continuamente seus estados em resposta uns aos outros, mas continuam em execução na CPU sem realizar nenhum progresso prático em suas tarefas. Essa complicação acontece frequentemente quando o sistema tenta evitar um deadlock permitindo que um processo abra mão dos recursos que já possui caso não consiga adquirir o próximo recurso necessário. Se múltiplos processos executam essa mesma lógica de desistência e nova tentativa simultaneamente e na mesma cadência, eles entram em um ciclo infinito de alocação e liberação de recursos. Esse comportamento é comparado à situação em que duas pessoas tentam passar uma pela outra em um corredor estreito e continuam se desviando juntas para o mesmo lado repetidamente, impedindo que ambas avancem.

# Condições para Ocorrência de Deadlock

A ocorrência de um deadlock de recursos em um sistema operacional exige a presença simultânea de quatro condições fundamentais:

* **Exclusão Mútua:** Os recursos envolvidos são mantidos em modo não compartilhável, ou seja, cada recurso só pode estar alocado a um único processo por vez.
* **Posse e Espera:** Um processo retém um ou mais recursos previamente concedidos enquanto permanece aguardando a alocação de novos recursos mantidos por outros processos.
* **Não Preempção de Recursos:** O sistema operacional não pode retirar forçadamente um recurso que já foi atribuído a um processo; a liberação ocorre apenas voluntariamente pelo processo proprietário ao finalizar sua tarefa.
* **Espera Circular:** Estabelece-se uma cadeia ou ciclo fechado de processos na qual cada participante aguarda por um recurso alocado ao processo seguinte da lista, com o último processo do ciclo esperando pelo recurso mantido pelo primeiro.
