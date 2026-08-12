# Programação Concorrente

A programação concorrente consiste na execução cooperativa de processos ou _threads_ para realizar uma tarefa comum. Em sistemas com um único processador, o escalonador alterna as execuções rapidamente para criar a ilusão de simultaneidade, enquanto em sistemas com múltiplos processadores o processamento ocorre de forma verdadeiramente paralela.

Como essas unidades de execução compartilham recursos lógicos e físicos, podem ocorrer as **condições de corrida**. Nesse cenário, o acesso simultâneo a um recurso compartilhado faz com que o resultado final dependa de forma imprevisível da ordem das instruções, o que pode comprometer a integridade dos dados.

Para solucionar isso, o sistema operacional sincroniza as execuções por meio da **exclusão mútua**, uma técnica que protege a região crítica do código e garante que apenas um processo acesse o recurso por vez. Um controle falho dessas regras pode gerar problemas graves, como a **inanição**, onde um processo é injustiçado e nunca acessa o recurso, ou o **deadlock**, que causa o bloqueio permanente dos processos em uma espera circular.

Para gerenciar esses bloqueios sem desperdiçar ciclos da CPU com a ineficiente espera ocupada, os sistemas utilizam mecanismos avançados de sincronização. Destacam-se os **semáforos**, que operam variáveis inteiras através de comandos atômicos de decremento (_wait_/_down_) e incremento (_signal_/_up_), e os **monitores**, que são estruturas de alto nível gerenciadas automaticamente pelo próprio compilador.

# Criação e Gerência de Processos

Um processo é definido como um programa em execução, representando a unidade de trabalho básica e ativa gerenciada pelo sistema operacional. Ele serve como o ambiente físico e lógico onde um programa é executado de forma protegida. Para que a concorrência ocorra sem erros, o sistema operacional organiza cada processo em três partes integradas:

- **Contexto de hardware:** preserva o estado dos registradores da CPU, incluindo o contador de programa, que aponta para a próxima instrução a ser executada, e o apontador de pilha, ou _stack pointer_. Essa estrutura viabiliza a mudança de contexto, também chamada de _context switch_, que consiste em salvar os registradores do processo atual e carregar o estado salvo de outro processo diferente, permitindo revezar o uso do processador.
- **Contexto de software:** define características, limites de quotas e atributos do processo, como seu identificador único, chamado de PID, prioridade, limites de memória e descritores de arquivos abertos.
- **Espaço de endereçamento:** determina a área de memória exclusiva de cada processo, contendo as instruções do programa, variáveis globais, a área de alocação dinâmica, ou _heap_, e a pilha, ou _stack_, para variáveis locais e parâmetros.

O controle sistemático dessas informações é organizado e mantido pelo sistema operacional através do Bloco de Controle do Processo, ou **PCB**, localizado na memória principal. Ao longo de sua existência, um processo transiciona ativamente entre estados específicos comandados por eventos de software ou hardware. Ele inicia como **novo** durante sua criação; move-se para o estado de **pronto** quando está carregado em memória aguardando escalonamento; entra em **execução** ao receber tempo de processador; transiciona para o estado de **espera** ou **bloqueio** quando precisa aguardar a conclusão de uma atividade externa, como uma operação de entrada e saída ou recepção de sinal; e, finalmente, alcança o estado de **terminado** ao concluir sua execução e liberar seus recursos do sistema.

A criação de processos pode ocorrer por meio de comandos do usuário ou de chamadas explícitas de rotinas do sistema. Em sistemas UNIX, o processo-pai cria uma duplicata do seu próprio espaço de endereçamento utilizando a chamada de sistema `fork` e carrega o novo executável em memória por meio da chamada `exec`. No ecossistema Windows, a inicialização de um novo executável e de seu contexto limpo de memória é realizada de maneira direta chamando-se a API nativa `CreateProcess`.

# Criação e Gerência de Threads

O conceito de _thread_ ou fluxo de controle surgiu como uma evolução necessária para reduzir o desperdício de tempo e de recursos computacionais envolvidos na criação, eliminação e troca de contexto de processos independentes e pesados. Uma _thread_ atua como a unidade básica de utilização e escalonamento da CPU dentro de um mesmo processo.

Em um ambiente _multithreaded_, ao invés de alocar espaços de memória e PCBs totalmente isolados para cada tarefa concorrente, um único processo hospeda múltiplos fluxos de execução. Essas _threads_ pertencentes ao mesmo processo compartilham o mesmo espaço de endereçamento, o que engloba o código do programa, variáveis globais e a área do _heap_, além dos mesmos recursos de contexto de software, como arquivos abertos e sinais. Todavia, para que cada _thread_ execute suas funções de maneira independente, cada uma mantém de forma exclusiva o seu próprio contexto de hardware privado, composto por um identificador exclusivo, um contador de programa ou _program counter_, conjunto de registradores de CPU e uma pilha ou _stack_ individual para suas variáveis locais. Internamente, o _kernel_ ou a aplicação gerencia esses fluxos por meio de estruturas denominadas Blocos de Controle de Thread ou **TCBs**.

Essa arquitetura de compartilhamento íntimo confere às _threads_ uma vantagem expressiva: criar, chavear e eliminar _threads_ consome drasticamente menos tempo e _overhead_ do que operar com processos tradicionais. Além disso, a troca de informações entre elas é imediata e eficiente, dispensando mecanismos complexos e lentos de comunicação interprocessos.

O suporte e a implementação dessas unidades podem seguir três modelos arquiteturais:

- **Threads em modo usuário (TMUs):** são criadas e gerenciadas por bibliotecas fora do núcleo do sistema operacional, sendo rápidas para criar e alternar, mas com a grande desvantagem de que se uma única _thread_ fizer uma chamada bloqueante, como aguardar entrada e saída, todo o processo e suas demais _threads_ são suspensos.
- **Threads em modo kernel (TMKs):** são escalonadas e reconhecidas diretamente pelo núcleo do sistema, o que viabiliza o paralelismo real de _threads_ do mesmo processo em CPUs _multicore_, embora exijam maior custo computacional para alternar seu contexto devido às transições entre modos de acesso de hardware.
- **Modelos híbridos ou de ativações do escalonador (_scheduler activations_):** buscam equilibrar esses mundos mapeando _threads_ de usuário em processos de peso leve ou **LWPs** associados ao _kernel_ de modo cooperativo.
