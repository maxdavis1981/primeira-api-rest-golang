# Desafio Dio - Criando a sua Primeira API Rest com Go



### **projeto completo com bastante código em Go para criar sua primeira API REST:**

go

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
)

type Pessoa struct {
    ID        int    `json:"id"`
    Nome      string `json:"nome"`
    Sobrenome string `json:"sobrenome"`
    Idade     int    `json:"idade"`
}

var pessoas []Pessoa

func main() {
    http.HandleFunc("/pessoas", pessoasHandler)
    http.HandleFunc("/pessoas/", pessoaHandler)

    log.Fatal(http.ListenAndServe(":8080", nil))
}

func pessoasHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        json.NewEncoder(w).Encode(pessoas)
    case http.MethodPost:
        var pessoa Pessoa
        if err := json.NewDecoder(r.Body).Decode(&pessoa); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }
        pessoa.ID = len(pessoas) + 1
        pessoas = append(pessoas, pessoa)
        json.NewEncoder(w).Encode(pessoa)
    default:
        http.Error(w, "Método não permitido", http.StatusMethodNotAllowed)
    }
}

func pessoaHandler(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(strings.TrimPrefix(r.URL.Path, "/pessoas/"))
    if err != nil {
        http.Error(w, "ID inválido", http.StatusBadRequest)
        return
    }

    switch r.Method {
    case http.MethodGet:
        for _, pessoa := range pessoas {
            if pessoa.ID == id {
                json.NewEncoder(w).Encode(pessoa)
                return
            }
        }
        http.Error(w, "Pessoa não encontrada", http.StatusNotFound)
    case http.MethodPut:
        var pessoa Pessoa
        if err := json.NewDecoder(r.Body).Decode(&pessoa); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }
        for i, p := range pessoas {
            if p.ID == id {
                pessoas[i] = pessoa
                json.NewEncoder(w).Encode(pessoa)
                return
            }
        }
        http.Error(w, "Pessoa não encontrada", http.StatusNotFound)
    case http.MethodDelete:
        for i, p := range pessoas {
            if p.ID == id {
                pessoas = append(pessoas[:i], pessoas[i+1:]...)
                w.WriteHeader(http.StatusNoContent)
                return
            }
        }
        http.Error(w, "Pessoa não encontrada", http.StatusNotFound)
    default:
        http.Error(w, "Método não permitido", http.StatusMethodNotAllowed)
    }
}
```



**Este projeto cria uma API REST simples que permite gerenciar uma lista de pessoas.**



- **`/pessoas`**: Retorna uma lista de todas as pessoas.
- **`/pessoas/{id}`**: Retorna uma única pessoa pelo seu ID.
- **`/pessoas`**: Cria uma nova pessoa.
- **`/pessoas/{id}`**: Atualiza uma pessoa pelo seu ID.
- **`/pessoas/{id}`**: Exclui uma pessoa pelo seu ID.



**Para executar o projeto, execute o seguinte comando:**

```plaintext
go run main.go
```



**Em seguida, você pode acessar a API usando um cliente HTTP como o cURL:**

```plaintext
curl http://localhost:8080/pessoas
```



**Isso retornará uma lista de todas as pessoas.**



**Você também pode criar uma nova pessoa usando o seguinte comando:**

```plaintext
curl -X POST -H "Content-Type: application/json" -d '{"nome": "João", "sobrenome": "Silva", "idade": 30}' http://localhost:8080/pessoas
```



**Isso criará uma nova pessoa com o nome "João Silva" e idade 30.**



**Você pode obter mais informações sobre como** 



#### **Middleware para registro de requisições:**

go

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s %s", r.Method, r.URL.Path, r.RemoteAddr)
        next.ServeHTTP(w, r)
    })
}
```



#### **Exemplo de uso do middleware:**

go

```go
func main() {
    http.Handle("/pessoas", loggingMiddleware(pessoasHandler))
    http.Handle("/pessoas/", loggingMiddleware(pessoaHandler))

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```



#### **Manipulação de erros personalizados:**

go

```go
type AppError struct {
    Code    int
    Message string
}

func (e AppError) Error() string {
    return fmt.Sprintf("%d: %s", e.Code, e.Message)
}
```



#### **Exemplo de uso da manipulação de erros personalizados:**

go

```go
func pessoaHandler(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(strings.TrimPrefix(r.URL.Path, "/pessoas/"))
    if err != nil {
        http.Error(w, AppError{Code: http.StatusBadRequest, Message: "ID inválido"}, http.StatusBadRequest)
        return
    }

    // ...
}
```



#### **Validação de entrada:**

go

```go
func validatePessoa(p Pessoa) error {
    if p.Nome == "" {
        return AppError{Code: http.StatusBadRequest, Message: "Nome é obrigatório"}
    }
    if p.Sobrenome == "" {
        return AppError{Code: http.StatusBadRequest, Message: "Sobrenome é obrigatório"}
    }
    if p.Idade <= 0 {
        return AppError{Code: http.StatusBadRequest, Message: "Idade deve ser maior que zero"}
    }

    return nil
}
```



#### **Exemplo de uso da validação de entrada:**

go

```go
func pessoaHandler(w http.ResponseWriter, r *http.Request) {
    var pessoa Pessoa
    if err := json.NewDecoder(r.Body).Decode(&pessoa); err != nil {
        http.Error(w, AppError{Code: http.StatusBadRequest, Message: "Entrada inválida"}, http.StatusBadRequest)
        return
    }

    if err := validatePessoa(pessoa); err != nil {
        http.Error(w, err, http.StatusBadRequest)
        return
    }

    // ...
}
```



#### **Documentação da API:**

go

```go
// @title       API de Pessoas
// @description Gerencie uma lista de pessoas.
// @version     1.0
// @host        localhost:8080
// @BasePath    /
```



#### **Exemplo de uso da documentação da API:**



```plaintext
swagger: "2.0"
info:
  title: "API de Pessoas"
  description: "Gerencie uma lista de pessoas."
  version: "1.0"
host: "localhost:8080"
basePath: "/"
```
