
```bash
  pix-integration/
  ├── cmd/
  │   ├── main.go
  ├── internal/
  │   ├── api/
  │   │   ├── handler.go
  │   ├── service/
  │   │   ├── pix.go
  │   ├── utils/
  │   │   ├── transaction.go
  ├── scripts/
  │   ├── setup.sh
  │   ├── run.sh
  ├── go.mod
  ├── go.sum
``

# Estrutura do Projeto

O projeto está dividido nos seguintes diretórios:

- `cmd/`: Contém o ponto de entrada da aplicação (`main.go`).
- `internal/api/`: Define os handlers das requisições HTTP.
- `internal/service/`: Implementa a lógica de negócio das transações PIX.
- `internal/utils/`: Contém funções auxiliares como geração de IDs de transação.
- `scripts/`: Contém scripts úteis para configurar e executar o projeto.

## Como Executar

### 1. Criar a Estrutura de Pastas
Execute o seguinte script para criar a estrutura do projeto na sua máquina:

```bash
#!/bin/bash
mkdir -p pix-integration/cmd
mkdir -p pix-integration/internal/api
mkdir -p pix-integration/internal/service
mkdir -p pix-integration/internal/utils
mkdir -p pix-integration/scripts

touch pix-integration/cmd/main.go

touch pix-integration/internal/api/handler.go

touch pix-integration/internal/service/pix.go

touch pix-integration/internal/utils/transaction.go

touch pix-integration/scripts/setup.sh

touch pix-integration/scripts/run.sh

echo "Estrutura do projeto criada com sucesso."
```

Salve esse script como `setup.sh` e execute:

```bash
chmod +x setup.sh
./setup.sh
```

### 2. Rodar o Serviço
Use o seguinte script para rodar o serviço:

```bash
#!/bin/bash
cd pix-integration/cmd

go run main.go
```

Execute com:

```bash
chmod +x scripts/run.sh
./scripts/run.sh
```

---

## Implementação dos Arquivos

### `cmd/main.go`

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"pix-integration/internal/api"
)

func main() {
	http.HandleFunc("/pix", api.ProcessPix)
	fmt.Println("PIX Integration Service running on port 8080...")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### `internal/api/handler.go`

```go
package api

import (
	"encoding/json"
	"net/http"
	"pix-integration/internal/service"
)

type PixRequest struct {
	Sender    string  `json:"sender"`
	Receiver  string  `json:"receiver"`
	Amount    float64 `json:"amount"`
}

type PixResponse struct {
	TransactionID string `json:"transaction_id"`
	Status        string `json:"status"`
	Message       string `json:"message"`
}

func ProcessPix(w http.ResponseWriter, r *http.Request) {
	var request PixRequest
	decoder := json.NewDecoder(r.Body)
	if err := decoder.Decode(&request); err != nil {
		http.Error(w, "Invalid request", http.StatusBadRequest)
		return
	}

	response := service.HandlePixTransaction(request.Sender, request.Receiver, request.Amount)

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

### `internal/service/pix.go`

```go
package service

import (
	"pix-integration/internal/utils"
)

type PixResponse struct {
	TransactionID string `json:"transaction_id"`
	Status        string `json:"status"`
	Message       string `json:"message"`
}

func HandlePixTransaction(sender string, receiver string, amount float64) PixResponse {
	return PixResponse{
		TransactionID: utils.GenerateTransactionID(),
		Status:        "SUCCESS",
		Message:       "Transaction processed successfully.",
	}
}
```

### `internal/utils/transaction.go`

```go
package utils

import (
	"fmt"
	"math/rand"
)

func GenerateTransactionID() string {
	return fmt.Sprintf("TX-%d", rand.Intn(1000000))
}
```
