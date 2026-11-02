# Projeto ERP — Sibels Elegance

![Disciplina](https://img.shields.io/badge/Disciplina-Modelagem%20de%20Dados-6f42c1)
![Entrega](https://img.shields.io/badge/Entrega-Final%20-2ea44f)
![Banco](https://img.shields.io/badge/MySQL-8.0.16%2B-4479A1?logo=mysql&logoColor=white)
![DER](https://img.shields.io/badge/DER-Nota%C3%A7%C3%A3o%20de%20Chen-orange)

Projeto Integrador da disciplina de **Modelagem de Dados** (Grupo 03). A equipe atuou como uma startup contratada para organizar as informações da **Sibels Elegance**, microempresa familiar de semijoias com pedras naturais que vende por lives no Instagram e pelo WhatsApp e que hoje controla vendas, estoque, clientes e fornecedores em um caderno e em conversas soltas.

O trabalho percorre todo o caminho **do problema real ao banco de dados**: levantamento com a dona do negócio, processos, problemas, requisitos, regras de negócio, modelo conceitual (DER em notação de Chen), modelo lógico e script SQL para MySQL.

> 📄 **Documentação completa:** [Projeto ERP — Sibels Elegance (PDF, 145 páginas)](Projeto_ERP_Sibels_Elegance/Projeto%20ERP%20%E2%80%94%20Sibels%20Elegance%20.pdf)
>
> Este README é um guia do repositório. Todas as decisões, regras e justificativas estão detalhadas no documento.

---

## Sumário

1. [Sobre o projeto](#1-sobre-o-projeto)
2. [A empresa](#2-a-empresa)
3. [Problemas e necessidades](#3-problemas-e-necessidades)
4. [O projeto em números](#4-o-projeto-em-números)
5. [Processos de negócio e fluxogramas](#5-processos-de-negócio-e-fluxogramas)
6. [Modelo conceitual (DER)](#6-modelo-conceitual-der)
7. [Modelo lógico](#7-modelo-lógico)
8. [Script SQL](#8-script-sql)
9. [Como executar](#9-como-executar)
10. [Estrutura do repositório](#10-estrutura-do-repositório)
11. [Guia da documentação](#11-guia-da-documentação)
12. [Códigos de rastreabilidade](#12-códigos-de-rastreabilidade)

---

## 1. Sobre o projeto

O trabalho seguiu, uma etapa por vez, o caminho:

```
EMPRESA → PROCESSOS → PROBLEMAS → REQUISITOS → REGRAS DE NEGÓCIO
        → ENTIDADES → ATRIBUTOS → RELACIONAMENTOS → CARDINALIDADES → DER
        → MODELO LÓGICO → SCRIPT SQL
```

Cada etapa foi validada com a dona do negócio antes de seguir para a próxima, e o DER foi construído por último, como consequência da análise — não como ponto de partida.

- **Levantamento:** 103 perguntas à administração da empresa e 8 suposições submetidas a confirmação (S01 a S08).
- **Validações:** 40 validações com a empresa (VAL01 a VAL40), que alteraram o modelo e estão incorporadas à documentação.
- **Rastreabilidade:** toda decisão remete ao código de uma resposta da entrevista ou de uma validação. No script SQL, cada tabela, coluna e restrição traz em comentário o código da regra que a sustenta.

> O documento da entrevista (etapa 0) fica com a equipe e não é publicado neste repositório (VAL23).

## 2. A empresa

A **Sibels Elegance** ([@sibels_elegance](https://www.instagram.com/sibels_elegance/)) é uma microempresa familiar administrada por mãe e filho, que vende colares e pulseiras com pedras naturais. Funciona como loja virtual, sem loja física e sem marketplace.

| Aspecto | Como funciona hoje |
|---|---|
| **Vendas** | Lives no Instagram duas vezes por mês, de 6 a 7 horas, concentram cerca de 80% das vendas; direct e WhatsApp respondem pelo restante |
| **Reserva** | A cliente "fecha" a peça comentando no chat; vale a ordem do comentário, e a reserva dura até 3 dias |
| **Grupo VIP** | Peça não vendida vai para um grupo VIP no WhatsApp pelo mesmo preço e volta à live seguinte |
| **Estoque** | Controlado por quantidade do modelo; procura acima do saldo vira **Manifestação de Interesse** |
| **Produção** | Peças são desmontadas e remontadas a partir de fio e componentes |
| **Pagamento** | Crédito, débito e Pix; até 10x sem juros a partir de R$ 400; desconto só no Pix (5%, 10% para cliente fiel e 12% para revendedora) |
| **Envio** | Superfrete, frete fixo de R$ 20, remessa única com código de rastreio |
| **Compras** | 6 a 7 importadores, pagamento à vista, sem registro em sistema |
| **Pós-venda** | Garantia de um ano sobre o acabamento, com conserto gratuito |
| **Controles** | Um caderno e conversas no WhatsApp; nunca foi feito inventário |

## 3. Problemas e necessidades

O levantamento identificou **24 problemas** (P01 a P24) e **15 necessidades** (N01 a N15). Alguns dos principais e como o modelo responde a eles:

| Problema | Como o modelo resolve |
|---|---|
| A ordem do comentário decide quem leva a peça, mas nenhum horário é registrado (P03) | `reserva` guarda o momento exato do comentário e o momento da reserva, de onde se conta o prazo de 3 dias |
| Nunca foi feito inventário e o saldo real é desconhecido (P05) | Todo saldo resulta de `movimento_estoque`, com motivo obrigatório e uma única origem; `inventario` confere e ajusta |
| Peças desmontadas para gerar outras sem registro (P06) | `producao` e `consumo_producao` registram a baixa da origem, o fio consumido e a entrada da peça nova, com custo |
| Compras aos importadores não registradas (P09) | `compra`, `item_compra` e `desembolso` separam o que foi comprado do dinheiro que saiu do caixa |
| A cliente é identificada pelo @ do Instagram, que pode mudar (P17) | `cliente` é identificada pelo CPF; contatos e endereços ficam em tabelas próprias, com histórico |
| CPF, telefone e endereço coletados sem consentimento (P22) | `consentimento` registra data, finalidade, meio e revogação (LGPD) |
| A Manifestação de Interesse se perde ao fim da live (P23) | `manifestacao_interesse` registra cada manifestação (pessoa, peça, momento) e orienta a reposição |
| Não se sabe quanto dinheiro entra e sai em cada mês (P24) | `periodo_caixa` apura o caixa mensal com pagamentos recebidos e desembolsos |

Uma das propostas do projeto é um **Bot de atendimento** (PR12) que monitora os comentários com o código da peça, chama uma interessada por vez no privado, identifica a cliente pelo CPF ou conduz o cadastro e cria a reserva — registrando cada chamada e cada etapa, com prazo e resposta.

## 4. O projeto em números

| Artefato | Quantidade |
|---|---|
| Processos de negócio | 13 (PR01 a PR13) |
| Fluxogramas | 11 (F01 a F11) |
| Requisitos funcionais | 88 |
| Requisitos não funcionais | 15 (RNF07 removido em revisão) |
| Regras de negócio | 104 |
| Políticas organizacionais | 13 |
| Entidades | 35 (+ 2 especializações: PEÇA DE VENDA e INSUMO) |
| Relacionamentos | 56 ativos, sendo 6 N:N |
| Atributos derivados (calculados, não armazenados) | 57 |
| Tabelas no modelo lógico | 39 |
| Testes de restrição (devem falhar) | 58 |
| Consultas gerenciais | 15 |
| Consultas de auditoria | 40 |

## 5. Processos de negócio e fluxogramas

| Código | Processo | Fluxograma |
|---|---|---|
| PR01 | Compra e reposição junto a importadores | [F01](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F01%20-%20Compra%20e%20reposi%C3%A7%C3%A3o%20junto%20a%20importadores%20%28PR01%29.png) |
| PR02 | Produção: montagem e desmontagem de peças | [F02](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F02%20-%20Produ%C3%A7%C3%A3o%20-%20desmontagem%20e%20montagem.png) |
| PR03 | Preparação da live | [F03](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F03%20-%20Prepara%C3%A7%C3%A3o%20e%20realiza%C3%A7%C3%A3o%20da%20live%20%28PR03%20e%20PR04%29.png) |
| PR04 | Realização da live | [F03](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F03%20-%20Prepara%C3%A7%C3%A3o%20e%20realiza%C3%A7%C3%A3o%20da%20live%20%28PR03%20e%20PR04%29.png) |
| PR05 | Reserva da peça e cadastro da cliente | [F04](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F04%20-%20Reserva%20e%20Manifesta%C3%A7%C3%A3o%20de%20Interesse%20%28PR05%29.png) |
| PR06 | Remanejamento da peça não vendida | [F05](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F05%20-%20Remanejamento%20da%20pe%C3%A7a%20n%C3%A3o%20vendida%20%28PR06%29.png) |
| PR07 | Formação e fechamento do pedido | [F06](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F06%20-%20Forma%C3%A7%C3%A3o%20do%20pedido%20e%20pagamento%20%28PR07%20e%20PR08%29.png) |
| PR08 | Pagamento | [F06](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F06%20-%20Forma%C3%A7%C3%A3o%20do%20pedido%20e%20pagamento%20%28PR07%20e%20PR08%29.png) |
| PR09 | Expedição e envio | [F07](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F07%20-%20Expedi%C3%A7%C3%A3o%20e%20envio%20%28PR09%29.png) |
| PR10 | Pós-venda: garantia, conserto e avaria | [F08](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F08%20-%20P%C3%B3s-venda%2C%20garantia%20e%20conserto%20%28PR10%29.png) |
| PR11 | Apuração de custo e resultado | [F09 — visão integrada](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F09%20-%20Vis%C3%A3o%20integrada%20dos%20processos.png) |
| PR12 | Atendimento automatizado da Manifestação de Interesse (Bot) — proposta | [F10](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F10%20-%20Atendimento%20automatizado%20da%20Manifesta%C3%A7%C3%A3o%20de%20Interesse%20pelo%20Bot%20%28PR12%29.png) |
| PR13 | Apuração do caixa do mês | [F11](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F11%20-%20Apura%C3%A7%C3%A3o%20do%20caixa%20do%20m%C3%AAs%20%28PR13%29.png) |

### Visão integrada dos processos (F09)

O estoque é o ponto de convergência de compra, produção, venda e pós-venda; a Manifestação de Interesse realimenta a decisão de compra; e a apuração de resultado depende de compra, estoque, pedido e envio estarem registrados.

<p align="center">
  <img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F09%20-%20Vis%C3%A3o%20integrada%20dos%20processos.png" alt="F09 — Visão integrada dos processos" width="760">
</p>

<details>
<summary><strong>Ver todos os fluxogramas (F01 a F11)</strong></summary>

#### F01 — Compra e reposição junto a importadores (PR01)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F01%20-%20Compra%20e%20reposi%C3%A7%C3%A3o%20junto%20a%20importadores%20%28PR01%29.png" alt="F01" width="480">

#### F02 — Produção: desmontagem e montagem
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F02%20-%20Produ%C3%A7%C3%A3o%20-%20desmontagem%20e%20montagem.png" alt="F02" width="360">

#### F03 — Preparação e realização da live (PR03 e PR04)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F03%20-%20Prepara%C3%A7%C3%A3o%20e%20realiza%C3%A7%C3%A3o%20da%20live%20%28PR03%20e%20PR04%29.png" alt="F03" width="380">

#### F04 — Reserva e Manifestação de Interesse (PR05)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F04%20-%20Reserva%20e%20Manifesta%C3%A7%C3%A3o%20de%20Interesse%20%28PR05%29.png" alt="F04" width="560">

#### F05 — Remanejamento da peça não vendida (PR06)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F05%20-%20Remanejamento%20da%20pe%C3%A7a%20n%C3%A3o%20vendida%20%28PR06%29.png" alt="F05" width="520">

#### F06 — Formação do pedido e pagamento (PR07 e PR08)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F06%20-%20Forma%C3%A7%C3%A3o%20do%20pedido%20e%20pagamento%20%28PR07%20e%20PR08%29.png" alt="F06" width="520">

#### F07 — Expedição e envio (PR09)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F07%20-%20Expedi%C3%A7%C3%A3o%20e%20envio%20%28PR09%29.png" alt="F07" width="320">

#### F08 — Pós-venda, garantia e conserto (PR10)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F08%20-%20P%C3%B3s-venda%2C%20garantia%20e%20conserto%20%28PR10%29.png" alt="F08" width="520">

#### F10 — Atendimento automatizado da Manifestação de Interesse pelo Bot (PR12)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F10%20-%20Atendimento%20automatizado%20da%20Manifesta%C3%A7%C3%A3o%20de%20Interesse%20pelo%20Bot%20%28PR12%29.png" alt="F10" width="360">

#### F11 — Apuração do caixa do mês (PR13)
<img src="Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Fluxograma/F11%20-%20Apura%C3%A7%C3%A3o%20do%20caixa%20do%20m%C3%AAs%20%28PR13%29.png" alt="F11" width="360">

</details>

## 6. Modelo conceitual (DER)

O DER segue a **notação de Chen** (ER estendido, Elmasri & Navathe) e representa o modelo final descrito nas seções 11 a 16 da documentação. Por ser conceitual, não mostra chaves estrangeiras.

- 🖼️ **DER em alta resolução:** [FLUXOGRAMA - DER.pdf](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/DER_Entidade_Relacionamento/FLUXOGRAMA%20-%20DER.pdf)
- ✏️ **DER editável (Lucidchart):** [abrir no Lucidchart](https://lucid.app/lucidchart/839f2062-56c8-4f7a-a553-64fbf70ef6d1/edit?viewport_loc=-3400%2C-1318%2C13695%2C7008%2CB0yy4WxB76_J&invitationId=inv_dfba602e-0bd9-45c9-95a5-2f407be9ab56)

### Entidades por núcleo

| Núcleo | Entidades |
|---|---|
| **Vendas** | CLIENTE, CONTATO DA CLIENTE, ENDEREÇO, CEP, CONSENTIMENTO, PEDRA, RESERVA, PEDIDO, PAGAMENTO, VALE-COMPRAS, ENVIO, ATENDIMENTO DE PÓS-VENDA |
| **Demanda não atendida** | MANIFESTAÇÃO DE INTERESSE, CHAMADA DA MANIFESTAÇÃO DE INTERESSE, ETAPA DA CHAMADA |
| **Estoque** | PRODUTO (especializado em PEÇA DE VENDA e INSUMO), FOTO DO PRODUTO, MOVIMENTO DE ESTOQUE, PRODUÇÃO, CONSUMO NA PRODUÇÃO, INVENTÁRIO, ITEM DE INVENTÁRIO, FORNECEDOR, CONTATO DO FORNECEDOR, COMPRA, ITEM DE COMPRA |
| **Custo e resultado** | COMPONENTE DE CUSTO, DESEMBOLSO, PERÍODO DE CAIXA |
| **Live e promoção** | LIVE, APRESENTAÇÃO NA LIVE, OFERTA NO GRUPO VIP, SORTEIO |
| **Controle** | USUÁRIO (inclui o Bot, com perfil próprio), REGISTRO DE OPERAÇÃO |

### Decisões de modelagem que mudaram o rumo do trabalho

1. **A premissa da peça única caiu na entrevista.** O controle é por quantidade do modelo, e a procura acima do saldo vira Manifestação de Interesse.
2. **A Manifestação de Interesse virou entidade.** Era uma informação que a empresa produzia em toda live e descartava — justamente a que deveria orientar a compra junto aos importadores.
3. **A verificação dos N:N eliminou um caminho redundante** entre pedido e produto, que teria gerado inconsistência no modelo lógico.
4. **A conferência das cardinalidades corrigiu a direção entre reserva e pedido:** um pedido reúne várias reservas, e não o contrário.

## 7. Modelo lógico

O modelo conceitual foi convertido em **39 tabelas**: uma para cada uma das 35 entidades, duas para as especializações PEÇA DE VENDA e INSUMO e duas para os relacionamentos N:N sem atributos (`pedra_peca` e `preferencia_pedra`).

- 🗂️ **Modelo no MySQL Workbench:** [modelo lógico.mwb](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Modelo%20l%C3%B3gico/modelo%20l%C3%B3gico.mwb) — veja [como abrir o modelo no Workbench](Projeto_ERP_Sibels_Elegance/Documentos%20complementares/Modelo%20l%C3%B3gico/Como_abrir_modelo_MySQL_Workbench.pdf) (use **File > Open Model...**, e não "Open SQL Script")

**Regras de mapeamento**

| Situação no conceitual | Como ficou no lógico |
|---|---|
| Entidade | Uma tabela; o identificador `id_…` vira chave primária |
| Relacionamento 1:N | Chave estrangeira no lado N, `NOT NULL` quando a participação é obrigatória |
| Relacionamento 1:1 (R10, R36, R41) | Chave estrangeira no lado definido em 14.9, com `UNIQUE` |
| Relacionamento N:N | Tabela associativa; as quatro com atributos têm chave própria e `UNIQUE` no par de FKs; `pedra_peca` e `preferencia_pedra` têm chave primária composta |
| Especialização total e exclusiva | Tabela `produto` com o discriminador `tipo_produto`; `peca_venda` e `insumo` com `id_produto` como PK e FK |
| Entidades com mais de uma origem | Uma FK opcional por origem e `CHECK` de exclusividade (`ck_movimento_origem`, `ck_componente_dona`, `ck_reserva_origem`, `ck_manifestacao_origem`, `ck_desembolso_compra`) |
| Atributos derivados | Não viram colunas: são calculados nas consultas, para nunca ficarem desatualizados |

**Tabelas por área**

| Área | Tabelas |
|---|---|
| Cliente e dados pessoais | `cep`, `cliente`, `endereco`, `contato_cliente`, `consentimento`, `pedra`, `preferencia_pedra` |
| Acesso e auditoria | `usuario`, `registro_operacao` |
| Produto, estoque e produção | `produto`, `peca_venda`, `insumo`, `pedra_peca`, `foto_produto`, `movimento_estoque`, `inventario`, `item_inventario`, `producao`, `consumo_producao`, `componente_custo` |
| Compras | `fornecedor`, `contato_fornecedor`, `compra`, `item_compra` |
| Financeiro | `desembolso`, `periodo_caixa` |
| Live, grupo VIP e promoções | `live`, `apresentacao_live`, `oferta_grupo_vip`, `sorteio`, `vale_compras` |
| Reserva e Manifestação de Interesse | `reserva`, `manifestacao_interesse`, `chamada_interesse`, `etapa_chamada` |
| Pedido, pagamento e entrega | `pedido`, `pagamento`, `envio`, `atendimento_pos_venda` |

O dicionário de dados físico (tabelas, colunas, tipos e restrições) está no **Apêndice C** da documentação.

## 8. Script SQL

Implementado em **MySQL 8.0.16 ou superior**, usando apenas `CREATE TABLE`, `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `CHECK` e `DEFAULT`. Todas as chaves estrangeiras são criadas no próprio `CREATE TABLE`, na ordem de dependência, sem referência circular. As consultas não usam `VIEW`, `CASE`, `COALESCE` nem funções de texto e de data.

| Ordem | Script | O que faz |
|:-:|---|---|
| 1 | `sql/ddl/sibels_01_criacao.sql` | Cria o banco `sibels_elegance` e as 39 tabelas, com todas as chaves e restrições |
| 2 | `sql/dados/sibels_02_carga.sql` | Carga válida e coerente de agosto a outubro de 2026 (datas fixas) |
| 3 | `sql/dados/sibels_03_testes_invalidos.sql` | 58 testes de restrição: cada comando **deve falhar** |
| 4 | `sql/dados/sibels_04_atualizacoes.sql` | Operações do dia a dia com `UPDATE` e `DELETE` (quitação, envio etc.) |
| 5 | `sql/consultas/sibels_05_consultas_gerenciais.sql` | 15 consultas gerenciais (G01 a G15) |
| 6 | `sql/auditoria/sibels_06_consultas_auditoria.sql` | 40 consultas de auditoria (A01 a A40) para as regras que o banco não garante sozinho |

### Consultas gerenciais

Cada consulta parte de uma pergunta do negócio e indica a decisão que apoia:

| Código | Pergunta |
|---|---|
| G01 | Quais peças mais vendem, em unidades e em valor? |
| G02 | Quais clientes mais compram (valor acumulado dos pedidos quitados)? |
| G03 | Quanto falta para cada cliente comum virar fiel (R$ 10.000 acumulados)? |
| G04 | Qual o resultado de cada live? |
| G05 | Quanto vendemos por mês e com qual ticket médio? |
| G06 | Qual a margem de cada peça vendida (preço praticado × custo atribuído na saída)? |
| G07 | Quanto compramos de cada fornecedor, quanto pagamos e se entregam no prazo? |
| G08 | Quais pedidos ainda têm saldo a receber e até quando? |
| G09 | Como fechou o caixa de cada mês? |
| G10 | Quem está na lista de espera do Brinco Gota Quartzo (BG03), em ordem? |
| G11 | Quais peças estão paradas há mais de 50 dias e ainda têm saldo? |
| G12 | Quais clientes estão cadastradas e nunca fizeram pedido? |
| G13 | Por qual canal as vendas acontecem e qual o ticket médio por reserva? |
| G14 | Quanto custa receber no cartão (taxa da operadora por forma de pagamento)? |
| G15 | Quantas manifestações de interesse viram venda pelo Bot? |

### Consultas de auditoria

Regras que dependem de outra tabela ou da data — como o prazo de 3 dias da reserva, o perfil fiel pelo valor acumulado, o saldo nunca negativo e o envio só após a quitação — não cabem em `CHECK`. Elas estão listadas no fim do script de criação e são verificadas pelas consultas **A01 a A40**: cada consulta lista as ocorrências que violam a regra, e **resultado vazio significa regra cumprida** (a A09 é uma lista de conferência e sempre traz linhas).

## 9. Como executar

**Pré-requisito:** MySQL 8.0.16 ou superior (versão a partir da qual o `CHECK` é aplicado) e um servidor **sem** o banco `sibels_elegance`.

Pelo terminal, a partir da raiz do repositório:

```bash
# 1. Criação do banco e das tabelas
mysql -u root -p < sql/ddl/sibels_01_criacao.sql

# 2. Carga de dados
mysql -u root -p < sql/dados/sibels_02_carga.sql

# 3. Testes de restrição — todos devem falhar.
#    --force faz o cliente seguir para o próximo comando após cada erro
mysql -u root -p --force < sql/dados/sibels_03_testes_invalidos.sql

# 4. Atualizações do dia a dia
mysql -u root -p < sql/dados/sibels_04_atualizacoes.sql

# 5 e 6. Consultas gerenciais e de auditoria
mysql -u root -p -t < sql/consultas/sibels_05_consultas_gerenciais.sql
mysql -u root -p -t < sql/auditoria/sibels_06_consultas_auditoria.sql
```

No **MySQL Workbench**, abra cada script em **File > Open SQL Script...** e execute na mesma ordem. No script 3, execute **um comando por vez** (ou ative *Continue on SQL Query Errors*), já que cada comando deve gerar erro.

> Para recomeçar do zero: `DROP DATABASE sibels_elegance;`

## 10. Estrutura do repositório

```
.
├── README.md
└── Projeto_ERP_Sibels_Elegance/
    ├── Projeto ERP — Sibels Elegance .pdf              # documentação completa (v2.5.0)
    └── Documentos complementares/
        ├── DER_Entidade_Relacionamento/
        │   └── FLUXOGRAMA - DER.pdf                     # DER em notação de Chen
        ├── Diario_Grupo_03/
        │   └── Diario_Grupo_03_<data>.pdf               # diários de bordo da equipe
        ├── Fluxograma/
        │   └── F01 … F11 - <processo>.png               # fluxogramas dos processos
        └── Modelo lógico/
            ├── modelo lógico.mwb                        # modelo lógico (MySQL Workbench)
            └── Como_abrir_modelo_MySQL_Workbench.pdf    # passo a passo para abrir o .mwb
```

## 11. Guia da documentação

| Seção | Conteúdo | Página |
|---|---|:-:|
| 1 | Introdução e artefatos do projeto | 1 |
| 2 | Caracterização da empresa (inclui diagnóstico financeiro) | 2 |
| 3 | Justificativa da escolha do negócio | 7 |
| 4 | Problemas e necessidades identificados | 9 |
| 5 | Processos de negócio | 11 |
| 6 | Requisitos funcionais | 16 |
| 7 | Requisitos não funcionais | 21 |
| 8 | Regras de negócio | 22 |
| 9 | Restrições e políticas organizacionais | 32 |
| 10 | Fluxogramas dos principais processos | 34 |
| 11 | Entidades | 44 |
| 12 | Atributos | 48 |
| 13 | Relacionamentos | 58 |
| 14 | Cardinalidades | 61 |
| 15 | Dicionário de dados conceitual preliminar | 74 |
| 16 | DER — Diagrama Entidade-Relacionamento | 91 |
| 17 | Justificativas técnicas | 94 |
| 18 | Modelo lógico | 100 |
| 19 | Conclusões | 105 |
| A | Apêndice A — Cenários de verificação | 105 |
| B | Apêndice B — Matriz de rastreabilidade das regras revisadas | 107 |
| C | Apêndice C — Dicionário de dados físico | 108 |
| D | Apêndice D — Script SQL | 118 |

## 12. Códigos de rastreabilidade

| Prefixo | Significado |
|---|---|
| LV, RS, PG, EN, ET, FR, CT, CL, PV, AC, S | Respostas da entrevista (live, reserva, pagamento, envio, estoque, fornecedores, custo, cliente, pós-venda, acesso e suposições) |
| VAL | Validações com a empresa |
| P / N | Problemas / necessidades |
| PR / F | Processos de negócio / fluxogramas |
| RF / RNF | Requisitos funcionais / não funcionais |
| RN / POL | Regras de negócio / políticas organizacionais |
| E / R | Entidades / relacionamentos |
| G / A | Consultas gerenciais / consultas de auditoria |

---

<sub>Projeto acadêmico desenvolvido no Projeto Integrador da disciplina de Modelagem de Dados, com a Sibels Elegance como estudo de caso real.</sub>
