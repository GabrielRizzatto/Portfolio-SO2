# Sincronização e Comunicação entre Processos

Quando diferentes processos precisam cooperar de maneira coordenada para realizar uma tarefa comum, eles devem se comunicar e compartilhar informações de forma controlada. O grande problema desse compartilhamento de recursos é o risco de inconsistências conhecidas como **condições de corrida**. Esse fenômeno ocorre quando dois ou mais fluxos de execução tentam ler ou gravar dados em uma área de armazenamento comum, seja ela em memória ou em arquivos, de modo que o resultado final da tarefa dependa diretamente de quem executa em qual momento específico. Para evitar esses incidentes e proteger a integridade dos dados, é fundamental que as ações dos processos concorrentes sejam sincronizadas por meio do princípio da **exclusão mútua**. Esse princípio visa garantir que, enquanto uma tarefa estiver manipulando uma variável ou arquivo comum, todas as demais sejam terminantemente impedidas de realizar a mesma operação.

A porção de um programa que acessa diretamente os dados ou recursos compartilhados é definida como **região crítica** ou **seção crítica**. Uma arquitetura de sincronização eficiente exige que nenhuma dupla de processos ocupe suas respectivas regiões críticas de forma simultânea. Ao mesmo tempo, o sistema de exclusão mútua deve funcionar perfeitamente sem fazer suposições sobre a velocidade de processamento ou o número de unidades de processamento central ativas. Também é imperativo que nenhuma tarefa executando fora de sua área crítica impeça o progresso de outros fluxos de execução, e nenhum processo deve ser forçado a aguardar indefinidamente para obter permissão de entrada em sua própria região crítica.

## Soluções para Exclusão Mútua

*   **Desabilitar interrupções:** Uma das maneiras mais simples de se obter a exclusão mútua é desabilitando as interrupções assim que um processo entra em sua região crítica. Contudo, essa estratégia só funciona em sistemas equipados com um único processador e é altamente perigosa se disponibilizada para programas de usuário, pois uma falha de encerramento poderia travar a máquina inteira.
*   **Alternância estrita:** Utiliza uma variável de controle para determinar a vez de cada participante, mas introduz o problema da **espera ocupada** (frequentemente chamado de *bloqueio por rotação*), em que um processo consome ciclos de processamento continuamente testando uma variável até que ela mude de valor. Além de desperdiçar tempo de processador, permite que uma tarefa muito mais lenta bloqueie sua parceira mais ágil.
*   **Algoritmo de Peterson (1981):** Solução puramente em software que utiliza variáveis indicadoras de interesse para garantir o revezamento justo e seguro sem necessidade de alternância rígida.
*   **Instruções atômicas de hardware:** Muitos computadores fornecem instruções como o *teste e definição de bloqueio* (conhecidas no ecossistema x86 como *trocas atômicas*), que realizam a leitura e a escrita em um endereço de memória de forma indivisível, impedindo interferências externas.

## Primitivas de Comunicação e Sincronização

Apesar de corretas, as técnicas de rotação continuam sendo ineficientes para esperas prolongadas. A alternativa é empregar primitivas de comunicação que bloqueiam os processos quando eles não podem prosseguir, suspendendo sua execução até que um sinal externo os desperte.

*   **Problema do Produtor e Consumidor:** Duas tarefas compartilham um reservatório de capacidade limitada. Se o reservatório estiver cheio, o produtor adormece, e se estiver vazio, o consumidor vai dormir. O grande risco é a perda de sinais de despertar devido a concorrências mal coordenadas, podendo congelar o sistema permanentemente.
*   **Semáforos (Dijkstra, 1965):** Variáveis inteiras capazes de contabilizar os sinais de despertar recebidos para uso futuro. Por meio de operações atômicas denominadas *baixar* e *subir*, os processos alteram esses contadores e adormecem ou acordam sem qualquer chance de interrupção.
*   **Mutex:** Quando a capacidade de contagem do semáforo não é necessária, utiliza-se o *mutex*, uma versão simplificada de apenas um bit que controla se o recurso está bloqueado ou livre. É muito popular em bibliotecas de fluxo de controle em nível de usuário.
*   **Monitores:** Como o uso indevido de semáforos pode causar um **impasse** ou **deadlock** (espera circular mútua e insolúvel), os monitores trazem o gerenciamento de exclusão mútua diretamente para as regras da linguagem de programação. Controlado pelo compilador, o monitor garante de forma automática que apenas um processo execute suas funções internas por vez.

## Outras Abordagens de Coordenação

*   **Troca de Mensagens:** Muito usada em computadores que não dividem a mesma memória física. Utilizando chamadas de sistema para enviar e receber, os processos comunicam-se de forma direta ou indireta através de caixas de correio. Exige mecanismos como numeração sequencial para evitar perdas ou duplicidades.
*   **Barreiras:** Utilizadas para coordenação de grupos maiores de processos. Dividem as tarefas em fases e barram o avanço de qualquer integrante até que todos os outros tenham alcançado o mesmo ponto (útil em simulações científicas).

## Desafios Específicos e Soluções Modernas

*   **Inversão de Prioridades:** Ocorre quando uma tarefa de alta prioridade precisa de um recurso retido por uma de baixa prioridade, mas uma terceira tarefa de prioridade média impede a execução da menor. A solução passa pela aplicação de *herança de prioridade* ou *impulsos aleatórios de prioridade*.
*   **Atualização por Cópia em Leitura (RCU):** Técnica concebida para evitar o uso de bloqueios tradicionais em leituras concorrentes. Permite que leitores acessem estruturas de dados de forma ágil e sem travas, enquanto os escritores duplicam e modificam as informações em áreas de memória isoladas antes de atualizar as referências principais de forma atômica. A memória antiga só é liberada após um *período de carência*, garantindo que todos os leitores ativos tenham abandonado as referências aos dados antigos.
