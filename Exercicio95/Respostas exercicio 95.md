> Exercise 95. A savings account object holds a nonnegative balance, and provides
> deposit(k) and withdraw(k) methods, where deposit(k) adds k to the balance,
> and withdraw(k) subtracts k, if the balance is at least k, and otherwise
> blocks until the balance becomes k or greater.

***

# Primeira Questão
A implementação desse sistema está definido em SavingsAccount1.java e seu caso de teste foi feito em testSavings1() em Main.java. Utilizamos o ReentrantLock dispoível em java.util.concurrent.locks e as condições também disponível no mesmo módulo. Dessa forma, ao tentar fazer uma retirada de valor k, checamos se o valor é menor que o que há disponível na conta, pois, se for, devemos entrar em modo de espera com await(). Além disso, ao fazer um depósito, mandamos as threads acordarem e checarem se ainda está com crédito insuficiente (motivo de precisarmos envolver o bloco por um loop).

# Segunda Questão
A implementação dessa vez conta com duas funções substitutas a withdraw (ordinaryWithdraw e preferredWithdraw). Para garantir que não ocorra ordinaryWithdraw antes que todas as preferredWithdraws tenham sido realizadas, precisamos adicionar uma variável preferredWithdrawCount e checar se ela é 0 quando formos executar o ordinaryWithdraw. Dessa forma, conseguimos implementar o que foi pedido pelo enunciado.

# Terceira Questão
**Não**, não é certo de retornar. Com a implementação que fizemos não podemos ter certeza que cada operação irá retornar.
Isso se dá pelo fato de que uma thread poderá ficar em espera infinita se a conta que ela está esperando ter crédito
nunca receber dinheiro suficiente. 
Podemos confirmar o pensamento se levarmos em conta o "pior caso":
<ul>
    <li> todas as contas tentam transferir saindo de apenas uma conta (vamos chamá-la de conta1, começando com saldo 0) para as outras, o débito máximo que se chegará será 100 * n (quantidade de threads)</li>
    <li> assim, se tivermos um valor de n <= 10, tudo funcionaria bem, pois mesmo que tentássemos retirar somente da conta1, ao receber 1000 da thread Boss, o débito seria realizado e a operação retornaria. </li>
    <li> entretanto, se houver um número de threads > 10, a conta1 ficará devendo eternamente, visto que não seria possível abater o valor.</li>
</ul>
