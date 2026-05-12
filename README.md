Segue o README com todos os dados sensíveis substituídos por placeholders padronizados:

---

# Automação RPA — COMEX (Conferência de Documentos de Embarque DHL)

## 📌 Descrição

Este conjunto de automações (Power Automate / Azure Logic Apps + UiFlow) gerencia o recebimento, organização e conferência de documentos de embarque da DHL (Aéreo). O processo cobre desde o alerta formal da DHL até a inserção dos dados validados no FollowNet.

O fluxo é dividido em dois principais gatilhos de e-mail:

1. **Alerta DHL** – recebe e-mails com assunto `DHL Express – Alerta Formal`, extrai o número do **HAWB** (House Air Waybill) do assunto e salva todos os anexos em uma pasta no SharePoint.
2. **PARA LANÇAMENTO** – recebe e-mails com assunto `PARA LANÇAMENTO - Processo:`, extrai o **IMP** e o **HAWB** do corpo/assunto, cria a mesma estrutura de pastas, processa invoices e packing lists, aguarda a chegada do HAWB em PDF e então executa um robô desktop que realiza a **conferência completa dos documentos**, insere os dados no FollowNet e registra ocorrências com todas as validações.

---

## 🎯 Objetivo

* Automatizar o recebimento e a organização dos documentos de embarque DHL.
* Eliminar a necessidade de criação manual de pastas e movimentação de anexos.
* Realizar a conferência automática de **HAWB, Invoice e Packing List**, comparando dados como:

  * Endereços de entrega e invoice (compatibilidade)
  * Pesos bruto e cobrável (com tolerância de 10%)
  * Quantidade de volumes
  * Itens por SKU (identificando divergências)
  * Taxa de combustível (obtida do site da DHL para o período do HAWB)
* Calcular o valor total do frete com base nos dados extraídos e validar contra o valor declarado no HAWB.
* Inserir automaticamente os dados da conferência no **FollowNet**, abrindo uma ocorrência com todas as validações documentadas no campo "Observações".
* Notificar a equipe quando um documento essencial (Waybill) não for encontrado.

---

## 🛠 Tecnologias Utilizadas

* Power Automate / Azure Logic Apps (fluxos de nuvem)
* Office 365 Outlook (conector de e-mail)
* SharePoint Online (armazenamento de documentos)
* Conversions Service (HTML → texto para extração do HAWB)
* UiFlow (Power Automate Desktop)

  * `COMEX-ConferenciaDeDocumentosDeEmbarqueDHLV6`
  * `COMEX-LeitorInvoiceePackingList`
  * `COMEX-LoginFollowNet`

---

## 🔄 Fluxo do Processo

### Fluxo de nuvem (recebimento e organização)

```text
[E-mail: DHL Express – Alerta Formal]
    ↓
Extrair HAWB do assunto
    ↓
Criar pasta no SharePoint: /COMEX - Conferência Pré-Alerta DHL/HAWB <número>
    ↓
Salvar anexos na pasta
```

```text
[E-mail: PARA LANÇAMENTO - Processo: ...]
    ↓
Extrair IMP e HAWB
    ↓
Criar ou reutilizar pasta do processo
    ↓
Filtrar anexos (invoice / packing list)
    ↓
Salvar anexos na pasta
    ↓
Mover e-mail para: [PASTA_OUTLOOK_PROCESSAMENTO]
    ↓
Aguardar chegada do HAWB (PDF)
    ↓
Se HAWB encontrado → executar fluxo desktop
Senão → enviar e-mail de alerta para [EMAIL_ALERTA]
```

---

### Fluxo desktop (conferência e follow-up)

```text
Login no FollowNet
    ↓
Buscar processo por HAWB/IMP
    ↓
Abrir ocorrência
    ↓
Executar conferência completa (OCR + validações)
    ↓
Comparar dados HAWB x Invoice x Packing List
    ↓
Validar frete, pesos, volumes e SKUs
    ↓
Preencher ocorrência no FollowNet
    ↓
Finalizar processo
```

---

## 📂 Estrutura de Diretórios (SharePoint)

**Site:** [URL_SHAREPOINT]
**Caminho base:** [CAMINHO_BASE]

Exemplo:

```text
/COMEX - Conferência Pré-Alerta DHL/
└── HAWB <número>/
    ├── InvoicePackinglist <número>_<arquivo>.pdf
    └── demais anexos
```

---

## ⚙️ Configuração de Ambiente

| Parâmetro        | Tipo   | Padrão   | Descrição             |
| ---------------- | ------ | -------- | --------------------- |
| Ambiente         | String | DEV      | Define DEV ou PRD     |
| Modo de execução | String | attended | attended / unattended |

---

## 🧩 Principais Fluxos

| Fluxo                                              | Descrição                          |
| -------------------------------------------------- | ---------------------------------- |
| COMEX-Obtemanexosdee-mailHAWB                      | Processa alerta DHL e salva anexos |
| COMEX-ObtemeconferedocumentoDHLHAWBeInvoicePacking | Processa e-mails de lançamento     |
| COMEX-ConfernciadedocumentosdeembarqueDHLV6        | Conferência desktop completa       |
| COMEX-LeitorInvoiceePackingList                    | Leitura OCR de documentos          |
| COMEX-LoginFollowNet                               | Login automático no FollowNet      |

---

## 📧 Regras de Disparo

* E-mails processados são movidos para: **[PASTA_OUTLOOK_PROCESSAMENTO]**
* Alertas de erro são enviados para: **[EMAIL_ALERTA]**
* Processamento baseado em HAWB e IMP extraídos de assunto/corpo

---

## 🚨 Tratamento de Erros

* Retry automático configurado no fluxo de nuvem
* Falha de HAWB gera notificação por e-mail
* Arquivos não PDF são removidos automaticamente
* Execução serial (sem paralelismo)

---

## 📝 Logs e Evidências

* Histórico mantido no SharePoint por pasta de HAWB
* E-mails arquivados em pasta de processamento
* Ocorrências registradas no FollowNet com validações completas

---

## 🔧 Dependências

* Conta Office 365 ([CONTA_CORPORATIVA])
* Site SharePoint: [URL_SHAREPOINT]
* Agente UiFlow instalado (ID: [AGENT_ID])
* Conexões Power Automate configuradas:

  * [CONN_OFFICE365]
  * [CONN_SHAREPOINT]
  * [CONN_UIFLOW]
  * [CONN_CONVERSION]

---

## 🔐 Segurança

* Nenhuma credencial em texto plano
* Autenticação via connection references
* Acesso restrito por grupo no SharePoint
* Execução desktop baseada em sessão segura

---

## 👥 Time Responsável

* RPA / Automação Power Platform
* Equipe COMEX
* Infraestrutura SharePoint
* Sistema FollowNet

---

## 📄 Licença

Uso interno corporativo. Proibida redistribuição sem autorização.

---
