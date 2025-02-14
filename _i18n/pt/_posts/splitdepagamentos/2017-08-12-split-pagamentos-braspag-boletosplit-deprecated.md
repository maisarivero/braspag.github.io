---
layout: manual
title: Manual Boleto Braspag
description: Manual de Integração do Boleto Braspag
search: true
toc_footers: false
categories: manual
sort_order: 7
hub_visible: false
tags:
---

# Boleto Braspag

## Boleto Registrado

Com o objetivo de promover maior controle e segurança ao transacional de boletos no e-commerce e garantir mais confiabilidade e comodidade aos usuários, a Febraban em conjunto com os Bancos lançou a Nova Plataforma de Cobrança.

A partir de 21 de julho de 2018 todos os boletos emitidos no e-commerce, obrigatoriamente, terão de ser registrados. [Clique aqui](https://portal.febraban.org.br/pagina/3150/1094/pt-br/servicos-novo-plataforma-boletos) para acessar o comunicado completo.

## Criando uma transação de Boleto

Para gerar um boleto em sandbox, é necessário fornecer dados do comprador como CPF e endereço. Abaixo temos um exemplo de como criar um pedido com o meio de pagamento boleto.

### Criando uma transação utilizando a integração via Pagador

**Request**

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/v2/sales/</span></aside>

```json
--header "Content-Type: application/json"
--header "MerchantId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
--header "MerchantKey: 0123456789012345678901234567890123456789"
{  
    "MerchantOrderId":"2017091101",
    "Customer":
    {  
        "Name":"Nome do Comprador",
        "Identity":"12345678909",
        "IdentityType":"CPF",
        "Address":{  
            "Street":"Alameda Xingu",
            "Number":"512",
            "Complement":"27 andar",
            "ZipCode":"12345987",
            "City":"Sao Paulo",
            "State":"SP",
            "Country":"BRA",
            "District":"Alphaville"
        }
    },
    "Payment":
    {  
     "Provider":"Braspag",
     "Bank": "BancoDoBrasil",
     "Type":"Boleto",
     "Amount":10000,
     "BoletoNumber":"2017091101",
     "Assignor": "Empresa Teste",
     "Demonstrative": "Desmonstrative Teste",
     "ExpirationDate": "2017-12-31",
     "Identification": "12346578909",
     "Instructions": "Aceitar somente até a data de vencimento.",
     "DaysToFine": 1,
     "FineRate": 10.00000,
     "FineAmount": 1000,
     "DaysToInterest": 1,
     "InterestRate": 5.00000,
     "InterestAmount": 500
    }
}
```

|Propriedade|Tipo|Tamanho|Obrigatório|Descrição|
|-----------|----|-------|-----------|---------|
|`MerchantId`|Guid|36|Sim|Identificador da loja na Braspag|
|`MerchantKey`|Texto|40|Sim|Chave Publica para Autenticação Dupla na Braspag|
|`MerchantOrderId`|Texto|vide tabela abaixo|Sim|Numero de identificação do Pedido. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Name`|Texto|vide tabela abaixo|Sim|Nome do comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Identity`|Texto |14 |Sim|Número do RG, CPF ou CNPJ do Cliente| 
|`Customer.IdentityType`|Texto|255|Sim|Tipo de documento de identificação do comprador (CPF ou CNPJ)|
|`Customer.Address.Street`|Texto|vide tabela abaixo|Sim|Endereço de contato do comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Address.Number`|Texto|vide tabela abaixo|Sim|Número endereço de contato do comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Address.Complement`|Texto|vide tabela abaixo|Não|Complemento do endereço de contato do Comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Address.ZipCode`|Texto|8|Sim|CEP do endereço de contato do comprador|
|`Customer.Address.District`|Texto|vide tabela abaixo|Sim|Bairro do endereço de contato do comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Address.City`|Texto|vide tabela abaixo|Sim|Cidade do endereço de contato do comprador. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Customer.Address.State`|Texto|2|Sim|Estado do endereço de contato do comprador|
|`Customer.Address.Country`|Texto|35|Sim|Pais do endereço de contato do comprador|
|`Payment.Provider`|Texto|15|Sim|Nome da provedora de Meio de Pagamento de Boleto (Braspag)|
|`Payment.Bank`|Texto|15|Sim|Nome do Banco que o boleto será emitido|
|`Payment.Type`|Texto|100|Sim|Tipo do Meio de Pagamento. No caso "Boleto"|
|`Payment.Amount`|Número|15|Sim|Valor do Pedido (deve ser enviado em centavos)|
|`Payment.BoletoNumber`|Texto |vide tabela abaixo|Não|Número do Boleto ("Nosso Número"). Caso preenchido, sobrepõe o valor configurado no meio de pagamento. A regra varia de acordo com o Provider utilizado (vide tabela abaixo|
|`Payment.Assignor`|Texto |200|Não|Nome do Cedente. Caso preenchido, sobrepõe o valor configurado no meio de pagamento|
|`Payment.Demonstrative`|Texto |vide tabela abaixo|Não|Texto de Demonstrativo. Caso preenchido, sobrepõe o valor configurado no meio de pagamento. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Payment.ExpirationDate`|Date |AAAA-MM-DD|Não|Dias para vencer o boleto. Caso não esteja previamente cadastrado no meio de pagamento, o envio deste campo é obrigatório. Se enviado na requisição, sobrepõe o valor configurado no meio de pagamento.|
|`Payment.Identification`|Texto |14 |Não|CNPJ do Cedente. Caso preenchido, sobrepõe o valor configurado no meio de pagamento|
|`Payment.Instructions`|Texto |vide tabela abaixo|Não|Instruções do Boleto. Caso preenchido, sobrepõe o valor configurado no meio de pagamento. A regra varia de acordo com o Provider utilizado (vide tabela abaixo)|
|`Payment.NullifyDays`|Número |2 |Não|Prazo para baixa automática do boleto. O cancelamento automático do boleto acontecerá após o número de dias estabelecido neste campo contado a partir da data do vencimento. Ex.: um boleto com vencimento para 15/12 que tenha em seu registro o prazo para baixa de 5 dias, poderá ser pago até 20/12, após esta data o título é cancelado. *Recurso válido somente para boletos registrados do Banco Santander.|
|`Payment.DaysToFine`|Número |15 |Não|Opcional e somente para provider Bradesco2. Quantidade de dias após o vencimento para cobrar o valor da multa, em número inteiro. Ex: 3|
|`Payment.FineRate`|Número |15 |Não|Opcional e somente para provider Bradesco2. Valor da multa após o vencimento em percentual, com base no valor do boleto (%). Permitido decimal com até 5 casas decimais. Não enviar se utilizar FineAmount. Ex: 10.12345 = 10.12345%|
|`Payment.FineAmount`|Número |15 |Não|Opcional e somente para provider Bradesco2. Valor da multa após o vencimento em valor absoluto em centavos. Não enviar se utilizar FineRate.  Ex: 1000 = R$ 10,00|
|`Payment.DaysToInterest`|Número |15 |Não|Opcional e somente para provider Bradesco2.Quantidade de dias após o vencimento para iniciar a cobrança de juros por dia sobre o valor do boleto, em número inteiro. Ex: 3|
|`Payment.InterestRate`|Número |15 |Não|Opcional e somente para provider Bradesco2. Valor de juros mensal após o vencimento em percentual, com base no valor do boleto (%). O valor de juros é cobrado proporcionalmente por dia (Mensal dividido por 30). Permitido decimal com até 5 casas decimais. Não enviar se utilizar InterestAmount. Ex: 10.12345|
|`Payment.InterestAmount`|Número |15 |Não|Opcional e somente para provider Bradesco2. Valor absoluto de juros diário após o vencimento em centavos. Não enviar se utilizar InterestRate. Ex: 1000 = R$ 10,00|

**Response**

```json
{
    "MerchantOrderId": "2017091101",
    "Customer": {
        "Name": "Nome do Comprador",
        "Identity": "12345678909",
        "IdentityType": "CPF",
        "Address": {
            "Street": "Alameda Xingu",
            "Number": "512",
            "Complement": "27 andar",
            "ZipCode": "12345987",
            "City": "Sao Paulo",
            "State": "SP",
            "Country": "BRA",
            "District": "Alphaville"
        }
    },
    "Payment": {
        "Instructions": "Aceitar somente até a data de vencimento.",
        "ExpirationDate": "2017-12-31",
        "Demonstrative": "Desmonstrative Teste",
        "Url": "https://transactionsandbox.pagador.com.br/post/pagador/reenvia.asp/d605c399-96b2-4bb9-ae75-33824ec01be9",
        "BoletoNumber": "0000000155",
        "BarCodeNumber": "",
        "DigitableLine": "",
        "Assignor": "Empresa Teste",
        "Address": "ESTRADA TENENTE MARQUES, 1818, SALA 6 B",
        "Identification": "12346578909",
        "IsRecurring": false,
        "InterestAmount": 500,
        "InterestRate": 5.0,
        "FineRate": 10.0,
        "FineAmount": 1000,
        "DaysToFine": 1,
        "DaysToInterest": 1,
        "Bank": "BancoDoBrasil",
        "PaymentId": "d605c399-96b2-4bb9-ae75-33824ec01be9",
        "Type": "Boleto",
        "Amount": 10000,
        "ReceivedDate": "2019-12-03 12:05:37",
        "Currency": "BRL",
        "Country": "BRA",
        "Provider": "Braspag",
        "ReasonCode": 0,
        "ReasonMessage": "Successful",
        "Status": 1,
        "ProviderReturnCode": "0",
        "ProviderReturnMessage": "Transação criada com sucesso",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.braspag.com.br/v2/sales/d605c399-96b2-4bb9-ae75-33824ec01be9"
            }
        ]
    }
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`PaymentId`|Campo Identificador do Pedido. |Guid |36 |xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx |
|`ExpirationDate`|Data de expiração. |Texto |10 |2014-12-25 |
|`Url`|URL do Boleto gerado |string |256 |https://.../pagador/reenvia.asp/8464a692-b4bd-41e7-8003-1611a2b8ef2d |
|`BoletoNumber`|"NossoNumero" gerado. |Texto|50 |2017091101 |
|`BarCodeNumber`|Representação numérica do código de barras. |Texto |44 |00091628800000157000494250100000001200656560 |
|`DigitableLine`|Linha digitável. |Texto |256 |00090.49420 50100.000004 12006.565605 1 62880000015700 |
|`Address`|Endereço do Loja cadastrada no banco |Texto |256 |Av. Teste, 160 |
|`Status`|Status da Transação. |Byte | 2 | Ex. 1 |

### Criando uma transação utilizando a integração via Cielo 3.0

**Request**

<aside class="request"><span class="method post">POST</span> <span class="endpoint">/1/sales/</span></aside>

```json
--header "Content-Type: application/json"
--header "Authorization: Bearer {{access_token}}"

{
    "MerchantOrderId":  "31029785000159",
    "Customer":  {
        "Name":"Gabriela Isis Malu Aparício",
        "Identity":  "60191661040",
        "IdentityType":  "CPF",
        "Address":  {
            "Street":  "Rua Brasil",
            "Number":  "123",
            "Complement":  "AP 123",
            "ZipCode":  "12345987",
            "City":  "Rio de Janeiro",
            "State":  "RJ",
            "Country":  "BRA",
            "District":  "Centro"
        }
    },
    "Payment":  {
        "Type":  "Boleto",
        "Provider":  "Braspag",
        "Bank":  "BancoDoBrasil",
        "Amount":  10000,
        "ExpirationDate":  "2021-02-15",
        "Identification":"60191661040",
        "Instructions":  "Intruções para o cliente final.",
        "SplitPayments":  [
            {
                "SubordinateMerchantId":  "768d0acf-9502-4411-9ec0-c5413c671771",
                "Amount":  1000,
                "fares":{
                    "mdr":  5.0,
                    "fee":  100
                }
            }
        ],
        "SplitTransaction":  {
            "MasterRateDiscountType":  "Commission"
        }
    }
}
```

|Propriedade|Descrição|Tipo|Tamanho|Obrigatório|
|-----------|----|-------|-----------|---------|
|Payment.Assignor|Nome do Cedente.|Texto|200|Não|
|Payment.BoletoNumber|Número do Boleto enviado pelo lojista. Usado para contar boletos emitidos (“NossoNumero”).|Texto|Banco do Brasil: 9|Não|
|Payment.Demonstrative|Texto de Demonstrativo.|Texto|255|Não|
|Payment.ExpirationDate|Data de expiração do Boleto. Ex. 2020-12-31|Date|10|Não|
|Payment.Identification|Documento de identificação do Cedente.|Texto|14|Não|
|Payment.Instructions|Instruções do Boleto.|Texto|255|Não|
|Customer.Address.City|Cidade do endereço do Comprador.|Texto|Banco do Brasil: 18|Sim|
|Customer.Address.Country|Pais do endereço do Comprador.|Texto|35|Sim|
|Customer.Address.District|Bairro do Comprador.|Texto|50|Sim|
|Customer.Address.Number|Número do endereço do Comprador.|Texto|15|Sim|
|Customer.Address.State|Estado do endereço do Comprador.|Texto|2|Sim|
|Customer.Address.Street|Endereço do Comprador.|Texto|255|Sim|
|Customer.Address.ZipCode|CEP do endereço do Comprador.|Texto|9|Sim|
|Customer.Name|Nome do Comprador.|Texto|Banco do Brasil: 60|Sim|
|MerchantOrderId|Numero de identificação do Pedido.|Texto|Banco do Brasil: 50|Sim|
|Payment.Amount|Valor do Pedido (deve ser enviado em centavos)|Número|15|Sim|
|Payment.Provider|Define comportamento do meio de pagamento. (Valor: Braspag)|Texto|15|Sim|
|Payment.Type|Tipo do Meio de Pagamento. (Valor: Boleto)|Texto|100|Sim|
|Customer.Identity|Número do RG, CPF ou CNPJ do cliente.|Texto|14|Sim|
|Customer.IdentityType|Tipo de documento de identificação do comprador (CPF ou CNPJ).|Texto|255|Sim|
|SplitPayments.[].SubordinateMerchantId|MerchantId do subordinado/Master participante da venda|GUID|36|Não|
|SplitPayments.[].Amount|Valor referente a venda do partcipante|Número||Não|
|SplitPayments.[].Fares.Mdr|Porcentagem cobrado pelo Master sobre a venda do participante|Decimal||Não|
|SplitPayments.[].Fares.Fares|Valor fixo em centavos cobrado pelo Master sobre a venda do participante|Número||Não|
|SplitTransaction.MasterRateDiscountType|Tipo de desconto da taxa Braspag. Valores disponíveis: Commision (Será descontado da comissão recebida pelo Master), Sale (Será descontado somente do valor da venda do Master)|Texto||Não |

**Response**

```json
{
    "MerchantOrderId": "31029785000159",
    "Customer": {
        "Name": "Gabriela Aparicio",
        "Identity": "60191661040",
        "IdentityType": "CPF",
        "Address": {
            "Street": "Rua Brasil",
            "Number": "123",
            "Complement": "AP 123",
            "ZipCode": "12345987",
            "City": "Rio de Janeiro",
            "State": "RJ",
            "Country": "BRA",
            "District": "Centro"
        }
    },
    "Payment": {
        "Instructions": "Intruções para o cliente final.",
        "ExpirationDate": "2021-02-15",
        "Url": "https://transactionsandbox.pagador.com.br/post/pagador/reenvia.asp/bc99c4c5-010a-48be-a0e4-b2143c764a52",
        "BoletoNumber": "0000002172",
        "BarCodeNumber": "",
        "DigitableLine": "",
        "Address": "N/A, 1",
        "Identification": "60191661040",
        "ProviderReturnCode": "0",
        "ProviderReturnMessage": "Transação criada com sucesso",
        "Bank": 4,
        "Amount": 10000,
        "ReceivedDate": "2021-01-20 15:15:33",
        "Provider": "Braspag",
        "Status": 1,
        "IsSplitted": false,
        "ReturnMessage": "Transação criada com sucesso",
        "ReturnCode": "0",
        "PaymentId": "bc99c4c5-010a-48be-a0e4-b2143c764a52",
        "Type": "Boleto",
        "Currency": "BRL",
        "Country": "BRA",
        "Links": [
            {
                "Method": "GET",
                "Rel": "self",
                "Href": "https://apiquerysandbox.cieloecommerce.cielo.com.br/1/sales/bc99c4c5-010a-48be-a0e4-b2143c764a52"
            }
        ],
        "SplitPayments": [
            {
                "SubordinateMerchantId": "768d0acf-9502-4411-9ec0-c5413c671771",
                "Amount": 10000,
                "Fares": {
                    "Mdr": 5.0,
                    "Fee": 100
                }
            }
        ],
        "SplitTransaction": {
            "MasterRateDiscountType": "Commission"
        }
    }
}
```

|Propriedade|Descrição|Tipo|Tamanho|Formato|
|-----------|---------|----|-------|-------|
|`PaymentId`|Campo Identificador do Pedido. |Guid |36 |xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx |
|`ExpirationDate`|Data de expiração. |Texto |10 |2014-12-25 |
|`Url`|URL do Boleto gerado |string |256 |https://.../pagador/reenvia.asp/8464a692-b4bd-41e7-8003-1611a2b8ef2d |
|`BoletoNumber`|"NossoNumero" gerado. |Texto|50 |2017091101 |
|`BarCodeNumber`|Representação numérica do código de barras. |Texto |44 |00091628800000157000494250100000001200656560 |
|`DigitableLine`|Linha digitável. |Texto |256 |00090.49420 50100.000004 12006.565605 1 62880000015700 |
|`Address`|Endereço do Loja cadastrada no banco |Texto |256 |Av. Teste, 160 |
|`Status`|Status da Transação. |Byte | 2 | Ex. 1 |
