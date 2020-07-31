# Qiskit_transpile_passManager
Demonstração de transpile e pass manager no qiskit

Original do texto em: https://informacaoquantica.wordpress.com/2020/07/31/o-transpiler-e-passmanager-no-qiskit/

No Qiskit, podemos criar circuitos que atendam a nossa necessidade, como o seguinte.
![](https://informacaoquantica.files.wordpress.com/2020/07/circ01.png)


Porém, não necessariamente o hardware quântico vai ter exatamente as portas lógicas que suportam tais operações.

O que o hardware vai ter são algumas portas lógicas básicas, e todas as que utilizamos são transformadas em portas equivalentes.

Em termos físicos, a porta lógica é um laser, um pulso eletromagnético, algo assim, que modifica o qubit de alguma forma (uma rotação por algum dos eixos, por exemplo).

O transpiler é a ferramenta que faz a tradução do circuito genérico para um que pode ser executado num dispositivo específico.


Este deve aproveitar ao máximo os pontos fortes do dispositivo e minimizar a influência de erros.

O transpiler é como se fosse o compilador da computação tradicional.

Exemplo. O código a seguir cria o circuito acima e o traduz em portas u3 e cx:

from qiskit import QuantumCircuit
from qiskit.compiler import transpile
from qiskit.transpiler import PassManager
from qiskit.transpiler.passes import Unroller

circ = QuantumCircuit(3)
circ.h(0)
circ.cx(0,1)
circ.cx(0,2)

pass1 = Unroller(['u3','cx'])
pm = PassManager(pass1)

new_circ = pm.run(circ)

print(new_circ)

![](https://informacaoquantica.files.wordpress.com/2020/07/circ02.png)

Outro exemplo. Se eu quiser desenrolar em portas u1, u2 e cx:

pass1 = Unroller(['u1','u2','cx'])
![](https://informacaoquantica.files.wordpress.com/2020/07/circ03.png)

Sobre otimização de circuito

Como cada qubit é extremamente caro nos dias de hoje, otimizar cada operação também se torna importante.

Ocorreu recentemente até um desafio da IBM, para novas ideias de como fazer a otimização.

O PassManager faz a tradução entre o layout físico do hardware e a necessidade do circuito.

É layout físico é como o da foto abaixo, retirado do Summer School do Qiskit.

Destaquei o circuito de um qubit.

![](https://informacaoquantica.files.wordpress.com/2020/07/qiskitqubitscircuit.jpg)

Cada qubit tem ligação com poucos outros qubits.

O mapa da ligação com outros qubits é dado por um coupling map.

coupling = [[0,1],[1,2],[2,3]]

A lista acima diz que o qubit 0 tem ligação com o 1, o 1 com o 2, etc.

Portanto, para fazer um cnot entre o 0 e o 3, o que deve ser feito é o qubit 0 trocar com o 1, depois trocar com o 2, para então fazer o cnot com o 3.

Seguem alguns exemplos de código com o PassManager.

from qiskit import QuantumCircuit
from qiskit.compiler import transpile
from qiskit.transpiler import PassManager
from qiskit.transpiler import CouplingMap, Layout
from qiskit.transpiler.passes import BasicSwap, LookaheadSwap, StochasticSwap

circ = QuantumCircuit(3)
circ.h(0)
circ.cx(0,1)
circ.cx(0,2)

coupling = [[0,1],[1,2]]

coupling_map = CouplingMap(couplinglist = coupling)

bs = BasicSwap(coupling_map = coupling_map)
pass_manager = PassManager(bs)
basic_circ = pass_manager.run(circ)

![](https://informacaoquantica.files.wordpress.com/2020/07/circ04.png)

O primeiro esquema é o circuito original. O segundo é após o PassManager. Note que há um swap entre q0 e q1.

O circuito acima é muito fácil. Vejamos com um mais complicado.

O código está no Github (https://github.com/asgunzi/Qiskit_transpile_passManager), porque ficou grande para descrever aqui.

O circuito original é o primeiro, e a decomposição mais básica, o segundo, dado o coupling = [[0,1],[1,2],[0,3]]
![](https://informacaoquantica.files.wordpress.com/2020/07/circ05.png)


Note que a decomposição básica utiliza muitas portas. Há outros tipos de decomposição, como o LookaheadSwap e o StochasticSwap, demonstrados abaixo.
![](https://informacaoquantica.files.wordpress.com/2020/07/circ06.png)


Achar a melhor configuração é um problema NP-Hard, então não espere que este seja um problema fácil. Por outro lado, é uma fonte de oportunidades para melhorar os circuitos existentes.

Links:


https://ibmqawards.com/developer-challenge-circuit-optimization/

https://qiskit.org/documentation/tutorials/circuits_advanced/4_transpiler_passes_and_passmanager.html

