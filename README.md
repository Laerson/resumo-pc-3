# Resumo prova 3

## goroutines

Para invocar uma thread, basta usar a palavra reservada `go` antes da chamada da função.

exemplo com função anônima:

```go
go func() {
    fmt.Println("Hello from another goroutine")
}()
```

exemplo com função nomeada:

```go
func foo() {
    // Do Something
}

func main() {
    go foo()
}
```

## Canais

Forma de comunicação entre goroutines.

Para criar um canal:

```go
ch := make(chan <type>, size)
```

onde `<type>` é o tipo de dado que será trafegado pelo canal e `size` é o tamanho do buffer.

Se omitir o tamanho do buffer, o canal será criado sem buffer (canal só aceita uma mensagem por vez).

ex:

```go
ch := make(chan int)
```

Para enviar mensagens para um canal usamos o operador `<-`:

```go
ch <- 10
```

Para consumir a mensagem também usamos o operador `<-` (mas do outro lado):

```go
x := <- ch
```

Podemos consumir sem atribuir a uma variável:

```go
<- ch
```

Canal bloqueia chamadas de envio e recebimento até que seja possivel ler/escrever no canal.

É possível definir um canal somente para envio ou somente para recebimento [Link](<https://gobyexample.com/channel-directions>) (não necessário para a prova).

## Select

Select funciona como um switch para canais. Ao inveś de executar o primeiro case verdadeiro, ele executa o primeiro canal desbloqueado (ou aleatorio se tive mais de um canal desbloqueado).

Podemos também usar o `default` para executar um case caso nenhum canal esteja desbloqueado.

Importante: `select` é bloqueante sem um `default`. Se nenhum canal estiver desbloqueado, a execução do programa será bloqueada até que um canal seja desbloqueado.

ex:

```go
select {
    case x := <- ch1:
        fmt.Println(x)
    case ch2 <- y:
        fmt.Println("enviou")
    default:
        fmt.Println("nenhum canal desbloqueado")
}
```

## Close

Podemos fechar um canal usando a função `close`:

```go
close(ch)
```

Quando uma mensagem é consumida de um canal, a operação retorna dois valores: A mensagem, e um booleano que indica se o canal está aberto ou fechado E **Vazio**.

Mesmo apos usar close() se o canal não estiver vazio, ainda é possivel consumir mensagens dele.

ex:

```go
package main

import "fmt"

func main() {
    jobs := make(chan int, 5)
    done := make(chan bool)

    go func() {
        for {
            j, more := <-jobs
            if more {
                fmt.Println("received job", j)
            } else {
                fmt.Println("received all jobs")
                done <- true
                return
            }
        }
    }()

    for j := 1; j <= 3; j++ {
        jobs <- j
        fmt.Println("sent job", j)
    }
    close(jobs)
    fmt.Println("sent all jobs")

    <-done
}
```

## Range

Podemos usar o `range` para consumir mensagens de um canal. O range só termina quando o canal é fechado.

ex:

```go
package main

import "fmt"

func main() {

    queue := make(chan string, 2)
    queue <- "one"
    queue <- "two"
    close(queue)

    for elem := range queue {
        fmt.Println(elem)
    }
}
```

## WaitGroup

WaitGroup é uma estrutura que permite que uma goroutine espere o termino de outras goroutines.

necessário importar o pacote `sync`.

### Criação

```go
var wg sync.WaitGroup
```

### Adicionar goroutines

```go
wg.Add(1)
```

wg.Add(n) adiciona um valor N ao contador de goroutines.

### Esperar goroutines

```go
wg.Wait()
```

wg.Wait() bloqueia a execução onde foi chamado até que o contador de goroutines chegue a zero.

### Finalizar goroutines

```go
wg.Done()
```

wg.Done() decrementa o contador de goroutines em 1.

## Defer

Defer é uma palavra reservada que permite que garante que uma função seja executada antes do retorno de uma função.

Os valores são calculados no momento da chamada da função, mas a execução é adiada até o retorno da função.

O defer é executado na ordem inversa que foi declarado, se houver mais de um defer na função (LIFO).

A razão de existir do defer é para garantir que uma rotina seja executada antes do retorno de uma função, mesmo que ocorra um erro.

ex:

```go
func main() {
    defer fmt.Println("world")

    fmt.Println("hello")
}
```

## Extra stuff

### Ler argumentos passados para o programa

`os.Args` 

### Converter string para inteiro

`strconv.Atoi(string)`

### Gerar numeros aleatorios

`random, err := rand.Int(rand.Read, big.NewInt(max))`

max é o valor maximo que o numero aleatorio pode ter(não incluso).

precisa importar o pacote `crypto/rand` e `math/big`.

random tem o tipo `*big.Int`, para converter para `int` basta usar `random.Int64()`.

### Fazer um sleep

`time.Sleep(time.Duration(N) * time.Second)`

Faz a thread esperar N segundos

Fazer a thread esperar um numero aleatorio de segundos (entre 0 e max  `[0,max)``):

```go
random, err := rand.Int(rand.Read, big.NewInt(max))
time.Sleep(time.Duration(random.Int64()) * time.Second)
```
