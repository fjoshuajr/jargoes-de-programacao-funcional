# Jargões de Programação Funcional

Esta é uma tradução em Português (pt-PT) de [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon).

A programação funcional (PF) tem várias vantagens, por isso a sua popularidade tem aumentado. No entanto, cada paradigma de programação vem com o seu próprio jargão e a PF não é exceção. Ao dar este glossário, esperamos tornar o aprendizado da PF mais fácil.

Os exemplos são apresentados em JavaScript (versão ES2015). [Why JavaScript? (em inglês)](https://github.com/hemanth/functional-programming-jargon/wiki/Why-JavaScript%3F)

Onde aplicável, este documento usa termos definidos em [Fantasy Land spec (em inglês)](https://github.com/fantasyland/fantasy-land)

__Tabela de Conteúdos__
<!-- RM(noparent,notop) -->
* [Aridade](#aridade)
* [Funções de Ordem-Superior (FOS)](#funções-de-ordem-superior-fos)
* [Closure](#closure)
* [Aplicação Parcial](#aplicação-parcial)
* [Currying](#currying)
* [Currying Automático](#currying-automático)
* [Composição de Funções](#composição-de-funções)
* [Continuação](#continuação)
* [Puridade](#puridade)
* [Efeitos Secundários](#efeitos-secundários)
* [Idempotência](#idempotência)
* [Estilo Livre de Apontamento](#estilo-livre-de-apontamento)
* [Predicado](#predicado)
* [Contratos](#contratos)
* [Categoria](#categoria)
* [Valor](#valor)
* [Constante](#constante)
* [Functor](#functor)
* [Functor Apontado](#functor-apontado)
* [Elevação](#elevação)
* [Transparência Referencial](#transparência-referencial)
* [Raciocínio Equacional](#raciocínio-equacional)
* [Lambda](#lambda)
* [Cálculo Lambda](#cálculo-lambda)
* [Avaliação Preguiçosa](#avaliação-preguiçosa)
* [Monoid](#monoid)
* [Monad](#monad)
* [Comonad](#comonad)
* [Functor Aplicável](#functor-aplicável)
* [Morfismo](#morfismo)
  * [Endomorfismo](#endomorfismo)
  * [Isomorfismo](#isomorfismo)
  * [Homomorfismo](#homomorfismo)
  * [Catamorfismo](#catamorfismo)
  * [Anamorfismo](#anamorfismo)
  * [Hylomorfismo](#hylomorfismo)
  * [Paramorfismo](#paramorfismo)
  * [Apomorfismo](#apomorfismo)
* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Foldable](#foldable)
* [Lens](#lens)
* [Type Signatures](#type-signatures)
* [Algebraic data type](#algebraic-data-type)
  * [Sum type](#sum-type)
  * [Product type](#product-type)
* [Option](#option)
* [Function](#function)
* [Partial function](#partial-function)
* [Functional Programming Libraries in JavaScript](#functional-programming-libraries-in-javascript)
<!-- /RM -->

## Aridade

O número de argumentos que uma função leva. De palavras como unário/a, binário/a, ternário/a, etc. Estas palavras distinguem-se por serem compostas pelo sufixo, "-ário/a". A adição, por exemplo, leva dois argumentos, e por essa razão é definida com uma função binária com uma aridade de dois. Tal função pode também ser chamada de "diádica" por pessoas que preferem origens Gregas em vez de Latinas. Da mesma forma, uma função que toma um número variado de argumentos é chamada de "variádica", enquanto que uma função binária deve ser dada dois e somente dois argumentos, mesmo que se faça currying e aplicação parcial (veja abaixo).

```js
const sum = (a, b) => a + b

const arity = sum.length
console.log(arity) // 2

// A aridade de sum é 2
```

## Funções de Ordem-Superior (FOS)

Uma função que leva uma função como argumento e/ou retorna uma função.

```js
const filter = (predicate, xs) => xs.filter(predicate)
```

```js
const is = (type) => (x) => Object(x) instanceof type
```

```js
filter(is(Number), [0, '1', 2, null]) // [0, 2]
```

## Closure

Um closure é uma maneira de acessar uma variável fora do seu escopo.
Formalmente, um closure é uma técnica para implementar um vínculo nomeado com escopo lexicamente.
É uma forma de guardar uma função com o seu ambiente.

Um closure é um escopo que captura variáveis locais de uma função para que possam ser acessadas até mesmo se a execução for feita fora do bloco no qual este foi definido. Exemplo: permite que se faça referência ao escopo depois que o bloco no qual as variáveis foram declaradas tenha terminado de executar.

```js
const addTo = x => y => x + y;
var addToFive = addTo(5);
addToFive(3); // retorna 8
```

A função ```addTo()``` retorna uma função (internamente chamada ```add()```), vamos guardá-la numa variável chamada ```addToFive``` com uma invocação com currying com argumento _5_.

Normalmente, quando a função ```addTo``` termina de executar, o seu escopo, com variáveis locais _add_, _x_, _y_ não pode ser acessado. Mas, a função retorna _8_ quando se invoca ```addToFive()```. Isto quer dizer que o estado da função ```addTo``` continua guardado mesmo depois que o bloco de código tenha terminado de executar, caso contrário não haveria forma de saber que ```addTo```  foi invocada como ```addTo(5)``` e o valor de _x_ foi definido para _5_.

Escopo léxico é a razão pela qual a função é capaz de encontrar os valores de _x_ e de _add_ - as variáveis privadas to pai que terminou de executar. Este valor é chamado de Closure.

A pilha juntamente com o escopo léxico da função é guardada na forma de referência para o pai. Isso previne que o closure e respetivas variáveis sejam coletadas durante o _garbage collection_ (já que existe pelo menos uma referência ativa para o pai).

Lamda Vs Closure: Um lambda é basicamente uma função que é definida em linha em vez do metódo normal de declaração de funções. Lambdas podem frequentemente ser passados entre funções tal como objetos.

Um closure é uma função que envolve o seu estado através da referência de campos fora do seu escopo. O estado envolvido continua a existir durante invocações do closure.

__Leitura adicional/Fontes__

* [Lambda Vs Closure](http://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda)
* [JavaScript Closures highly voted discussion](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)

## Aplicação Parcial

Aplicar parcialmente uma função é criar uma nova função através do pré-preenchimento de alguns dos seus argumentos até a função original.

```js
// Auxiliar para criar funções aplicadas parcialmente
// Leva uma função e alguns argumentos
const partial = (f, ...args) =>
  // retorna a função que leva os argumentos restantes
  (...moreArgs) =>
    // e invoca a função original com todos eles
    f(...args, ...moreArgs)

// Algo para aplicar
const add3 = (a, b, c) => a + b + c

// Aplicar parcialmente `2` e `3` para `add3` dá-nos uma função com um argumento
const fivePlus = partial(add3, 2, 3) // (c) => 2 + 3 + c

fivePlus(4) // 9
```

Podemos também usar `Function.prototype.bind` para aplicar parcialmente uma função em JS:

```js
const add1More = add3.bind(null, 2, 3) // (c) => 2 + 3 + c
```

A aplicação parcial ajuda-nos a criar funções mais simples a partir de funções mais complexas pela entrega dos dados quando estiverem disponíveis. Funções [curry](#currying) são automaticamente aplicadas parcialmente.

## Currying

O processo de converter uma função que leva vários argumentos para uma função que leva um argumento de cada vez.

Cada vez que a função é invocada, ela apenas aceita um argumento e retorna uma função que leva um argumento até que todos argumentos sejam passados.

```js
const sum = (a, b) => a + b

const curriedSum = (a) => (b) => a + b

curriedSum(40)(2) // 42

const add2 = curriedSum(2) // (b) => 2 + b

add2(10) // 12

```

## Currying Automático

Transformar uma função que leva vários argumentos para uma que, se dada menos que o número correto de argumentos, retorna uma função que leva os argumentos restantes. A função é analisada no momento em que possui o número correto de argumentos.

lodash e Ramda têm a função `curry` que funciona dessa maneira.

```js
const add = (x, y) => x + y

const curriedAdd = _.curry(add)
curriedAdd(1, 2) // 3
curriedAdd(1) // (y) => 1 + y
curriedAdd(1)(2) // 3
```

__Leitura adicional__

* [Favoring Curry](http://fr.umio.us/favoring-curry/)
* [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)

## Composição de Funções

O acto de juntar duas funções para formar uma terceira função, onde a saída de uma função é a entrada de outra.

```js
const compose = (f, g) => (a) => f(g(a)) // Definição
const floorAndToString = compose((val) => val.toString(), Math.floor) // Uso
floorAndToString(121.212121) // '121'
```

## Continuação

Num dado ponto de um programa, a parte do código que ainda está por ser executada é chamada de continuação.

```js
const printAsString = (num) => console.log(`Recebido ${num}`)

const addOneAndContinue = (num, cc) => {
  const result = num + 1
  cc(result)
}

addOneAndContinue(2, printAsString) // 'Recebido 3'
```

Continuações são geralmente encontradas em programação assíncrona quando o programa precisa esperar para receber dados antes de prosseguir. A resposta é geralmente passada para o resto do programa, que é a continuação, no momemto em que é recebida.

```js
const continueProgramWith = (data) => {
  // Continua o programa com os dados
}

readFileAsync('path/to/file', (err, response) => {
  if (err) {
    // Tratar o erro
    return
  }
  continueProgramWith(response)
})
```

## Puridade

Uma função é pura se o valor de retorno é determinado apenas pelos seus valores de entrada e não efeitos secundários.

```js
const greet = (name) => `Olá, ${name}`

greet('Brianne') // 'Olá, Brianne'
```

Ao contrário dos seguintes exemplos:

```js
window.name = 'Brianne'

const greet = () => `Olá, ${window.name}`

greet() // "Olá, Brianne"
```

A saída do exemplo acima é baseada em dados guardados fora da função...

```js
let greeting

const greet = (name) => {
  greeting = `Olá, ${name}`
}

greet('Brianne')
greeting // "Olá, Brianne"
```

... e este modifica o estado fora da função.

## Efeitos Secundários

Diz-se que uma função ou expressão tem efeitos secundários se além de retornar um valor, ela interage (lê ou escreve) com um estado mutável externo.

```js
const differentEveryTime = new Date()
```

```js
console.log('IO é um efeito secundário!')
```

## Idempotência

Uma função é idempotente se ao reaplicá-la com o seu resultado não produz um resultado diferente.

```
f(f(x)) ≍ f(x)
```

```js
Math.abs(Math.abs(10))
```

```js
sort(sort(sort([2, 1])))
```

## Estilo Livre de Apontamento

Escrever funções onde a definição não identifica explicitamente os argumentos usados. Este estilo normalmente requer [currying](#currying) ou [funções de Ordem-Superior (FOS)](#funções-de-ordem-superior-fos). Mais conhecido por programação tácita.

```js
// Dado
const map = (fn) => (list) => list.map(fn)
const add = (a) => (b) => a + b

// Então

// Não é livre de apontamento - `numbers` é um argumento explícito
const incrementAll = (numbers) => map(add(1))(numbers)

// Livre de apontamento - Esta lista é um argumento implícito
const incrementAll2 = map(add(1))
```

`incrementAll` identifica e utiliza o parâmetro `numbers`, então não é livre de apontamento. `incrementAll2` é escrita apenas pela combinaçao de funções e valores, sem fazer menção dos seus argumentos. __É__ livre de apontamento.

Definições de funções livre de apontamento parecem-se com atribuições normais sem `function` ou `=>`.

## Predicado

Um predicado é uma função que retorna verdadeiro ou falso para um dado valor. Um uso comum de um predicado é como o _callback_ para um filtro de array.

```js
const predicate = (a) => a > 2

;[1, 2, 3, 4].filter(predicate) // [3, 4]
```

## Contratos

Um contrato especifica as obrigações e garantias do comportamento de uma função ou expressão durante a execução. Isso age como um conjunto de regras que são esperadas na entrada e saída de uma função ou expressão, e o erros são geralmente relatados sempre que um contrato é violado.

```js
// Define o nosso contrato : int -> boolean
const contract = (input) => {
  if (typeof input === 'number') return true
  throw new Error('Contrato violado: esperado int -> boolean')
}

const addOne = (num) => contract(num) && num + 1

addOne(2) // 3
addOne('uma string') // Contrato violado: esperado int -> boolean
```

## Categoria

Uma categoria em teoria de categorias é um conjunto de objetos e morfismos entre eles. Em programaçào, geralmente os tipos agem como objetos e funções como morfismos.

Para que uma categoria seja válida, três regras devem ser cumpridas:

1. Deve existir um morfismo identidade que mapeia um objeto para si próprio.

    Onde `a` é um objeto em alguma categoria, deve existir uma função de `a -> a`.

2. Morfismos devem compor.

    Onde `a`, `b` e `c` são objetos em alguma categoria, e `f` é um morfismo de `a -> b` e `g` é um morfismo de `b -> c`; `g(f(x))` deve ser equivalente à `(g • f)(x)`.

3. Composição deve ser associativa.

    `f • (g • h)` é o mesmo que `(f • g) • h`.

Já que essas regras governam a composiçao num nível abstrato, a teoria de categorias é fantástica na descoberta de novas formas de compor coisas.

__Leitura adicional__

* [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

## Valor

Qualquer coisa que pode ser atribuída à uma variável.

```js
5
Object.freeze({name: 'John', age: 30}) // A função `freeze` força a imutabilidade.
;(a) => a
;[1]
undefined
```

## Constante

Um variável que não pode ser reatribuída uma vez definida.

```js
const five = 5
const john = Object.freeze({name: 'John', age: 30})
```

Constantes são [transparentes referencialmente](#transparência-referencial). Isto é, podem ser substituídos pelos valores que representam sem afetar o resultado.

Com as duas constantes acima, a seguinte expressão sempre vai retornar `true`.

```js
john.age + five === ({name: 'John', age: 30}).age + (5)
```

## Functor

Um objeto que implementa a função `map` que, ao passar por cada valor dentro do objeto para produzir um novo objeto, adere à duas regras:

__Preserva a identidade__

```
object.map(x => x) ≍ object
```

__Pode ser composta__

```
object.map(compose(f, g)) ≍ object.map(g).map(f)
```

(`f`, `g` são funções arbitrárias)

Um functor comum em JavaScript é o `Array` já que obedece as duas regras de um functor:

```js
;[1, 2, 3].map(x => x) // = [1, 2, 3]
```

e

```js
const f = x => x + 1
const g = x => x * 2

;[1, 2, 3].map(x => f(g(x))) // = [3, 5, 7]
;[1, 2, 3].map(g).map(f)     // = [3, 5, 7]
```

## Functor Apontado

Um objeto com uma função `of` que coloca _qualquer_ valor único nele.

ES2015 adiciona `Array.of` tornando arrays functores apontado.

```js
Array.of(1) // [1]
```

## Elevação

Elevação é quando se leva um valor e se coloca dentro de um objeto como um [functor](#functor-apontado). Se fizeres elevação de uma função para um [functor aplicável](#functor-aplicável) então podes fazê-lo funcionar em valores que estão também nesse functor.

Algumas implementações têm uma função chamada `lift` ou `liftA2` para tornar fácil a execução de funções dentro de functors.

```js
const liftA2 = (f) => (a, b) => a.map(f).ap(b) // note que é `ap` e não `map`.

const mult = a => b => a * b

const liftedMult = liftA2(mult) // esta função agora funciona em functors como array

liftedMult([1, 2], [3]) // [3, 6]
liftA2(a => b => a + b)([1, 2], [3, 4]) // [4, 5, 5, 6]
```

Elevar uma função de um argumento e aplicá-la faz o mesmo que `map`.

```js
const increment = (x) => x + 1

lift(increment)([2]) // [3]
;[2].map(increment) // [3]
```

## Transparência Referencial

Uma expressão que pode ser substituída pelo seu valor sem mudar o comportamento do programa diz-se que é referencialmente transparente.

Digamos que temos a função _greet_:

```js
const greet = () => 'Hello World!'
```

Qualquer invocação de `greet()` pode ser substituída por `Hello World!` daí que _greet_ referencialmente transparente.

## Raciocínio Equacional

Quando uma aplicação é composta de expressões e sem efeitos secundários, as verdades sobre o sistema podem ser derivadas das partes.

## Lambda

Uma função anônima que pode ser tratada como um valor.

```js
;(function (a) {
  return a + 1
})

;(a) => a + 1
```

Lambdas são geralmente passados como argumentos para [funções de ordem superior](#funções-de-ordem-superior-fos).

```js
;[1, 2].map((a) => a + 1) // [2, 3]
```

Podes atribuir a lambda à uma variável.

```js
const add1 = (a) => a + 1
```

## Cálculo Lambda

Um ramo da matemática que usa funções para criar um [modelo universal de computação](https://pt.wikipedia.org/wiki/Cálculo_lambda).

## Avaliação Preguiçosa

A avaliação preguiçosa é um mecanismo de avaliação por necessidade que atrasa a avaliação de uma expressão até que seu valor seja necessário. Nas linguagens funcionais, isso permite estruturas como listas infinitas, que normalmente não estariam disponíveis em uma linguagem imperativa na qual a sequência de comandos é significativa.

```js
const rand = function*() {
  while (1 < 2) {
    yield Math.random()
  }
}
```

```js
const randIter = rand()
randIter.next() // Cada execução gera um valor aleatório, a expressão é avaliada conforme necessário.
```

## Monoid

An object with a function that "combines" that object with another of the same type.

One simple monoid is the addition of numbers:

```js
1 + 1 // 2
```

In this case number is the object and `+` is the function.

An "identity" value must also exist that when combined with a value doesn't change it.

The identity value for addition is `0`.

```js
1 + 0 // 1
```

It's also required that the grouping of operations will not affect the result (associativity):

```js
1 + (2 + 3) === (1 + 2) + 3 // true
```

Array concatenation also forms a monoid:

```js
;[1, 2].concat([3, 4]) // [1, 2, 3, 4]
```

The identity value is empty array `[]`

```js
;[1, 2].concat([]) // [1, 2]
```

If identity and compose functions are provided, functions themselves form a monoid:

```js
const identity = (a) => a
const compose = (f, g) => (x) => f(g(x))
```

`foo` is any function that takes one argument.

```
compose(foo, identity) ≍ compose(identity, foo) ≍ foo
```

## Monad

A monad is an object with [`of`](#pointed-functor) and `chain` functions. `chain` is like [`map`](#functor) except it un-nests the resulting nested object.

```js
// Implementation
Array.prototype.chain = function (f) {
  return this.reduce((acc, it) => acc.concat(f(it)), [])
}

// Usage
Array.of('cat,dog', 'fish,bird').chain((a) => a.split(',')) // ['cat', 'dog', 'fish', 'bird']

// Contrast to map
Array.of('cat,dog', 'fish,bird').map((a) => a.split(',')) // [['cat', 'dog'], ['fish', 'bird']]
```

`of` is also known as `return` in other functional languages.
`chain` is also known as `flatmap` and `bind` in other languages.

## Comonad

An object that has `extract` and `extend` functions.

```js
const CoIdentity = (v) => ({
  val: v,
  extract () {
    return this.val
  },
  extend (f) {
    return CoIdentity(f(this))
  }
})
```

Extract takes a value out of a functor.

```js
CoIdentity(1).extract() // 1
```

Extend runs a function on the comonad. The function should return the same type as the comonad.

```js
CoIdentity(1).extend((co) => co.extract() + 1) // CoIdentity(2)
```

## Applicative Functor

An applicative functor is an object with an `ap` function. `ap` applies a function in the object to a value in another object of the same type.

```js
// Implementation
Array.prototype.ap = function (xs) {
  return this.reduce((acc, f) => acc.concat(xs.map(f)), [])
}

// Example usage
;[(a) => a + 1].ap([1]) // [2]
```

This is useful if you have two objects and you want to apply a binary function to their contents.

```js
// Arrays that you want to combine
const arg1 = [1, 3]
const arg2 = [4, 5]

// combining function - must be curried for this to work
const add = (x) => (y) => x + y

const partiallyAppliedAdds = [add].ap(arg1) // [(y) => 1 + y, (y) => 3 + y]
```

This gives you an array of functions that you can call `ap` on to get the result:

```js
partiallyAppliedAdds.ap(arg2) // [5, 6, 7, 8]
```

## Morfismo

Uma função de transformação.

### Endomorfismo

Uma função cujo tipo de entrada é o mesmo que a saída.

```js
// uppercase :: String -> String
const uppercase = (str) => str.toUpperCase()

// decrement :: Number -> Number
const decrement = (x) => x - 1
```

### Isomorfismo

A pair of transformations between 2 types of objects that is structural in nature and no data is lost.

For example, 2D coordinates could be stored as an array `[2,3]` or object `{x: 2, y: 3}`.

```js
// Providing functions to convert in both directions makes them isomorphic.
const pairToCoords = (pair) => ({x: pair[0], y: pair[1]})

const coordsToPair = (coords) => [coords.x, coords.y]

coordsToPair(pairToCoords([1, 2])) // [1, 2]

pairToCoords(coordsToPair({x: 1, y: 2})) // {x: 1, y: 2}
```

### Homomorfismo

A homomorfismo is just a structure preserving map. In fact, a functor is just a homomorfismo between categories as it preserves the original category's structure under the mapping.

```js
A.of(f).ap(A.of(x)) == A.of(f(x))

Either.of(_.toUpper).ap(Either.of("oreos")) == Either.of(_.toUpper("oreos"))
```

### Catamorfismo

A `reduceRight` function that applies a function against an accumulator and each value of the array (from right-to-left) to reduce it to a single value.

```js
const sum = xs => xs.reduceRight((acc, x) => acc + x, 0)

sum([1, 2, 3, 4, 5]) // 15
```

### Anamorfismo

An `unfold` function. An `unfold` is the opposite of `fold` (`reduce`). It generates a list from a single value.

```js
const unfold = (f, seed) => {
  function go(f, seed, acc) {
    const res = f(seed);
    return res ? go(f, res[1], acc.concat([res[0]])) : acc;
  }
  return go(f, seed, [])
}
```

```js
const countDown = n => unfold((n) => {
  return n <= 0 ? undefined : [n, n - 1]
}, n)

countDown(5) // [5, 4, 3, 2, 1]
```

### Hylomorfismo

The combination of anamorfismo and catamorfismo.

### Paramorfismo

A function just like `reduceRight`. However, there's a difference:

In paramorfismo, your reducer's arguments are the current value, the reduction of all previous values, and the list of values that formed that reduction.

```js
// Obviously not safe for lists containing `undefined`,
// but good enough to make the point.
const para = (reducer, accumulator, elements) => {
  if (elements.length === 0)
    return accumulator

  const head = elements[0]
  const tail = elements.slice(1)

  return reducer(head, tail, para(reducer, accumulator, tail))
}

const suffixes = list => para(
  (x, xs, suffxs) => [xs, ... suffxs],
  [],
  list
)

suffixes([1, 2, 3, 4, 5]) // [[2, 3, 4, 5], [3, 4, 5], [4, 5], [5], []]
```

The third parameter in the reducer (in the above example, `[x, ... xs]`) is kind of like having a history of what got you to your current acc value.

### Apomorfismo

it's the opposite of paramorfismo, just as anamorfismo is the opposite of catamorfismo. Whereas with paramorfismo, you combine with access to the accumulator and what has been accumulated, apomorfismo lets you `unfold` with the potential to return early.

## Setoid

An object that has an `equals` function which can be used to compare other objects of the same type.

Make array a setoid:

```js
Array.prototype.equals = function (arr) {
  const len = this.length
  if (len !== arr.length) {
    return false
  }
  for (let i = 0; i < len; i++) {
    if (this[i] !== arr[i]) {
      return false
    }
  }
  return true
}

;[1, 2].equals([1, 2]) // true
;[1, 2].equals([0]) // false
```

## Semigroup

An object that has a `concat` function that combines it with another object of the same type.

```js
;[1].concat([2]) // [1, 2]
```

## Foldable

An object that has a `reduce` function that applies a function against an accumulator and each element in the array (from left to right) to reduce it to a single value.

```js
const sum = (list) => list.reduce((acc, val) => acc + val, 0)
sum([1, 2, 3]) // 6
```

## Lens ##
A lens is a structure (often an object or function) that pairs a getter and a non-mutating setter for some other data
structure.

```js
// Using [Ramda's lens](http://ramdajs.com/docs/#lens)
const nameLens = R.lens(
  // getter for name property on an object
  (obj) => obj.name,
  // setter for name property
  (val, obj) => Object.assign({}, obj, {name: val})
)
```

Having the pair of get and set for a given data structure enables a few key features.

```js
const person = {name: 'Gertrude Blanch'}

// invoke the getter
R.view(nameLens, person) // 'Gertrude Blanch'

// invoke the setter
R.set(nameLens, 'Shafi Goldwasser', person) // {name: 'Shafi Goldwasser'}

// run a function on the value in the structure
R.over(nameLens, uppercase, person) // {name: 'GERTRUDE BLANCH'}
```

Lenses are also composable. This allows easy immutable updates to deeply nested data.

```js
// This lens focuses on the first item in a non-empty array
const firstLens = R.lens(
  // get first item in array
  xs => xs[0],
  // non-mutating setter for first item in array
  (val, [__, ...xs]) => [val, ...xs]
)

const people = [{name: 'Gertrude Blanch'}, {name: 'Shafi Goldwasser'}]

// Despite what you may assume, lenses compose left-to-right.
R.over(compose(firstLens, nameLens), uppercase, people) // [{'name': 'GERTRUDE BLANCH'}, {'name': 'Shafi Goldwasser'}]
```

Other implementations:
* [partial.lenses](https://github.com/calmm-js/partial.lenses) - Tasty syntax sugar and a lot of powerful features
* [nanoscope](http://www.kovach.me/nanoscope/) - Fluent-interface

## Type Signatures

Often functions in JavaScript will include comments that indicate the types of their arguments and return values.

There's quite a bit of variance across the community but they often follow the following patterns:

```js
// functionName :: firstArgType -> secondArgType -> returnType

// add :: Number -> Number -> Number
const add = (x) => (y) => x + y

// increment :: Number -> Number
const increment = (x) => x + 1
```

If a function accepts another function as an argument it is wrapped in parentheses.

```js
// call :: (a -> b) -> a -> b
const call = (f) => (x) => f(x)
```

The letters `a`, `b`, `c`, `d` are used to signify that the argument can be of any type. The following version of `map` takes a function that transforms a value of some type `a` into another type `b`, an array of values of type `a`, and returns an array of values of type `b`.

```js
// map :: (a -> b) -> [a] -> [b]
const map = (f) => (list) => list.map(f)
```

__Further reading__
* [Ramda's type signatures](https://github.com/ramda/ramda/wiki/Type-Signatures)
* [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html#whats-your-type)
* [What is Hindley-Milner?](http://stackoverflow.com/a/399392/22425) on Stack Overflow

## Algebraic data type
A composite type made from putting other types together. Two common classes of algebraic types are [sum](#sum-type) and [product](#product-type).

### Sum type
A Sum type is the combination of two types together into another one. It is called sum because the number of possible values in the result type is the sum of the input types.

JavaScript doesn't have types like this but we can use `Set`s to pretend:
```js
// imagine that rather than sets here we have types that can only have these values
const bools = new Set([true, false])
const halfTrue = new Set(['half-true'])

// The weakLogic type contains the sum of the values from bools and halfTrue
const weakLogicValues = new Set([...bools, ...halfTrue])
```

Sum types are sometimes called union types, discriminated unions, or tagged unions.

There's a [couple](https://github.com/paldepind/union-type) [libraries](https://github.com/puffnfresh/daggy) in JS which help with defining and using union types.

Flow includes [union types](https://flow.org/en/docs/types/unions/) and TypeScript has [Enums](https://www.typescriptlang.org/docs/handbook/enums.html) to serve the same role.

### Product type

A **product** type combines types together in a way you're probably more familiar with:

```js
// point :: (Number, Number) -> {x: Number, y: Number}
const point = (x, y) => ({ x, y })
```
It's called a product because the total possible values of the data structure is the product of the different values. Many languages have a tuple type which is the simplest formulation of a product type.

See also [Set theory](https://en.wikipedia.org/wiki/Set_theory).

## Option
Option is a [sum type](#sum-type) with two cases often called `Some` and `None`.

Option is useful for composing functions that might not return a value.

```js
// Naive definition

const Some = (v) => ({
  val: v,
  map (f) {
    return Some(f(this.val))
  },
  chain (f) {
    return f(this.val)
  }
})

const None = () => ({
  map (f) {
    return this
  },
  chain (f) {
    return this
  }
})

// maybeProp :: (String, {a}) -> Option a
const maybeProp = (key, obj) => typeof obj[key] === 'undefined' ? None() : Some(obj[key])
```
Use `chain` to sequence functions that return `Option`s
```js

// getItem :: Cart -> Option CartItem
const getItem = (cart) => maybeProp('item', cart)

// getPrice :: Item -> Option Number
const getPrice = (item) => maybeProp('price', item)

// getNestedPrice :: cart -> Option a
const getNestedPrice = (cart) => getItem(cart).chain(getPrice)

getNestedPrice({}) // None()
getNestedPrice({item: {foo: 1}}) // None()
getNestedPrice({item: {price: 9.99}}) // Some(9.99)
```

`Option` is also known as `Maybe`. `Some` is sometimes called `Just`. `None` is sometimes called `Nothing`.

## Function
A **function** `f :: A => B` is an expression - often called arrow or lambda expression - with **exactly one (immutable)** parameter of type `A` and **exactly one** return value of type `B`. That value depends entirely on the argument, making functions context-independant, or [referentially transparent](#referential-transparency). What is implied here is that a function must not produce any hidden [side effects](#side-effects) - a function is always [pure](#purity), by definition. These properties make functions pleasant to work with: they are entirely deterministic and therefore predictable. Functions enable working with code as data, abstracting over behaviour:

```js
// times2 :: Number -> Number
const times2 = n => n * 2

[1, 2, 3].map(times2) // [2, 4, 6]
```

## Partial function
A partial function is a [function](#function) which is not defined for all arguments - it might return an unexpected result or may never terminate. Partial functions add cognitive overhead, they are harder to reason about and can lead to runtime errors. Some examples:
```js
// example 1: sum of the list
// sum :: [Number] -> Number
const sum = arr => arr.reduce((a, b) => a + b)
sum([1, 2, 3]) // 6
sum([]) // TypeError: Reduce of empty array with no initial value

// example 2: get the first item in list
// first :: [A] -> A
const first = a => a[0]
first([42]) // 42
first([]) // undefined
//or even worse:
first([[42]])[0] // 42
first([])[0] // Uncaught TypeError: Cannot read property '0' of undefined

// example 3: repeat function N times
// times :: Number -> (Number -> Number) -> Number
const times = n => fn => n && (fn(n), times(n - 1)(fn))
times(3)(console.log)
// 3
// 2
// 1
times(-1)(console.log)
// RangeError: Maximum call stack size exceeded
```

### Dealing with partial functions
Partial functions are dangerous as they need to be treated with great caution. You might get an unexpected (wrong) result or run into runtime errors. Sometimes a partial function might not return at all. Being aware of and treating all these edge cases accordingly can become very tedious.
Fortunately a partial function can be converted to a regular (or total) one. We can provide default values or use guards to deal with inputs for which the (previously) partial function is undefined. Utilizing the [`Option`](#Option) type, we can yield either `Some(value)` or `None` where we would otherwise have behaved unexpectedly:
```js
// example 1: sum of the list
// we can provide default value so it will always return result
// sum :: [Number] -> Number
const sum = arr => arr.reduce((a, b) => a + b, 0)
sum([1, 2, 3]) // 6
sum([]) // 0

// example 2: get the first item in list
// change result to Option
// first :: [A] -> Option A
const first = a => a.length ? Some(a[0]) : None()
first([42]).map(a => console.log(a)) // 42
first([]).map(a => console.log(a)) // console.log won't execute at all
//our previous worst case
first([[42]]).map(a => console.log(a[0])) // 42
first([]).map(a => console.log(a[0])) // won't execte, so we won't have error here
// more of that, you will know by function return type (Option)
// that you should use `.map` method to access the data and you will never forget
// to check your input because such check become built-in into the function

// example 3: repeat function N times
// we should make function always terminate by changing conditions:
// times :: Number -> (Number -> Number) -> Number
const times = n => fn => n > 0 && (fn(n), times(n - 1)(fn))
times(3)(console.log)
// 3
// 2
// 1
times(-1)(console.log)
// won't execute anything
```
Making your partial functions total ones, these kinds of runtime errors can be prevented. Always returning a value will also make for code that is both easier to maintain as well as to reason about.

## Functional Programming Libraries in JavaScript

* [mori](https://github.com/swannodette/mori)
* [Immutable](https://github.com/facebook/immutable-js/)
* [Immer](https://github.com/mweststrate/immer)
* [Ramda](https://github.com/ramda/ramda)
* [ramda-adjunct](https://github.com/char0n/ramda-adjunct)
* [Folktale](http://folktale.origamitower.com/)
* [monet.js](https://cwmyers.github.io/monet.js/)
* [lodash](https://github.com/lodash/lodash)
* [Underscore.js](https://github.com/jashkenas/underscore)
* [Lazy.js](https://github.com/dtao/lazy.js)
* [maryamyriameliamurphies.js](https://github.com/sjsyrek/maryamyriameliamurphies.js)
* [Haskell in ES6](https://github.com/casualjavascript/haskell-in-es6)
* [Sanctuary](https://github.com/sanctuary-js/sanctuary)
* [Crocks](https://github.com/evilsoft/crocks)
* [Fluture](https://github.com/fluture-js/Fluture)
* [fp-ts](https://github.com/gcanti/fp-ts)

---