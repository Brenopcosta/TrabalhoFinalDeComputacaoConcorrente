A ideia geral por trás das duas implementações é um banheiro compartilhado, duas
salas de espera, uma para homens e outra para mulheres, e dois corredores levando
 cada uma das salas ao banheiro. A estrutura do algorítmo é a seguinte:
- quando alguém chega:
  - enquanto seu corredor estiver bloqueado:
    - bloqueia o corredor da outra sala
    - espera sinal
  - entra no banheiro

- quando alguém sai do banheiro:
  - se for o último a sair:
    - desbloqueia o corredor da outra sala
  - sinaliza a outra sala de espera

** Implementação com Locks:
*** Exclusão Mútua
Para ver que a solução satisfaz a exclusão mútua, basta imaginar que há um
homem e uma mulher dentro do banheiro ao mesmo tempo e "voltar no tempo". Como
o protocolo manda, caso para que alguém esteja dentro do banheiro, os três
eventos a seguir aconteceram em ordem: a pessoa encontrou seu corredor aberto,
a pessoa fechou o corredor da outra sala, a pessoa entrou no banheiro. Além
disso, como essas três coisas acontecem quando a pessoa está com o lock, elas
podem ser pensadas como atômicas. Assim, quando um homem entra, ele encontra
seu caminho aberto e fecha o das mulheres. E quando uma mulher entra, ela
encontra seu caminho aberto e fecha o dos homens. Como para entrar é
necessário segurar o lock, um deve entrar antes do outro. Assim, se um entra,
o corredor do outro está fechado.

*** Starvation Freedom
Para ver que a solução é starvation free, vamos imaginar que uma mulher está
usando o banheiro. Nesse caso, para que um homem H sofra starvation, todas as
pessoas que chegarem depois devem conseguir entrar na frente dele. Quando H
chega, ele encontra seu corredor fechado e portanto espera. Caso o corredor
das mulheres esteja aberto, todas as mulheres que chegarem entram na sua
frente. Contudo, após a primeira mulher sair, ela notifica todos os homens,
inclusive H e algum deles, quando conseguir o lock, vai fechar o corredor
feminino. A partir desse momento, nenhuma outra mulher vai conseguir entrar,
já que seu corredor só será aberto quando algum homem sair do banheiro. Quando
a última mulher sair, ela abrirá o corredor masculino e todos os homens que
conseguirem o lock entrarão até que alguma mulher acorde e feche o corredor
masculino. Assim, a única possível fonte de starvation seria na disputa pelos
locks. Por isso usamos um ReentrantLock com parâmetro de justiça (construída
com ReentrantLock(true)). Esse lock favorece quem está esperando há mais tempo.


** Implementação com Monitores:
*** Exclusão Mútua
Assim como na implementação usando locks, os mesmos três eventos devem
acontecer para que alguém entre no banheiro. Como os métodos de entrada são
sincronizados usando a instância do banheiro como monitor, apenas um deles
pode ser executado por vez, de modo que podemos pensar nos três eventos
ocorrendo atomicamente. Assim, o mesmo raciocínio vale.
*** Starvation Freedom
Vamos novamente pensar que há uma mulher no banheiro e que um homem H que está
esperando vai sofrer o starvation. Nesse caso, como antes, todas as mulheres
que chegarem enquanto o corredor feminino estiver aberto vão conseguir entrar.
Como toda mulher que sai notifica o monitor de espera masculino, algum homem
irá acordar e fechar o acesso para as mulheres. Assim, após a última mulher
sair, todos os homens em espera entrarão no banheiro.



** Desafios Encontrados
- Conceitualmente, esse problema é bastante similar ao
  FairReadWriteLock. O maior desafio foi encontrar uma maneira de
  fazer homens e mulheres bloquearem o avanço dos outros sem que isso
  gerasse deadlock. Em uma primeira tentativa, colocamos fizemos todas
  as chegadas de uma categoria bloquearem o acesso da outra. Contudo,
  isso pode gerar deadlocks, como quando uma mulher sai, sinaliza para
  os homens que estão esperando e outra mulher chega antes que os
  homens consigam entrar. Essa mulher vai encontrar seu acesso fechado
  e vai fechar o acesso dos homens que estão esperando, deixando todos
  esperando para entrar em um banheiro vazio. Assim, a solução que
  encontramos, de uma pessoa ser benevolente e deixar todos da outra
  categoria passarem na sua frente até que alguém da outra categoria
  também seja benevolente e ceda a vez, não nos pareceu muito intuitiva.

- O uso do parâmetro de justiça do ReentrantLock ajudou bastante a
  pensar como evitar starvation entre threads do mesmo tipo.

- A melhor solução que conseguimos usando synchronized usa dois
  monitores, cada uma para um conjunto de espera. Contudo, dois
  monitores implicam em dois locks distintos. Assim, também usamos o
  próprio banheiro como monitor, fazendo os métodos serem synchronized.

