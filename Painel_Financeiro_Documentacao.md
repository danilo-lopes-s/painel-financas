# Painel Financeiro Pessoal — documentação funcional e guia de uso

**Fonte desta documentação:** arquivo `Painel_Financeiro_Pessoal_com_Edicao_Completa.html`, versão mais recente usada nesta revisão.  
**Revisão da documentação:** 13/10/2026.  
**Idioma da interface:** português do Brasil.  
**Formato da aplicação:** arquivo HTML independente, com CSS, JavaScript e dados financeiros incorporados no próprio arquivo.

> Esta revisão documenta também os controles de edição adicionados aos cadastros de contas e cartões, movimentos de poupança/previdência, aplicações e tipos personalizados de investimento, além dos caminhos de edição já existentes nos demais módulos e do controle contextual **Voltar ao início** em listas longas.

> Esta documentação foi preparada a partir da estrutura, dos formulários, das funções e dos dados existentes no HTML anexado. Os nomes dos campos e das telas acompanham a implementação encontrada no código. Onde há estimativas, limitações ou regras específicas, elas são descritas como o sistema realmente as implementa — não como recomendações financeiras gerais.

## Sumário

- [O que o código faz](#o-que-o-código-faz)
- [Papel e Arquitetura](#papel-e-arquitetura)
- [Estrutura e Funcionamento](#estrutura-e-funcionamento)
- [1. Navegação geral e período](#1-navegação-geral-e-período)
- [2. Visão Geral](#2-visão-geral)
- [3. Calendário](#3-calendário)
- [4. Contas e Transferências](#4-contas-e-transferências)
- [5. Cartões e Parcelamentos](#5-cartões-e-parcelamentos)
- [6. Despesas](#6-despesas)
- [7. Recebimentos](#7-recebimentos)
- [8. Metas a Receber](#8-metas-a-receber)
- [9. Categorias](#9-categorias)
- [10. Investimentos](#10-investimentos)
- [11. Lixeira](#11-lixeira)
- [12. Composição do Saldo Líquido e evolução histórica](#12-composição-do-saldo-líquido-e-evolução-histórica)
- [13. Formulários: campos e preenchimento correto](#13-formulários-campos-e-preenchimento-correto)
  - [13.3 Edição de cadastros e lançamentos](#133-edição-de-cadastros-e-lançamentos)
- [14. Regras de cálculo, status e consistência](#14-regras-de-cálculo-status-e-consistência)
- [15. Salvar, alterações pendentes e recuperação dos dados](#15-salvar-alterações-pendentes-e-recuperação-dos-dados)
- [16. Estado dos dados existentes no arquivo anexado](#16-estado-dos-dados-existentes-no-arquivo-anexado)
- [17. Glossário simples](#17-glossário-simples)
- [Resumo geral](#resumo-geral)

---

## O que o código faz

O Painel Financeiro Pessoal é uma aplicação local para organizar e consultar informações de finanças pessoais. Ele reúne cadastros de contas bancárias, cartões, despesas, recebimentos, metas de recebimento, categorias, investimentos e transferências. Também oferece gráficos, indicadores, navegação por período, edição de campos, exclusão com lixeira e restauração de registros.

O painel trabalha com lançamentos datados. A data define em qual mês ou dia o lançamento aparece nas listas, no calendário e em boa parte dos indicadores. O sistema diferencia despesas pagas, pendentes e canceladas; recebimentos recebidos, pendentes e cancelados; e metas planejadas, confirmadas ou canceladas.

Os investimentos ficam em uma carteira separada dos saldos correntes e dos lançamentos de receitas e despesas. A aplicação calcula indicadores de investimentos usando os aportes, resgates, compras, vendas, taxas, cotações manuais e parâmetros cadastrados. Os resultados de renda fixa são projeções baseadas nas hipóteses informadas pelo usuário, e não cotações ou previsões garantidas.

A aplicação também oferece uma lixeira. Em vez de remover imediatamente muitos cadastros ou lançamentos relacionados, o sistema os guarda em uma entrada de lixeira, permitindo restaurá-los. A restauração recompõe registros e, quando necessário, reaplica efeitos nos saldos. A exclusão definitiva e o esvaziamento da lixeira são operações irreversíveis dentro do painel.

## Papel e Arquitetura

### 1. Arquivo único e independente

A aplicação está concentrada em um arquivo `.html`. O próprio documento contém:

- **HTML:** estrutura da interface, cartões de navegação, áreas de conteúdo, tabelas, formulários e janelas modais.
- **CSS:** cores, disposição dos painéis, cartões, tabelas, gráficos, botões, tema escuro e ajustes para telas menores.
- **JavaScript:** regras de negócio, cálculos, filtros, atualização das telas, tratamento dos lançamentos, gráficos, exclusões, restaurações e salvamento do arquivo.
- **Dados incorporados:** o elemento `<script id="dados_injetados">` carrega o objeto `window.INJETADO`, serializado dentro do HTML.

O código informa expressamente que a aplicação carrega os dados incorporados no arquivo e não usa um armazenamento compartilhado do navegador para manter os dados financeiros. Por isso, o arquivo HTML salvo é a principal cópia persistente do painel.

### 2. Como o painel é iniciado

A inicialização ocorre pela função `boot()`:

1. Lê o objeto `window.INJETADO` já incorporado no HTML.
2. Copia seus dados para o objeto de trabalho `dados`.
3. Garante que coleções essenciais — como investimentos, transferências, movimentos de conta, tipos de investimento e lixeira — existam no formato esperado.
4. Prepara dados de demonstração apenas quando as flags e as condições do arquivo permitem. No arquivo anexado, a flag `zerado` está marcada como `true`, e os fluxos de demonstração que verificam essa flag não devem acrescentar suas coleções fictícias.
5. Restaura a ordem salva dos cartões de navegação.
6. Atualiza o seletor de período, redesenha o painel e habilita a funcionalidade de arrastar e soltar.

O mês inicial é definido pelo relógio do computador/navegador ao carregar o arquivo. A marcação HTML pode conter textos que foram renderizados em uma sessão anterior; depois da inicialização, as telas são redesenhadas usando o estado e o período atuais.

### 3. Estrutura central dos dados

O objeto `dados` organiza as informações em coleções. As principais propriedades utilizadas pelo código são:

| Propriedade | Conteúdo |
|---|---|
| `contas` | Cadastros de contas e seus saldos. |
| `cartoes` | Cadastros dos cartões, limites, vencimentos e dados complementares. |
| `despesas` | Despesas únicas, despesas em cartão, parcelas e séries recorrentes antigas ou atuais. |
| `recebimentos` | Entradas recebidas, pendentes ou canceladas. |
| `planejamentos` | Metas de recebimento previstas e seu estado de confirmação. |
| `categorias` | Nome, ícone e cor de cada categoria. |
| `investimentos` | Aplicações cadastradas e seus movimentos internos. |
| `transferencias` | Transferências entre dinheiro físico e contas. |
| `tiposInvestimento` | Tipos personalizados de investimentos. |
| `movContas` | Movimentos manuais de contas poupança e previdência, incluindo saldo inicial. |
| `lixeira` | Itens excluídos, detalhes da exclusão, registros guardados e metadados para restauração. |
| `dinheiroFisico` | Saldo de dinheiro em mãos, separado das contas cadastradas. |
| `navOrder` | Ordem personalizada dos cartões de navegação. |
| `timestamp` | Marca temporal atualizada quando o arquivo é salvo. |

O código também usa flags, como `zerado`, `exemplosV13`, `exemplosInvestV17` e `demoContasV18`, para controlar a inicialização e impedir que algumas demonstrações sejam inseridas repetidamente.

### 4. Como as telas são atualizadas

A função `refresh()` centraliza a atualização após mudanças importantes. Ela garante a estrutura dos dados, processa despesas agendadas quando aplicável, recalcula os totais dos cartões laterais, atualiza a composição do saldo, redesenha o gráfico histórico e renderiza a tela atualmente selecionada.

A navegação é controlada por variáveis como `visao`, `itemAtual` e `historicoVisoes`. `selecionarVisao()` abre uma tela ou um detalhe; `navegarVoltar()` retorna à tela anterior registrada no histórico.

Os gráficos usam elementos HTML e SVG construídos pelo próprio JavaScript. Alguns são clicáveis: clicar em uma barra, fatia, legenda ou ponto pode abrir uma composição, uma tabela de lançamentos ou os detalhes de uma conta ou investimento.

### 5. Separação entre interface, cálculo e dados

O código não utiliza um servidor ou banco remoto para realizar as operações descritas. Os cálculos e as alterações são executados no navegador, sobre o objeto de dados carregado do HTML. As informações dos investimentos, as taxas e as cotações usadas nas projeções são fornecidas manualmente pelo usuário.

## Estrutura e Funcionamento

## 1. Navegação geral e período

### 1.1 Cabeçalho superior

No topo da tela ficam quatro controles principais:

- **Modo escuro / Modo claro:** alterna a aparência visual do painel. A alteração é sinalizada como pendente até que o arquivo seja salvo.
- **Salvar Arquivo:** grava as informações atuais dentro de um arquivo HTML atualizado.
- **Alterações não salvas:** indicador de que houve alguma mudança desde o último salvamento concluído.
- **Zerar painel:** remove os cadastros e lançamentos abrangidos pela ação, depois de duas confirmações. As categorias são mantidas. Para tornar o estado zerado permanente, é necessário salvar o HTML; fechar sem salvar preserva o arquivo anterior.

### 1.2 Cartões de navegação à esquerda

A navegação lateral apresenta as áreas principais:

1. **Visão Geral**
2. **Calendário**
3. **Contas e Transferências**
4. **Cartões e Parcelamentos**
5. **Despesas**
6. **Metas a Receber**
7. **Recebimentos**
8. **Categorias**
9. **Investimentos**
10. **Lixeira**

O valor mostrado em cada cartão depende do módulo. Pode ser um total monetário, uma contagem de cadastros ou apenas um ícone. O cartão ativo recebe uma aparência destacada.

**Abrir uma área:** clique no cartão.  
**Reordenar os cartões laterais:** use a alça lateral de arrasto, e não o restante do cartão. A ordem personalizada é registrada nos dados do arquivo quando ele é salvo.

Algumas áreas abrem um menu intermediário. Por exemplo, Contas e Transferências abre as opções de contas e transferências; Cartões e Parcelamentos abre cartões, parcelamentos e despesas únicas na fatura.

### 1.3 Botões Voltar e navegação por detalhes

Ao clicar em um lançamento, conta, cartão, grupo de parcelas ou aplicação, o painel abre uma tela de detalhe. O botão **Voltar** utiliza o histórico de navegação interno para tentar retornar à tela anterior.

Em telas abertas a partir de gráficos, a legenda, barra ou fatia pode abrir um quadro de detalhes. Alguns desses quadros têm seu próprio botão para abrir o módulo ou registro relacionado.

### 1.4 Seletor global de período

Quando está disponível, o seletor de período aparece acima do conteúdo principal. Ele contém:

- **Hoje:** seleciona a data atual.
- **◀ e ▶:** retrocedem ou avançam no mês ou no dia, conforme o modo selecionado.
- **Exibir:** alterna entre `Mês / ano` e `Dia`.
- **Mês / ano:** seleciona um mês pelo controle de mês.
- **Dia:** seleciona uma data específica quando o modo diário está ativo.
- **Rótulo do período:** informa o mês/ano ou a data selecionada.

O período escolhido afeta as listas e métricas que usam o filtro global. Em algumas telas, como categorias e investimentos, o seletor é ocultado porque a tela apresenta uma visão de carteira/cadastro mais ampla. A tela de detalhe de um parcelamento também oculta esse seletor; o detalhe de uma despesa parcelada, por sua vez, usa o período selecionado para localizar a parcela daquele mês ou dia.

### 1.5 Voltar ao início em listas longas

Nas áreas que podem apresentar uma tabela ou lista extensa, o painel inclui o controle contextual **Voltar ao início**. Ele é inserido abaixo da tabela, da lista repetida ou de um bloco de informações que ultrapasse o tamanho considerado longo pelo código. O recurso contempla o conteúdo principal (`#contextContent`) e os quadros de detalhamento (`#quadroCorpo`).

O botão não fica permanentemente visível: no topo da visualização, ele permanece escondido e desativado. Quando a rolagem vertical ultrapassa um limite adaptado ao tamanho da janela, o controle aparece com transição suave de opacidade e movimento discreto. Ao clicar, a página e os contêineres roláveis associados retornam ao início com rolagem suave; se o dispositivo ou navegador indicar preferência por movimento reduzido, o retorno é imediato.

Listas curtas não recebem o botão quando não atingem os critérios de altura e quantidade definidos pela rotina. O recurso é apenas de navegação: não altera registros, filtros, saldos, cálculos ou a ordenação dos lançamentos.

---

## 2. Visão Geral

A Visão Geral resume o período selecionado e oferece caminhos para investigar os números.

### 2.1 Indicadores principais

- **Receitas no Período:** soma dos recebimentos datados dentro do período.
- **Despesas no Período:** soma das despesas datadas dentro do período.
- **Saldo do Período:** receitas menos despesas do período.
- **Metas no Período:** soma dos valores das metas previstas dentro do período. Uma meta planejada não é, por si só, um recebimento.

Os indicadores são clicáveis e abrem uma lista dos lançamentos que compõem o número selecionado.

### 2.2 Distribuição de gastos

A rosca e a lista lateral mostram como as despesas do período se distribuem por categoria. A categoria pode ser aberta clicando em sua legenda ou em uma faixa do gráfico. A lista de valores por categoria também é clicável e permite navegar até as despesas daquela categoria.

O percentual de uma categoria é calculado em relação ao total de despesas do período. Se não houver gastos, o painel exibe uma mensagem de ausência de dados em vez de uma distribuição útil.

### 2.3 Investimentos na Visão Geral

A área de investimentos mostra o patrimônio investido total, a parcela de renda fixa e a parcela de renda variável. Os indicadores abrem o respectivo detalhamento.

**Importante:** o próprio painel avisa que os investimentos são separados dos saldos bancários e do saldo do período. Não some o patrimônio investido duas vezes ao conferir as métricas de caixa.

---

## 3. Calendário

O Calendário apresenta uma grade do mês, com os dias da semana e os lançamentos associados a cada data.

### 3.1 Tipos de lançamento exibidos

- **Despesas:** aparecem como saídas, com sinal negativo quando a visualização usa valores.
- **Recebimentos:** aparecem como entradas, com sinal positivo quando a visualização usa valores.
- **Metas:** aparecem com uma identificação visual própria.

Cada lançamento ocupa um marcador no dia em que sua data está registrada. Clicar em um marcador abre os detalhes da despesa, do recebimento ou da meta correspondente.

### 3.2 Alternar entre valores e títulos

Acima da grade há dois botões:

- **Valores:** exibe o valor monetário do lançamento, precedido pelo sinal que representa entrada ou saída.
- **Títulos:** exibe o nome/descrição do lançamento.

Essa escolha altera apenas a forma de exibição dos marcadores. Não muda valores, datas ou status dos registros.

### 3.3 Selecionar um dia

Clicar em um quadrado de dia faz o painel entrar no modo `Dia`, selecionar aquela data e atualizar as telas que dependem do período. A borda destacada indica o dia selecionado.

### 3.4 Gráfico de movimentação

O gráfico do período compara valores de recebimentos, despesas e metas. As barras são clicáveis e levam à lista de registros correspondente.

---

## 4. Contas e Transferências

A área abre um menu com duas entradas: **Contas** e **Transferências**.

### 4.1 Visão de contas

A tela de contas reúne indicadores de saldo atual, entradas e saídas do período, patrimônio calculado, gráfico de saldos, tabela das contas cadastradas, cartões de tipos de conta e gráficos históricos.

A tabela mostra nome da conta, instituição (quando informada), tipo, valor atual e ações. O lápis (`✏️`) abre o formulário preenchido para editar o cadastro; a lixeira (`🗑️`) move a conta para a Lixeira. Clicar no restante da linha abre os detalhes da conta. Também é possível editar o cadastro no próprio detalhe da conta.

O gráfico **Saldos atuais por conta** permite reordenar as barras usando a alça em forma de cruzinha (`✣`). O clique normal na barra/linha é reservado para abrir os detalhes; arrastar pela alça é a ação de reordenação.

### 4.2 Tipos de conta disponíveis

| Tipo interno | Nome exibido | Como o painel trata os valores |
|---|---|---|
| `corrente` | Conta Corrente | O saldo cadastrado é a base do saldo atual. Transferências e despesas pagas automaticamente podem alterar esse saldo. Recebimentos e despesas também são apresentados como movimentações. |
| `poupança` | Conta Poupança | O saldo, os depósitos, saques e rendimentos são acompanhados por movimentos da conta. |
| `investimento` | Conta Investimento | O valor mostrado pela conta pode incluir o saldo de caixa cadastrado mais o valor das aplicações vinculadas àquela conta. |
| `previdência` | Conta Previdência | A evolução depende dos aportes, resgates e rendimentos lançados. |
| Dinheiro físico | Dinheiro | É um saldo separado chamado `dinheiroFisico`, não uma conta cadastrada com esse nome. |

O código não inclui contas de investimento e de previdência no indicador **Saldo atual em contas** nem na **Composição do Saldo Líquido**. Elas podem aparecer em outros indicadores de patrimônio e nos detalhamentos de seus tipos.

### 4.3 Formulário Nova Conta

Campos exibidos:

| Campo | Como preencher |
|---|---|
| **Nome da conta *** | Nome único para reconhecer a conta, por exemplo, `Caixa - Poupança`. O nome `Dinheiro` é reservado para o dinheiro físico. |
| **Categoria da conta *** | Escolha Conta Corrente, Poupança, Investimento ou Previdência. |
| **Banco / instituição** | Nome do banco, instituição ou entidade que mantém a conta. |
| **Saldo inicial (R$) *** | Saldo de partida em reais. Pode ser zero; deve ser numérico. |
| **Agência** | Número da agência, se aplicável. |
| **Número da conta** | Número ou identificação da conta. |
| **Data do saldo inicial** | Data usada para registrar o saldo de partida no histórico de poupança e previdência. |
| **Observações** | Texto livre para registrar contexto, origem do saldo ou notas úteis. |

**Como preencher corretamente:** dê um nome fácil de distinguir; selecione o tipo que corresponde ao uso real da conta; informe o saldo de partida sem misturar valores de outra conta; preencha banco, agência e número quando ajudarem a identificação. Para poupança e previdência, o saldo inicial diferente de zero também cria um movimento de saldo inicial com a data informada.

A implementação valida que o nome não esteja vazio, que não seja `Dinheiro`, que não exista outra conta com o mesmo nome (desconsiderando maiúsculas/minúsculas) e que o saldo seja um número válido.

### 4.4 Detalhes de uma conta

A tela de detalhe reúne:

- Dados de identificação, banco, agência, número e observações, se existirem.
- Indicadores compatíveis com o tipo da conta.
- Gráfico de evolução mensal do saldo ou patrimônio.
- Gráfico de entradas versus saídas, ou de aportes versus saques/resgates, conforme o tipo.
- Composição por categoria, capital/rendimento ou aplicações, conforme o tipo.
- Tabela de lançamentos e transferências relacionadas.
- Botão **Editar cadastro da conta**, que abre o formulário com os dados atuais carregados.
- Botão para mover uma conta cadastrada para a Lixeira.

Ao salvar a edição, o formulário atualiza o cadastro existente em vez de criar uma segunda conta. O nome da conta precisa continuar único e não pode ser `Dinheiro`, reservado ao saldo físico. Se o nome for alterado, o código atualiza referências ativas nos movimentos de conta, transferências, origens de despesas, destinos de recebimentos, metas, aplicações vinculadas e contas selecionadas para pagamento de cartão.

As contas corrente e dinheiro mostram o saldo cadastrado, alterado pelas operações que modificam esse saldo. O histórico de entradas e saídas ajuda a consultar os lançamentos, mas não é usado para substituir automaticamente o saldo cadastrado por uma soma de todas as despesas e receitas antigas.

### 4.5 Lançamentos manuais em Poupança e Previdência

O botão **+ Lançamento** aparece para contas poupança e previdência. O formulário contém:

| Campo | Uso |
|---|---|
| **Tipo** | Poupança: Depósito, Rendimento ou Saque. Previdência: Aporte, Rendimento ou Resgate. |
| **Data** | Data em que a movimentação ocorreu. |
| **Valor (R$)** | Valor do movimento. Depósito, aporte, saque e resgate precisam ser maiores que zero. Rendimento aceita valor positivo ou negativo, mas não zero. |
| **Observação** | Descrição complementar do lançamento. |

Depósitos e aportes aumentam o saldo; saques e resgates diminuem. Rendimentos somam quando positivos e reduzem o saldo quando negativos. O sistema bloqueia uma operação que deixaria a conta com saldo negativo, conforme a regra usada no código.

A tabela de lançamentos dessas contas oferece duas ações por registro: **Editar** (`✏️`) e **Mover para a Lixeira** (`🗑️`). Editar abre o formulário preenchido com tipo, data, valor e observação atuais. O painel calcula o efeito do movimento antigo, retira esse efeito do saldo e então aplica o efeito do novo tipo/valor; uma alteração que deixe o saldo abaixo de zero é recusada. O tipo `saldo_inicial`, quando já existe no registro editado, também pode ser carregado no formulário.

Ao mover um movimento para a Lixeira, o efeito do lançamento no saldo é desfeito. Ao restaurá-lo, o efeito é reaplicado.

### 4.6 Transferências entre contas e dinheiro físico

Transferências permitem mover dinheiro entre:

- Dinheiro físico em mãos.
- Qualquer conta cadastrada disponível na lista de contas de transferência.

O valor é subtraído da origem e adicionado ao destino. A transferência **não é considerada receita nem despesa**, pois representa apenas movimento entre saldos próprios.

#### Formulário Nova transferência

| Campo | Como preencher |
|---|---|
| **Origem (debitar)** | De onde o valor sairá. |
| **Destino (creditar)** | Para onde o valor irá. Deve ser diferente da origem. |
| **Valor a transferir (R$)** | Quantia positiva maior que zero. |
| **Data** | Data da transferência. |
| **Descrição / observação** | Texto como “Depósito” ou “Pix entre contas”. |

O sistema valida data, origem, destino, diferença entre origem/destino e valor positivo. Também verifica o saldo disponível na origem. Ao editar uma transferência existente, o código tenta desfazer virtualmente a transferência anterior para validar a nova; se a reversão não for possível com os saldos atuais, a alteração pode ser recusada.

A tabela de transferências permite editar ou mover uma transferência para a lixeira. Na exclusão, o sistema estorna os efeitos nos saldos; a restauração reaplica o débito e o crédito. Para restaurar com segurança, as contas envolvidas precisam estar ativas ou também fazer parte da mesma entrada de lixeira.

---

## 5. Cartões e Parcelamentos

O cartão lateral **Cartões e Parcelamentos** abre um menu com três áreas:

1. **Cartões:** cadastro, limites, faturas e detalhes de cada cartão.
2. **Parcelamentos:** cronogramas completos das compras parceladas, com status e valores do período.
3. **Despesas únicas nesta fatura:** compras à vista no cartão que não pertencem a um parcelamento.

### 5.1 Formulário Novo Cartão

| Campo | Como preencher |
|---|---|
| **Nome do cartão *** | Nome identificador, único entre os cartões, como `Nubank`. |
| **Bandeira *** | Visa, Mastercard, Elo, Amex, Hipercard ou Outra. |
| **Banco emissor** | Banco que emitiu o cartão. |
| **Final do cartão (4 dígitos)** | Somente os quatro últimos dígitos, quando desejar registrar essa identificação. Se informado, deve conter exatamente quatro algarismos. |
| **Limite total (R$) *** | Limite contratado. Deve ser número não negativo. |
| **Conta para pagar a fatura** | Conta de referência que será usada para identificar o pagamento; o formulário registra esse dado no cadastro. |
| **Dia de fechamento** | Dia do mês, de 1 a 31. Opcional. |
| **Dia de vencimento *** | Dia do mês, de 1 a 31. |
| **Observações** | Notas de apoio. |

O sistema impede nomes repetidos, exige nome, limite e vencimento válidos, limita fechamento e vencimento a dias de 1 a 31 e valida o final com quatro dígitos quando preenchido.

A tela de cartões apresenta limite total, limite usado, limite livre, data de vencimento, fechamento e uma barra de disponibilidade. Na tabela, o lápis (`✏️`) abre o formulário de edição e a lixeira (`🗑️`) move o cartão e as despesas vinculadas para a Lixeira. Clicar na linha, fora dos botões, abre os detalhes do cartão; nessa tela também existe **Editar cadastro do cartão**.

Na edição, o formulário carrega os dados atuais: nome, bandeira, banco emissor, final do cartão, limite, conta para pagamento, dias de fechamento e vencimento e observações. O nome não pode duplicar outro cartão. Se o nome do cartão mudar, as despesas ativas cuja origem utiliza o nome antigo recebem a nova referência. A fatura e o limite usado são calculados novamente a partir das despesas ativas; o campo armazenado `faturaAtual` não é a fonte do cálculo mensal da fatura.

### 5.2 Como o limite usado é calculado

O indicador **Limite usado** reúne:

- Despesas pendentes com vencimento no mês selecionado; e
- Parcelas futuras ainda pendentes, posteriores ao mês selecionado, que continuam reservando limite.

Lançamentos com status **Pago** ou **Cancelado** não compõem a reserva de limite pendente. **Limite livre** é calculado como limite total menos limite usado. A fatura do mês e a reserva de parcelas futuras também aparecem separadamente para facilitar a conferência.

A fatura exibida no painel é calculada a partir das despesas do cartão com data no mês. O campo `faturaAtual` que pode existir no objeto do cartão não é usado por essa função de soma da fatura.

### 5.3 Criar Parcelamento

Este formulário é destinado a registrar a compra pelo **valor total**, deixando o painel distribuir esse total em parcelas mensais.

| Campo | Como preencher |
|---|---|
| **Cartão *** | Cartão em que a compra foi feita. É necessário ter ao menos um cartão cadastrado. |
| **Descrição da compra *** | Nome que aparecerá nos lançamentos, como `Passagem aérea`. |
| **Valor total da compra (R$) *** | Valor completo da compra, não o valor de cada parcela. |
| **Nº de parcelas *** | Número inteiro de 2 a 120. |
| **Data da 1ª parcela *** | Data de vencimento da primeira parcela. As seguintes são geradas mensalmente. |
| **Categoria** | Categoria usada para classificar a compra. Se não for escolhida, o código usa `Outros`. |
| **Data da compra** | Data de realização da compra; pode ser diferente do vencimento da primeira parcela. |

O painel mostra uma prévia aproximada de `n` parcelas. Ao salvar, ele converte o total em centavos e distribui os centavos restantes entre as primeiras parcelas, para evitar que o total calculado perca centavos por causa de arredondamento. Cada item guarda título com sua posição, por exemplo `(1/6)`, `(2/6)` e assim por diante.

Se a compra tem apenas uma cobrança, registre-a em **Despesas** ou em **Despesas únicas nesta fatura**. O formulário de parcelamento exige ao menos duas parcelas.

### 5.4 Parcelamentos: lista e cronograma

A lista agrupa despesas parceladas por `parcelamentoId` ou, em registros antigos sem esse identificador, por uma chave compatível construída a partir da origem e do título base.

A lista mostra compra, cartão/origem, número de parcelas, status, valor no período, data da compra e ações. Os títulos das colunas Compra, Parcelas, Valor no período e Data da Compra podem ser usados para ordenar a tabela. Há também uma opção de restaurar a visualização/ordem original.

A lista do módulo filtra os grupos que possuem ao menos uma parcela no período selecionado. Ao abrir o grupo, a tela de cronograma mostra todas as parcelas daquela compra, o total cadastrado, a quantidade contratada, o valor que ainda está pendente e o intervalo de datas. Nesse detalhe é possível:

- Abrir/editar um lançamento individual.
- Alterar o status das parcelas do período.
- Editar o parcelamento completo.
- Mover o parcelamento completo para a lixeira.

A exclusão do parcelamento guarda o conjunto completo em uma única entrada de lixeira. A mensagem de confirmação alerta quando há parcela marcada como paga.

### 5.5 Editar parcelamento completo

O formulário de edição contém:

- **Título do parcelamento**.
- **Cartão**.
- **Categoria**.
- **Valor total da compra (R$)**.
- **Nº de parcelas**.
- **Data da 1ª parcela**.
- **Data da compra**.

A quantidade precisa estar entre 2 e 120, o valor deve ser positivo e a primeira data deve ser informada. Se o número de parcelas, o total ou a primeira data mudar, o sistema recalcula os valores e datas da estrutura. Se a mudança reduzir o número de parcelas, os registros excedentes podem ser removidos; se entre eles houver parcela marcada como paga, o usuário recebe uma confirmação extra. Se a estrutura não mudar, o título, o cartão e a categoria podem ser atualizados em todas as parcelas sem redistribuir os valores.

### 5.6 Despesas únicas nesta fatura

Esta tela reúne despesas cuja origem é um cartão e cujo campo de parcelas é igual a 1 ou não representa um parcelamento. Ela mostra indicadores de quantidade, valor, valor pendente e quantidade de cartões com lançamentos; também exibe gráficos por cartão e categoria.

A tabela permite abrir o detalhe, editar o lançamento, alterar seu status diretamente e movê-lo para a lixeira. Essa tela representa compras de cobrança única no cartão; as compras parceladas são consultadas no módulo Parcelamentos.

---

## 6. Despesas

O módulo Despesas lista as saídas cujo lançamento está datado no mês ou dia selecionado.

### 6.1 Lista de despesas

A tabela contém:

- **Data:** data do lançamento ou vencimento da cobrança, conforme a forma de pagamento.
- **Despesa:** título e categoria.
- **Forma:** origem do pagamento, como dinheiro físico, conta ou cartão.
- **Status:** controle editável diretamente na linha.
- **Valor:** quantia do lançamento; para os itens com `parcelas > 1`, a tabela também indica a quantidade de vezes.
- **Ações:** lápis para editar o lançamento e lixeira para movê-lo para a Lixeira.

A barra de ferramentas tem busca por texto e filtro por categoria. A busca considera o título e a origem. O filtro permite selecionar uma categoria específica. O código da lista mantém ordenação pela data ou pelo valor, ascendente ou descendente, conforme o estado do filtro.

Clicar em uma linha abre os detalhes da despesa. O botão de lápis também abre diretamente o formulário completo de edição. No detalhe, os campos com lápis continuam permitindo alterar valores individualmente; use a edição completa quando precisar alterar mais de um campo ou as opções de recorrência. A alteração de status pelo controle da tabela não abre a tela de detalhe.

### 6.2 Formulário Nova Despesa

| Campo | Como preencher |
|---|---|
| **Título** | Descrição que identifique a compra ou pagamento. |
| **Valor** | Para uma despesa comum ou cobrança única, informe o valor total daquele lançamento. Para a opção mensal no cartão, o campo representa o valor de cada parcela. |
| **Data** | Se a origem for cartão, use a data de vencimento da cobrança/primeira parcela. Para dinheiro ou conta corrente, use a data do pagamento. |
| **Categoria** | Escolha uma categoria cadastrada. |
| **Origem/Pagamento** | Dinheiro físico, conta corrente disponível ou cartão. A lista é montada a partir dos cadastros existentes. |
| **Repetição** | Para uma despesa nova, o formulário permite cobrança única ou mensal no cartão. Registros/séries antigas podem preservar frequências semanais, quinzenais e anuais na edição. |
| **Vezes** | Quantidade de ocorrências em uma série recorrente quando esse campo aparece. O código genérico de recorrências aceita de 1 a 600 ocorrências. |
| **Aplicar alterações** | Ao editar uma série recorrente, escolha alterar somente o lançamento selecionado ou aplicar a alteração à série. |
| **Parcelas no Cartão** | Aparece para uma nova despesa mensal no cartão. Define quantas parcelas mensais serão criadas; o valor do campo Valor é o valor de cada parcela nessa modalidade. |

Nem todos os campos ficam visíveis ao mesmo tempo. A forma de pagamento determina os campos exibidos e a explicação mostrada no formulário.

#### Regras de pagamento implementadas

**Dinheiro físico e conta corrente:** a despesa pode ser debitada automaticamente do saldo. Se a data já chegou ou passou e a despesa entra no fluxo de débito automático, o painel marca o item como **Pago** e debita o valor do saldo. Se a data ainda for futura, registra a despesa como pendente/agendada; quando o HTML for aberto e o sistema processar os débitos agendados, a despesa vencida é debitada. Esse processamento não é um serviço em segundo plano: acontece quando a aplicação é executada.

Se o saldo disponível for inferior à despesa imediata, o sistema pergunta se o usuário deseja continuar e ficar com saldo negativo. É possível recusar a operação.

**Cartão — cobrança única:** a data é tratada como vencimento da cobrança. O lançamento entra na fatura do mês dessa data e não é debitado diretamente do saldo da conta.

**Cartão — mensal:** o sistema gera uma ocorrência por mês, usando o número de parcelas informado. O valor é interpretado como valor de cada parcela, e cada parcela reserva limite conforme as regras do cartão.

**Origem antiga/legada:** registros antigos com uma origem fora das categorias reconhecidas de dinheiro, conta corrente ou cartão podem ser preservados sem débito automático.

### 6.3 Detalhe de uma despesa

A tela de detalhe separa os dados em dois blocos.

**Dados Básicos:** Título, Valor, Data e Categoria.  
**Pagamento:** Origem, Parcelas, Parcela do mês (para despesas parceladas), Status e, quando existente, Repetição.

Os campos marcados com lápis podem ser editados diretamente no quadro. O tipo de controle depende do campo: texto para título, número para valor e parcelas, data para data e lista para categoria/origem. Para valor, é exigido número positivo; para parcelas, número inteiro a partir de 1.

Alterar valor, data ou origem de uma despesa já debitada pode exigir estorno e novo débito automático, para que o saldo acompanhe a mudança. Alterações em metas/recebimentos também têm ajustes próprios descritos nas seções correspondentes.

### 6.4 Detalhe de parcelas acompanha o período selecionado

Quando a despesa aberta pertence a um parcelamento, o painel identifica o grupo e procura a parcela cuja data cai no mês ou dia selecionado. Assim, ao navegar entre os meses, o detalhe deve apresentar a parcela correspondente àquele período, mantendo o título base e a posição correta, como `(2/5)` ou `(3/5)`.

Se não existir parcela do mesmo grupo no período selecionado, a tela informa que não há parcela naquele mês/dia e apresenta uma tabela com o cronograma disponível. Ao clicar em uma parcela do cronograma, a aplicação altera o período para o mês correspondente e atualiza a tela.

O cronograma usa as datas e os identificadores dos registros ativos. Se uma parcela individual for movida para a lixeira, ela deixa de aparecer como registro ativo até ser restaurada.

### 6.5 Exclusão de despesa

O botão de lixeira na lista ou **Mover para a Lixeira** no detalhe remove o registro ativo e guarda o conteúdo na lixeira. Um lançamento individual de parcelamento pode ser removido sem excluir automaticamente todas as outras parcelas. Para remover o grupo inteiro, use a ação de excluir parcelamento na tela de cronograma.

---

## 7. Recebimentos

Recebimentos registra entradas de dinheiro. A lista mostra apenas as entradas dentro do período selecionado, em ordem de data decrescente. As colunas são Data, Origem, Destino, Status, Valor e Ações.

### 7.1 Formulário Novo Recebimento

| Campo | Como preencher |
|---|---|
| **Origem/Cliente** | Nome da pessoa, empresa ou fonte do dinheiro, por exemplo, salário ou serviço prestado. |
| **Valor** | Valor da entrada, maior que zero. |
| **Data** | Data prevista/real da entrada. |
| **Tipo de Entrada** | `Pix` ou `Dinheiro`. |
| **Conta Destino** | Conta em que o valor será recebido quando o tipo for Pix. O campo fica oculto para Dinheiro. |
| **Repetição** | Única, semanal, quinzenal, mensal ou anual. |
| **Quantidade de ocorrências** | Número de vezes a gerar quando a repetição não for única. O código aceita de 1 a 600 ocorrências. |
| **Aplicar alterações** | Ao editar uma série, permite alterar apenas o lançamento ou a série inteira. |

**Como preencher:** descreva de onde vem o valor; registre a quantia positiva; escolha a data; use Pix quando houver uma conta de destino; use Dinheiro quando a entrada for em espécie. Não selecione uma conta bancária quando o tipo for Dinheiro.

O código define a conta associada a recebimentos em dinheiro como `Dinheiro`. A lista apresenta o destino como espécie ou como a conta informada.

Ao salvar uma série recorrente, cada ocorrência recebe uma data calculada conforme a frequência e um título com posição no conjunto, quando há mais de uma ocorrência. Os registros individuais preservam seus identificadores e estados durante a atualização da série quando possível.

### 7.2 Detalhe de recebimento

O detalhe contém os dados básicos — Origem/Cliente, Valor e Data — e os dados de destino — Tipo, Conta e Status. Se o recebimento faz parte de uma série recorrente, aparece a frequência e a quantidade de vezes, além da opção de editar a frequência, a quantidade ou a série.

O status pode ser alterado entre **Pendente**, **Recebido** e **Cancelado**. Use o lápis na lista ou **Editar recebimento** no detalhe para abrir o formulário completo com título/origem, valor, data, tipo de entrada, conta de destino e regras de repetição. Em séries recorrentes, o formulário permite escolher se a alteração se aplica somente ao item selecionado ou à série. A entrada pode ser movida para a Lixeira a partir do detalhe ou da lista. A exclusão de um recebimento gerado automaticamente por uma meta também deve ser tratada considerando a relação com essa meta.

---

## 8. Metas a Receber

Metas a Receber é o módulo de planejamento de entradas futuras. Uma meta representa um valor esperado numa data, não uma entrada confirmada por si só.

### 8.1 Lista e status

A lista mostra Data Prevista, Descrição da Meta, Status, Valor e Ações. As linhas podem ser abertas para consulta; o status pode ser alterado diretamente; o lápis (`✏️`) abre o formulário de edição e a lixeira (`🗑️`) move a meta para a Lixeira. No detalhe, o botão **Editar meta** abre o formulário completo.

Os status usados no código são:

- **Planejado:** meta ainda prevista.
- **Confirmado:** recebimento confirmado; exige um destino para gerar o recebimento correspondente.
- **Cancelado:** meta cancelada.

### 8.2 Formulário Planejar Meta

| Campo | Como preencher |
|---|---|
| **Descrição da Meta** | Descrição clara da entrada esperada, como venda, salário extra ou serviço. |
| **Valor Esperado** | Valor positivo que se espera receber. |
| **Data Prevista** | Data em que o dinheiro é esperado. |
| **Destino do recebimento (quando confirmada)** | Dinheiro físico ou uma conta cadastrada. Pode ficar sem definição enquanto a meta não foi confirmada. |
| **Frequência (Gerar lotes)** | Única, semanal, quinzenal ou mensal. É exibida no cadastro de uma meta nova. |
| **Qtd Meses/Semanas** | Quantidade de ocorrências do planejamento quando a frequência não é única. |

Ao criar uma série, o painel gera uma meta por ocorrência e ajusta a data conforme a frequência. Os títulos podem receber um sufixo de posição, como `(1/3)`, para diferenciar ocorrências.

### 8.3 Confirmar uma meta

Quando o usuário muda o status para Confirmado e a meta ainda não tem destino, o painel abre uma janela para selecionar onde o recebimento será destinado. O usuário pode escolher dinheiro físico ou uma conta.

Com destino definido, o sistema marca a meta como Confirmado e cria ou atualiza um recebimento vinculado à meta. Esse recebimento recebe o mesmo título/descrição, valor e data da meta e fica marcado como recebido. O relacionamento é guardado pelo identificador da meta (`metaId`).

Se a meta deixa de estar confirmada ou perde seu destino, o recebimento que foi gerado automaticamente por ela é removido. Se a meta for atualizada, o recebimento associado é sincronizado com seus dados.

**Consequência prática:** não crie manualmente um segundo recebimento para representar o mesmo valor confirmado da meta; use o mecanismo de confirmação para evitar duplicidade.

### 8.4 Detalhe da meta

Mostra Descrição, Valor Esperado, Data, Status e Destino. Os campos de descrição, valor, data e destino também possuem edição pontual por lápis. O botão **Editar meta** abre o formulário completo com descrição, valor esperado, data, destino e, para uma nova meta, as opções de frequência e quantidade. Ao editar uma meta existente, o formulário oculta as opções de frequência/geração de lote. Se a meta está confirmada sem destino, exibe um aviso para definir para onde irá o recebimento. Se o recebimento correspondente já foi criado, apresenta essa informação.

---

## 9. Categorias

O módulo Categorias permite criar, personalizar, editar e excluir categorias usadas para organizar despesas e alguns lançamentos associados.

### 9.1 Formulário Categoria

| Campo | Como preencher |
|---|---|
| **Nome** | Nome claro e preferencialmente único para a categoria. |
| **Cor** | Cor usada em cartões, barras e legendas dos gráficos. |
| **Ícone (Selecione)** | Emoji escolhido pela biblioteca de ícones disponível na tela. O ícone também é preenchido no campo quando um botão da biblioteca é clicado. |

Para salvar uma categoria, o código exige nome e ícone. Ao editar, ele atualiza nome, cor e ícone do cadastro. Ao criar, gera um novo identificador e adiciona a categoria à lista.

### 9.2 Uso nos demais módulos

As categorias são usadas para classificar despesas, filtrar listas e compor gráficos. A exclusão de uma categoria não exclui automaticamente os lançamentos que a utilizam. Os lançamentos permanecem ativos; até restaurar a categoria ou reclassificar os registros, gráficos e telas podem exibir a categoria sem a cor/ícone personalizados e usar o tratamento padrão.

A exclusão da categoria é enviada para a lixeira, podendo ser restaurada. Isso é diferente de excluir uma despesa: são registros independentes.

### 9.3 Categorias presentes no arquivo anexado

- Alimentação
- Moradia
- Transporte
- Lazer
- Médicos e Farmácia
- Viagens
- Saúde e Bem Estar
- Compras para Casa
- Roupas e Calçados
- Outros
- Telefone / internet

A lista pode mudar conforme o usuário cadastrar, editar ou excluir categorias.

---

## 10. Investimentos

Investimentos representa uma carteira própria, separada do saldo bancário, das despesas e dos recebimentos normais. A seção oferece visão geral, renda fixa, renda variável, grupos por tipo, detalhes de aplicações, histórico mensal, proventos/rendimentos e próximos vencimentos.

### 10.1 Tela principal da carteira

Os principais indicadores são:

- **Patrimônio investido:** soma dos valores atuais/estimados de todas as aplicações ativas.
- **Renda Fixa:** soma das aplicações classificadas como renda fixa.
- **Renda Variável:** soma das aplicações classificadas como renda variável.
- **Rendimentos / proventos registrados:** soma das operações registradas como rendimento ou provento.

Os blocos de distribuição e os indicadores são navegáveis. Os tipos são apresentados em blocos expansíveis; cada bloco reúne aplicações daquele tipo e mostra quantidade e valor agregado. O painel pode apresentar gráficos por conta de investimento, histórico do patrimônio e próximos vencimentos de renda fixa.

**Advertência da própria interface:** taxas, cotações e saldos de mercado são manuais. As projeções são estimativas calculadas com os dados informados e não constituem cotação em tempo real nem garantem retorno.

### 10.2 Classes e tipos embutidos

**Renda Fixa:**

- Poupança
- Tesouro Direto
- CDB
- LCI
- LCA
- Debêntures

**Renda Variável:**

- Ações
- FIIs
- ETFs
- BDRs
- Criptomoedas

Cada tipo tem uma descrição e pode apresentar campos complementares específicos. O cadastro também permite criar tipos personalizados para cada classe.

### 10.3 Formulário Novo Investimento

#### Campos comuns

| Campo | Como preencher |
|---|---|
| **Classe** | Renda Fixa ou Renda Variável. A classe não pode ser alterada quando a aplicação já possui operações registradas, porque isso mudaria o significado dessas operações. |
| **Tipo de investimento** | Escolha um tipo embutido ou personalizado, dentro da classe selecionada. A descrição do tipo aparece no formulário. |
| **Nome da aplicação** | Nome reconhecível, como `CDB Banco X 2028`, `PETR4` ou `BTC`. É obrigatório. |
| **Ticker / identificação** | Código do ativo ou outra identificação, quando aplicável. |
| **Banco / emissor / corretora** | Instituição, emissor ou plataforma. |
| **Data inicial** | Data de início/compra/aplicação. Por padrão, o formulário inicia com a data atual. |
| **Conta de investimento (origem dos aportes)** | Conta do tipo investimento à qual a aplicação será associada, se houver. Pode ficar sem conta definida. |
| **Observações** | Notas livres sobre a aplicação. |

#### Campos de renda fixa

| Campo | Como preencher |
|---|---|
| **Indexador** | Prefixado, CDI, Selic, IPCA+, TR+ ou Outro. |
| **Vencimento** | Data de vencimento, quando existe. Não pode ser anterior à data inicial. |
| **Taxa anual contratada (% a.a.; prefixados)** | Taxa anual informada para aplicações prefixadas. Exemplo exibido no formulário: `12`. |
| **Percentual do indexador (%)** | Percentual contratado do indexador, por exemplo, `110` para 110% do CDI. |
| **Taxa base anual hipotética (%)** | Taxa anual usada como hipótese para CDI, Selic, IPCA ou TR. |
| **Spread anual (%)** | Taxa adicional anual hipotética sobre o indexador, quando aplicável. |
| **Tributação estimada** | IR regressivo, Isento ou A verificar. |
| **Saldo atual real informado (R$; opcional)** | Quando preenchido, o valor informado é usado como valor atual da aplicação em vez da projeção calculada para hoje. Em branco, o painel utiliza a projeção baseada nos movimentos e na taxa cadastrada. |

#### Campos de renda variável

| Campo | Como preencher |
|---|---|
| **Cotação atual unitária (R$; manual)** | Preço unitário atual informado manualmente. |
| **Data da cotação** | Data a que se refere a cotação informada. |

A quantidade e o custo médio da renda variável vêm das operações de compra e venda registradas no histórico. A cotação não é consultada automaticamente pelo painel. Na tabela de aplicações, o lápis (`✏️`) abre o cadastro existente para edição e a lixeira move a aplicação e seu histórico para a Lixeira. O detalhe da aplicação também oferece **Editar aplicação**. Se a aplicação já tem operações, sua classe não pode ser alterada, pois isso mudaria a interpretação do histórico.

#### Campos adicionais por tipo

O tipo escolhido determina outros campos dinâmicos. Os campos predefinidos no código são:

| Campo técnico | Rótulo exibido |
|---|---|
| `banco` | Banco / emissor |
| `aniversario` | Dia de aniversário |
| `titulo` | Subtipo / título (ex.: Tesouro IPCA+ 2035) |
| `precoUnitario` | Preço unitário na compra (R$) |
| `quantidadeTitulos` | Quantidade de títulos (referência) |
| `custodia` | Custódia / corretora |
| `liquidez` | Liquidez / resgate |
| `carencia` | Carência até |
| `rating` | Classificação de risco |
| `garantias` | Garantias |
| `amortizacao` | Amortizações |
| `cupom` | Pagamento de cupons |
| `setor` | Setor |
| `pl` | P/L |
| `pvp` | P/VP |
| `roe` | ROE (%) |
| `dy` | Dividend yield (%) |
| `segmento` | Segmento / tipo de FII |
| `vacancia` | Vacância (%) |
| `indice` | Índice de referência |
| `gestora` | Gestora |
| `taxaAdm` | Taxa de administração (% a.a.) |
| `ativoExterior` | Ativo estrangeiro representado |
| `mercadoOrigem` | Mercado de origem |
| `moeda` | Moeda de referência |
| `rede` | Rede blockchain |
| `plataforma` | Exchange / plataforma |
| `carteira` | Carteira / custódia |
| `staking` | Staking / rendimentos |

Os campos aparecem de acordo com o tipo selecionado; não é necessário preencher todos os campos de todas as classes.

### 10.4 Cadastrar tipo personalizado de investimento

O formulário **Cadastrar tipo de investimento** possui:

- **Classe:** Renda Fixa ou Renda Variável.
- **Nome do tipo:** nome pelo qual o novo tipo será identificado.
- **Descrição / como funciona:** explicação exibida ao consultar o tipo.
- **Campos adicionais:** nomes de campos específicos, separados por vírgulas, ponto e vírgula ou quebras de linha. O código guarda até 15 campos por tipo.

Nome e descrição são obrigatórios. O painel não permite repetir um tipo com o mesmo nome dentro da mesma classe, desconsiderando maiúsculas/minúsculas. Para tipos personalizados, a tela de tipo possui também a ação **Editar tipo**. Ela reabre os campos existentes, e permite atualizar classe, nome, descrição e lista de campos adicionais. Ao salvar uma alteração de nome/classe, as aplicações ligadas ao nome e à classe anteriores são atualizadas para a nova referência. Os tipos embutidos são definidos pelo código; o controle de edição é apresentado para tipos personalizados.

### 10.5 Registrar operação em uma aplicação

O formulário de lançamento contém:

| Campo | Uso |
|---|---|
| **Operação** | Depende da classe da aplicação. |
| **Data** | Data da operação. |
| **Quantidade de unidades / cotas** | Usada em compras e vendas de renda variável. |
| **Preço unitário pago / vendido (R$)** | Usado em compras e vendas de renda variável. |
| **Valor total (R$)** | Valor da operação; para compra/venda de renda variável é calculado como quantidade × preço unitário. |
| **Custos / taxas da operação (R$)** | Custos adicionais, exibidos para compras e vendas de renda variável. |
| **Observação** | Texto livre para contextualizar a operação. |

Tipos de operação aceitos:

- **Renda fixa:** Aporte, Resgate e Juros / rendimento recebido.
- **Renda variável:** Compra, Venda e Provento / dividendo recebido.

Para uma compra ou venda de renda variável, quantidade e preço unitário devem ser positivos; o valor total é recalculado pelo produto entre esses campos e fica somente para leitura. Taxas não podem ser negativas. O sistema impede vender quantidade maior que a quantidade disponível calculada para o ativo. Em renda fixa, o valor do resgate não pode superar o saldo de principal calculado para a data da operação.

Editar uma operação substitui o registro daquele lançamento. Excluir uma operação move apenas o movimento para a lixeira, não a aplicação inteira. Excluir a aplicação guarda junto dela todo o histórico de operações.

### 10.6 Como os cálculos de investimentos funcionam

#### Renda fixa

O painel estima o valor com base nos aportes e resgates registrados e na taxa efetiva construída a partir dos campos informados:

- **CDI:** taxa base × percentual do indexador + spread.
- **IPCA+ e TR+:** composição multiplicativa entre taxa base e spread.
- **Selic:** taxa base + spread.
- **Prefixado/Outros:** utiliza a taxa anual contratada cadastrada.

A estimativa aplica capitalização diária equivalente à taxa anual informada, calculada a partir do intervalo entre a data de cada movimento e a data-alvo. O cálculo limita a taxa efetiva para não permitir taxa inferior a -99% ao ano na função que projeta os fluxos. Um vencimento anterior à data-alvo faz a data-alvo ser limitada ao vencimento.

Se **Saldo atual real informado** estiver preenchido, ele prevalece como valor atual. Caso esteja vazio, o painel usa a estimativa resultante dos movimentos. A projeção até o vencimento mostra principal projetado, rendimento bruto estimado, montante bruto e valor líquido estimado quando há uma tributação reconhecida pelo código.

A tributação é estimada de forma simplificada. Para `Isento`, a taxa de IR projetada é zero. Para `IR regressivo`, o código usa faixas aproximadas baseadas nos dias desde o primeiro aporte encontrado: até 180 dias, 22,5%; até 360 dias, 20%; até 720 dias, 17,5%; acima disso, 15%. Para `A verificar`, o líquido não é calculado. A tela alerta que regras tributárias reais podem variar por lote e pelas condições do produto.

#### Renda variável

O painel percorre compras e vendas até a data atual para calcular quantidade em carteira, custo acumulado e preço médio. O custo de compra inclui as taxas registradas. Nas vendas, o sistema calcula o resultado realizado comparando o valor líquido da venda (descontadas taxas) com o custo médio das unidades vendidas.

- Se houver cotação atual maior que zero, o valor de mercado estimado é quantidade atual × cotação manual.
- Se não houver cotação, o valor mostrado é estimado usando o preço médio/custo.
- Proventos e rendimentos registrados são somados separadamente.

Esses números dependem da qualidade dos lançamentos e das cotações digitadas. O sistema não consulta preços de mercado automaticamente.

### 10.7 Histórico, rendimentos e vencimentos

- **Evolução do patrimônio investido:** permite escolher últimos 3, 6 ou 12 meses ou um intervalo personalizado. Os pontos podem ser clicados para abrir as movimentações do mês.
- **Movimentações de um mês:** lista aportes, compras, vendas, resgates, rendimentos e proventos registrados naquele mês.
- **Proventos e rendimentos registrados:** lista operações do tipo provento ou rendimento, com data, ativo, tipo e valor.
- **Próximos vencimentos de renda fixa:** mostra até 15 aplicações de renda fixa com vencimento futuro registrado, junto de uma projeção bruta estimada.

A série histórica de investimentos é calculada a partir dos registros de movimentos e projeções do painel. A interface a descreve como série ilustrativa, e não como cotação histórica oficial.

### 10.8 Investimentos vinculados a contas

Uma aplicação pode ser vinculada a uma conta do tipo investimento. Essa associação serve para agrupar aplicações por conta e apresentar o patrimônio correspondente nos detalhes dessa conta.

**O vínculo não faz o débito automaticamente na conta bancária.** Ao registrar um aporte ou uma compra no investimento, o painel atualiza as métricas da carteira, mas não subtrai automaticamente o dinheiro da conta vinculada. Para movimentar dinheiro real entre saldos de conta e dinheiro físico, use o módulo Transferências.

---

## 11. Lixeira

A Lixeira é um módulo central para recuperar itens excluídos.

### 11.1 O que uma entrada da lixeira contém

Cada entrada guarda um título, uma descrição, a data da exclusão, um ou mais registros removidos e dados auxiliares quando necessários para reconstituir a operação. Itens relacionados podem ser agrupados na mesma entrada, por exemplo, uma conta com seus movimentos e lançamentos relacionados, um cartão com suas despesas ou um parcelamento completo.

### 11.2 Ações disponíveis

- **Restaurar:** coloca os registros de volta nas coleções ativas e atualiza os indicadores.
- **Excluir definitivamente:** remove a entrada da lixeira depois de confirmação. Não há restauração por esse módulo após a exclusão definitiva.
- **Esvaziar lixeira:** remove definitivamente todas as entradas, depois de confirmação.

O contador do cartão Lixeira mostra a quantidade de entradas guardadas, não necessariamente a quantidade de registros internos. Uma entrada pode conter vários registros vinculados.

### 11.3 Dependências e efeitos nos saldos

O sistema tenta restaurar os cadastros-pai antes de movimentos internos, por exemplo, restaurar uma aplicação antes de restaurar uma operação guardada daquela aplicação. Se o item depende de outra conta ou aplicação que continua excluída, o painel pode solicitar que a dependência seja restaurada primeiro.

Quando a exclusão afeta um saldo — por exemplo, uma transferência, um débito automático de despesa ou um movimento de poupança/previdência — o painel guarda metadados que permitem desfazer o efeito ao excluir e reaplicá-lo ao restaurar. A restauração também recalcula os totais e as métricas.

### 11.4 Como usar com segurança

1. Abra a Lixeira pelo cartão lateral.
2. Leia o título, o detalhe e a data de exclusão.
3. Para desfazer uma exclusão, clique em **Restaurar**.
4. Para remover definitivamente um item específico, clique em **Excluir definitivamente** e confirme.
5. Para remover todos os itens, use **Esvaziar lixeira** e confirme.

A exclusão definitiva e o esvaziamento são irreversíveis dentro do painel. Antes de usá-los, confira se não existe uma conta, aplicação ou lançamento vinculado que possa ser necessário restaurar.

### 11.5 O que pode ser movido para a lixeira

A implementação contém rotinas de exclusão para contas, cartões, despesas, recebimentos, metas, categorias, transferências, movimentos manuais de contas, aplicações, operações de investimentos, tipos personalizados de investimento e grupos completos de parcelamento. Dependendo do tipo, registros dependentes são armazenados em conjunto ou em entradas separadas para restauração individual.

---

## 12. Composição do Saldo Líquido e evolução histórica

Abaixo do painel principal existem dois elementos que permanecem disponíveis em todas as áreas: o cartão de composição do saldo e o painel de evolução financeira histórica.

### 12.1 Composição do Saldo Líquido

O cartão apresenta:

- **Dinheiro físico em mãos:** valor de `dinheiroFisico`.
- **Total em Contas:** soma dos saldos de contas correntes e poupanças considerados disponíveis pelo código.
- **Faturas Pendentes:** compras à vista no cartão que continuam pendentes no mês selecionado.
- **Parcelas Pendentes:** parcelas de compras parceladas pendentes no mês selecionado.
- **Total líquido:** dinheiro físico + saldos de contas considerados disponíveis − faturas pendentes − parcelas pendentes.

Lançamentos marcados como **Pago** ou **Cancelado** deixam de compor os totais de faturas e parcelas pendentes. O cartão é clicável e abre o detalhamento de composição do saldo.

O código exclui do cálculo deste saldo líquido as contas do tipo investimento e previdência. O patrimônio investido é apresentado separadamente.

### 12.2 Evolução Financeira Histórica

O gráfico pode mostrar duas métricas:

- **Saldo Projetado:** a legenda do gráfico usa “Progressão do Saldo”. O código soma progressivamente, dentro do intervalo escolhido, receitas menos despesas de cada mês. O cálculo da linha começa no zero do período exibido; não é a soma direta dos saldos bancários cadastrados.
- **Entradas x Saídas:** compara a soma de recebimentos e despesas por mês.

Períodos disponíveis: últimos 3 meses, últimos 6 meses, últimos 12 meses ou personalizado. No personalizado, informe data inicial e data final. A data inicial precisa ser menor ou igual à data final.

Os pontos do gráfico são clicáveis e abrem os lançamentos do mês selecionado. Se o intervalo personalizado estiver inválido, o gráfico exibe uma mensagem para corrigir as datas.

---

## 13. Formulários: campos e preenchimento correto

Esta seção reúne as principais validações e cuidados de entrada de dados em um único lugar.

### 13.1 Regras gerais de preenchimento

- **Valores monetários:** use números. Nas telas de dinheiro, o painel formata os valores como Real brasileiro (R$). Valores de despesas, recebimentos, metas, compras, aportes e resgates precisam ser positivos quando o código assim exige.
- **Datas:** os campos internos usam o formato ISO `AAAA-MM-DD`, mas a interface apresenta datas em formato brasileiro. Use a data real do evento; em cartão, diferencie vencimento da cobrança e data em que a compra foi feita.
- **Títulos e nomes:** prefira descrições reconhecíveis. Para contas e cartões, evite nomes duplicados.
- **Categorias:** escolha uma categoria já cadastrada ou crie uma antes de lançar a operação. Categorias organizam os relatórios, mas não alteram o valor financeiro do registro.
- **Parcelamentos:** não confunda valor total da compra com valor de cada parcela. No formulário Criar Parcelamento, informa-se o total; na modalidade de despesa mensal no cartão, o campo Valor representa cada parcela.
- **Séries recorrentes:** confirme a frequência e quantidade antes de salvar, porque o sistema gera registros independentes por ocorrência.
- **Destino de recebimentos:** Pix usa uma conta; dinheiro usa o saldo de dinheiro físico. Uma meta só gera recebimento automaticamente quando é confirmada e tem destino.
- **Investimentos:** cotações, taxas e valores atuais são dados manuais. Confira datas e unidades antes de registrar operações.

### 13.2 Resumo das validações implementadas

| Cadastro/formulário | Validações relevantes |
|---|---|
| Conta | Nome obrigatório e único; `Dinheiro` reservado; saldo inicial numérico válido. |
| Cartão | Nome obrigatório e único; limite preenchido e não negativo; vencimento inteiro de 1 a 31; fechamento opcional entre 1 e 31; final com quatro dígitos se preenchido. |
| Despesa | Título, valor positivo, data e origem obrigatórios. Em pagamento imediato por dinheiro/conta corrente, pode alertar para saldo insuficiente. |
| Recebimento | Título, valor positivo e data obrigatórios; se for Pix, exige conta de destino. |
| Meta | Descrição, valor positivo e data obrigatórios. Confirmação exige destino. |
| Categoria | Nome e ícone obrigatórios. |
| Parcelamento de cartão | Cartão existente; título; total positivo; número inteiro de 2 a 120; data da primeira parcela. |
| Edição de parcelamento | Título; total positivo; número de 2 a 120; data inicial; confirmação extra se forem removidas parcelas pagas. |
| Investimento | Nome da aplicação; tipo válido dentro da classe; vencimento não pode ser anterior à data inicial. A classe não pode mudar depois que existem operações. |
| Operação de investimento | Data e valor válidos; compra/venda em renda variável requer quantidade e preço positivos; taxas não negativas; venda limitada às unidades disponíveis; resgate limitado ao principal calculado. |
| Transferência | Origem e destino válidos e diferentes; data; valor positivo; saldo suficiente e possibilidade de reverter operações anteriores, quando estiver editando. |
| Movimento poupança/previdência | Data obrigatória; valor positivo para depósito/aporte/saque/resgate; rendimento diferente de zero, inclusive negativo; saldo não pode ficar abaixo de zero pela movimentação. |

Nem todos os campos estão marcados com o atributo HTML `required`; as validações principais são feitas pelas funções de JavaScript ao salvar. Uma mensagem de erro ou um aviso impede a operação quando a validação falha.

### 13.3 Edição de cadastros e lançamentos

A versão documentada oferece edição por três padrões de interface. **Lápis ao lado de uma linha** abre o formulário ou controle de edição daquele registro; **lápis dentro do detalhe** edita apenas o campo indicado; **Editar cadastro/aplicação/meta/parcelamento** abre uma edição mais ampla, quando ela existe para aquele módulo. Ao salvar, o painel atualiza os dados ativos e recalcula os indicadores que dependem deles. O desenho exato da edição varia por tipo de registro: não existe um único formulário universal para todos os módulos.

| O que editar | Onde encontrar | Campos e escopo efetivamente implementados | Observações importantes |
|---|---|---|---|
| **Conta cadastrada** | `Contas cadastradas` e detalhe da conta | Nome, categoria/tipo, banco/instituição, saldo cadastrado, agência, número da conta e observações. | O nome precisa ser único; `Dinheiro` é reservado ao saldo físico. Ao renomear, o código atualiza referências ativas em coleções relacionadas. A data do saldo inicial participa do cadastro de movimentos iniciais novos; na edição de uma conta existente, o código edita o saldo do cadastro e não recria automaticamente o movimento inicial. |
| **Cartão** | Tabela de cartões e detalhe do cartão | Nome, bandeira, banco emissor, final de quatro dígitos, limite, conta de pagamento, dia de fechamento, dia de vencimento e observações. | Nome não pode duplicar outro cartão. Ao renomear, o código ajusta as despesas ativas que usam `cartao_NomeAntigo` como origem. A fatura mensal é recalculada pelas despesas ativas. |
| **Movimento de conta poupança/previdência** | Detalhe de uma conta elegível, na tabela `Lançamentos da conta` | Tipo, data, valor e observação. | O formulário vem preenchido. Ao salvar, o painel retira o efeito do movimento antigo do saldo e aplica o novo. Depósito/aporte somam; saque/resgate subtraem; rendimento aceita valor positivo ou negativo, mas não zero. O saldo não pode ficar negativo devido à alteração. |
| **Despesa** | Lista de despesas, detalhe da despesa e tabelas relacionadas | Edição pontual de título, valor, data, categoria, origem, quantidade de parcelas e status; formulário completo para título, valor, data, categoria, origem/pagamento e regras de repetição/parcelas conforme os campos visíveis. | Em despesa parcelada, o formulário também pode direcionar à edição do parcelamento inteiro. Alterações em registros já debitados podem exigir estorno e novo débito automático. Confira se quer editar uma parcela, o lançamento selecionado ou o conjunto completo. |
| **Recebimento** | Lista e detalhe de recebimento | Título/origem, valor, data, tipo de entrada (Pix/Dinheiro), conta e opções de recorrência. Também há edição pontual de campos no detalhe. | Para séries recorrentes, o formulário pode aplicar a mudança só ao item selecionado ou à série. Para Pix, é necessária uma conta de destino; para Dinheiro, o destino é o saldo físico. |
| **Meta a receber** | Lista e detalhe da meta | Descrição, valor esperado, data prevista e destino. O detalhe tem lápis para campos individuais e botão **Editar meta** para abrir o formulário. | A edição de meta existente não mostra as opções de criar lote/frequência. Se a meta estiver confirmada e seus dados mudarem, o recebimento vinculado pode ser sincronizado pelas rotinas de meta. |
| **Categoria** | Módulo Categorias | Nome, cor e ícone. | A categoria é referenciada pelo nome em outros lançamentos. Excluir uma categoria não exclui automaticamente lançamentos classificados com ela. |
| **Transferência** | Histórico de transferências | Origem, destino, valor, data e descrição/observação. | A validação tenta reverter a transferência anterior antes de aplicar a nova. Se não for possível recuperar o saldo anterior de modo consistente, a alteração pode ser recusada. Transferência não é receita nem despesa. |
| **Parcelamento de cartão** | Lista de parcelamentos e cronograma do grupo | Título, cartão/origem, categoria, valor total, número de parcelas, data da primeira parcela e data da compra. | Se quantidade, total ou data inicial mudar, o cronograma é recalculado. O sistema pede confirmação adicional se a alteração remover parcelas marcadas como pagas. Alterar apenas o título renomeia as parcelas do grupo. |
| **Aplicação de investimento** | Tabela da carteira e detalhe da aplicação | Campos gerais da aplicação e os campos específicos da classe/tipo, como código, instituição, datas, taxa/indexador, saldo atual informado, cotação manual, conta vinculada, observações e campos dinâmicos do tipo. | Se há operações registradas, a classe não pode ser alterada. Os aportes/operações da carteira não debitam automaticamente contas bancárias. |
| **Operação dentro de uma aplicação** | Detalhe da aplicação, tabela `Histórico de aportes, operações e proventos` | Operação, data, quantidade/unidades, preço unitário, valor total, custos/taxas e observação, conforme o tipo de operação. | O lápis reabre os dados da operação existente e salva no mesmo identificador em vez de criar outro movimento. Venda não pode ultrapassar a quantidade disponível; resgate não pode exceder o principal restante calculado. |
| **Tipo personalizado de investimento** | Tela do tipo de investimento; botão **Editar tipo** | Classe, nome, descrição e lista de campos adicionais (máximo de 15 campos guardados). | Nome e descrição são obrigatórios; nomes duplicados dentro da mesma classe são bloqueados. Ao atualizar o nome/classe, o código ajusta as aplicações que correspondem ao tipo antigo e à classe antiga. |

#### Procedimento recomendado para editar

1. **Localize o registro:** abra o módulo e use busca/filtros ou clique na linha para entrar no detalhe.
2. **Escolha o lápis correto:** use o lápis da linha para abrir a edição do registro; use o lápis ao lado de um campo quando quiser alterar somente aquele dado; use **Editar parcelamento** quando a mudança precisar atingir o cronograma inteiro.
3. **Revise os campos carregados:** o formulário deve abrir preenchido com os dados atuais. Altere somente os campos necessários e confira a data, o valor, a origem/destino e a categoria.
4. **Salve e leia a confirmação:** se uma validação falhar, o painel mostra uma mensagem e não conclui a operação. Quando concluir, os indicadores e a visualização são atualizados.
5. **Confira os vínculos e os saldos:** ao editar transferências, movimentos de poupança/previdência, despesas com débito automático, nomes de conta/cartão ou metas confirmadas, verifique se os lançamentos relacionados continuam corretos.
6. **Salve o arquivo HTML:** a edição altera o estado em memória. Para preservar a versão no disco, use **Salvar Arquivo**.

#### Limites que a documentação não deve esconder

- Um lápis no detalhe pode alterar apenas aquele campo; ele não garante que todo o registro será editável por aquele controle.
- Algumas edições de grupos têm efeitos sobre várias linhas (parcelamentos e séries recorrentes); outras alteram só o registro individual.
- A edição da conta atualiza o saldo cadastrado. Para poupança/previdência, não presuma que mudar esse campo também reescreva todos os movimentos históricos já registrados: a rotina de edição do cadastro não cria novamente o movimento inicial.
- Renomear contas e cartões atualiza as referências ativas previstas nas rotinas de salvamento; antes de renomear, confira os itens vinculados e, depois, revise o detalhe e as listas correspondentes.
- A aplicação é local. Se o usuário fechar sem salvar o HTML atualizado, as alterações da sessão não ficam gravadas no arquivo original.

---


---

## 14. Regras de cálculo, status e consistência

### 14.1 Status das despesas

Os status armazenados para despesas são **Pendente**, **Pago** e **Cancelado**. A interface pode apresentar rótulos calculados a partir da data:

- **Pago/Concluído:** quando o status está pago.
- **Atrasado:** lançamento pendente com data anterior à data atual.
- **É Hoje:** lançamento pendente com data igual à data atual.
- **Em N d:** lançamento pendente que vence dentro dos próximos cinco dias.
- **Futuro:** lançamento pendente com data posterior a esse intervalo.
- **Cancelado:** lançamento cancelado.

Os textos “Atrasado”, “É Hoje”, “Em N d” e “Futuro” representam a situação temporal de um lançamento pendente. O código não os trata como valores de status separados de `Pendente`.

### 14.2 Status dos recebimentos

Os status armazenados são **Pendente**, **Recebido** e **Cancelado**. Se não houver status explícito num registro antigo ou novo, a interface considera **Recebido** como estado padrão para recebimentos.

### 14.3 Status das metas

Os status usados são **Planejado**, **Confirmado** e **Cancelado**. Confirmar uma meta com destino cria ou atualiza um recebimento associado. Tirar a confirmação ou deixar a meta sem destino remove o recebimento gerado por ela.

### 14.4 Status de grupos de parcelas

O seletor de status do grupo opera nas parcelas do grupo que estão presentes no período selecionado. Se as parcelas desse período têm estados diferentes, a interface apresenta `Misto`. Selecionar Pendente, Pago ou Cancelado aplica o status escolhido às parcelas daquele período, e não necessariamente a todos os meses do cronograma.

### 14.5 Cálculo de valores no período

A maioria das listas e KPIs de receitas, despesas e metas soma registros cuja data está dentro do mês ou dia selecionado. O período de uma fatura depende das datas guardadas em cada parcela/despesa; a data da compra não substitui a data de vencimento usada para filtrar a fatura.

O **Saldo do Período** na Visão Geral é receitas menos despesas. Metas aparecem em seu próprio indicador, pois são previsões e só se tornam recebimentos quando confirmadas conforme a regra do módulo.

### 14.6 Saldo líquido e limite do cartão não são a mesma coisa

- **Saldo líquido:** caixa físico + contas correntes/poupanças consideradas disponíveis − faturas e parcelas pendentes no mês selecionado.
- **Limite usado do cartão:** despesas pendentes no mês selecionado + parcelas pendentes de meses futuros.
- **Limite livre:** limite cadastrado − limite usado.

A fatura mensal considera os lançamentos com origem no cartão e data naquele mês. Para calcular quanto ainda está comprometido, o painel separa itens pagos/cancelados das despesas pendentes e considera parcelas futuras pendentes.

### 14.7 Contas correntes, dinheiro, poupança e previdência têm regras diferentes

- Nas contas correntes e em dinheiro, o saldo cadastrado é a base atual. Transferências e despesas pagas pelo fluxo de débito automático podem alterá-lo.
- Poupança e previdência usam movimentos datados para exibir evolução, capital e rendimento. O cadastro inicial de saldo dessas contas pode gerar movimento inicial na data informada.
- Contas de investimento podem agregar aplicações vinculadas ao valor da conta, mas os aportes de investimento não debitam automaticamente o saldo da conta.
- O dinheiro físico é mantido em `dinheiroFisico` e é selecionado como origem/destino em transferências.

### 14.8 Exclusões, vínculos e recálculo

A exclusão de um item não significa sempre excluir somente uma linha. Dependendo do tipo de registro:

- Excluir uma conta guarda também movimentos de saldo, transferências e despesas/recebimentos/metas vinculados à conta, e tenta reverter os efeitos de transferências nos saldos.
- Excluir um cartão guarda o cartão e as despesas com aquela origem.
- Excluir um parcelamento guarda todas as parcelas como uma entrada de lixeira.
- Excluir uma aplicação guarda o histórico de operações daquela aplicação.
- Excluir uma operação de investimento guarda apenas aquele movimento.
- Excluir um tipo personalizado pode guardar também as aplicações associadas a ele.
- Excluir uma categoria não remove as despesas que usam seu nome.

Após excluir ou restaurar, o código atualiza a interface e recalcula indicadores a partir dos dados que continuam ativos. Em certos vínculos, a restauração deve respeitar dependências: por exemplo, restaurar a aplicação antes de uma operação isolada quando a aplicação também foi excluída.

---

## 15. Salvar, alterações pendentes e recuperação dos dados

### 15.1 O que acontece ao alterar dados

Ao criar, editar, excluir, restaurar, mudar o tema ou reordenar itens, o painel sinaliza que há alterações pendentes. A rotina chamada `salvarDadosLocal()` apenas marca a alteração; ela não grava cada mudança automaticamente em armazenamento do navegador.

### 15.2 Como salvar corretamente

1. Clique em **Salvar Arquivo**.
2. Em navegadores que oferecem o seletor de arquivo, escolha o arquivo de destino para salvar/sobrescrever o HTML.
3. Se o seletor não estiver disponível ou ocorrer o caminho alternativo previsto no código, o painel gera um download do HTML atualizado.
4. Depois do salvamento concluído, o indicador de alterações pendentes é limpo.
5. Abra a versão salva quando quiser continuar trabalhando com o estado persistido mais recente.

O botão cria o HTML com os dados atuais serializados dentro do script `dados_injetados`. Portanto, o arquivo baixado/salvo contém a nova versão dos dados e da interface.

### 15.3 Aviso ao sair

Se houver alterações pendentes, o código registra um evento `beforeunload` para avisar que há alterações não salvas. Esse aviso não substitui o salvamento: para preservar as mudanças, use o botão Salvar Arquivo e confirme que o arquivo foi salvo ou baixado.

### 15.4 Zerar o painel

O botão **Zerar painel** solicita duas confirmações. O código remove contas, cartões, despesas, recebimentos, metas, investimentos, transferências, movimentos de contas, tipos personalizados e lixeira, zera o dinheiro físico e mantém as categorias. O estado zerado só se torna parte permanente do arquivo quando o HTML atualizado é salvo. Se a sessão for fechada sem salvar, o arquivo anterior permanece intacto.

### 15.5 Recomendações operacionais

- Salve o HTML após uma sessão relevante de alterações.
- Trabalhe sempre a partir da cópia que contém os dados mais recentes.
- Antes de zerar o painel, excluir definitivamente ou esvaziar a lixeira, confirme que não precisará dos registros.
- Antes de editar um parcelamento completo, revise o número de parcelas, o valor total e a data da primeira parcela.
- Ao confirmar uma meta, confira cuidadosamente o destino para que o recebimento seja criado no lugar correto.

---

## 16. Estado dos dados existentes no arquivo anexado

Esta seção descreve o conteúdo efetivamente incorporado no HTML anexado no momento da documentação. É um retrato daquele arquivo, não um limite do que o painel pode armazenar. Os valores podem mudar conforme novos lançamentos forem feitos e o arquivo for salvo.

### 16.1 Contas existentes

| Conta | Tipo | Saldo inicial/atual guardado | Instituição | Observações do cadastro |
|---|---|---:|---|---|
| Nubank - Débito em Conta | Corrente | R$ 1.095,31 | Nubank | O cadastro informa que o saldo foi iniciado em 10/10/2026 depois do recebimento do salário, do pagamento de fatura de R$ 1.641,00 e de outras despesas já abatidas. |
| Nubank Poupança / Caixinhas | Poupança | R$ 0,00 | Nubank | Agência 0001; número `N/A`. |
| Caixa Previdência | Previdência | R$ 4.020,00 | Caixa Econômica Federal | Agência 3115; conta 767252206-0; saldo inicial considerado a partir de 10/10/2026. |
| Caixa - Poupança | Poupança | R$ 617,50 | Caixa Econômica Federal | Agência 3115; conta 767252206-0; saldo inicial considerado a partir de 10/10/2026. |

A soma dos saldos guardados nesses quatro cadastros é R$ 5.732,81. O indicador de saldo disponível do painel usa somente contas correntes e poupanças, que somam R$ 1.712,81 no estado incorporado. A previdência, de R$ 4.020,00, não entra na composição do saldo líquido segundo a regra atual.

O arquivo contém dois movimentos de saldo inicial: Caixa Previdência, de R$ 4.020,00 em 10/10/2026; e Caixa - Poupança, de R$ 617,50 em 10/10/2026.

### 16.2 Cartão existente

| Campo | Valor incorporado |
|---|---|
| Nome | Nubank |
| Bandeira | Mastercard |
| Banco emissor | Nubank |
| Final do cartão | 8040 |
| Limite | R$ 11.000,00 |
| Fechamento | Dia 5 |
| Vencimento | Dia 12 |

O objeto do cartão tem `faturaAtual` igual a zero. A fatura apresentada pela aplicação é calculada a partir das despesas ativas associadas ao cartão e ao mês selecionado.

### 16.3 Categorias existentes

O arquivo contém 11 categorias: Alimentação; Moradia; Transporte; Lazer; Médicos e Farmácia; Viagens; Saúde e Bem Estar; Compras para Casa; Roupas e Calçados; Outros; Telefone / internet.

### 16.4 Despesas e parcelamentos existentes

O arquivo contém **38 registros ativos de despesas**, distribuídos entre outubro de 2026 e setembro de 2027. A soma bruta dos 38 registros em todos os meses é **R$ 4.143,12**; esse valor não é uma fatura mensal, pois agrega parcelas de vários meses e inclui lançamentos já marcados como pagos.

| Compra/lançamento | Estrutura encontrada | Datas registradas | Valor por parcela/lançamento | Total dos registros | Categoria e status relevantes |
|---|---|---|---:|---:|---|
| Peças suspensão Crossfox | 5 parcelas | 12/10/2026 a 12/02/2027 | R$ 248,30 | R$ 1.241,50 | Transporte; primeira parcela paga e as demais pendentes. |
| Plano Fidelidade - Azul Linhas Aéreas | 12 parcelas | 12/10/2026 a 12/09/2027 | R$ 42,00 | R$ 504,00 | Viagens; primeira parcela paga e as demais pendentes. |
| Mercado Livre - kimono (casaco e calça) — série A | 6 parcelas | 12/11/2026 a 12/04/2027 | R$ 55,91 | R$ 335,46 | A primeira parcela está em Roupas e Calçados; as cinco seguintes estão em Outros. Todas pendentes. |
| Mercado Livre - kimono (casaco e calça) — série B | 6 parcelas | 12/11/2026 a 12/04/2027 | R$ 55,91 | R$ 335,46 | Roupas e Calçados; todas pendentes. É um parcelamento separado da série A, apesar do título igual. |
| Passagem aérea - Samuel | 6 parcelas | 12/11/2026 a 12/04/2027 | R$ 188,10 | R$ 1.128,60 | Viagens; todas pendentes. |
| Mercado Livre - blusas (Rash Guard) | Cobrança única | 12/11/2026 | R$ 389,44 | R$ 389,44 | Roupas e Calçados; pendente. |
| Auto Escola - Diego (última parcela) | Cobrança única registrada no painel | 12/11/2026 | R$ 119,66 | R$ 119,66 | Outros; pendente. |
| Adicional Parcela 1 - Passagem Samuel | Lançamento adicional único | 12/11/2026 | R$ 89,00 | R$ 89,00 | Viagens; pendente. |

Todas essas despesas têm origem no cartão Nubank. O arquivo distingue as duas séries “Mercado Livre - kimono (casaco e calça)” por identificadores diferentes; elas não são o mesmo parcelamento para as regras de grupo, edição e exclusão.

O objeto incorporado não possui lançamentos ativos nas coleções de **recebimentos**, **planejamentos**, **investimentos** ou **transferências**. A coleção `tiposInvestimento` também está vazia. O saldo de dinheiro físico está em R$ 0,00.

### 16.5 Item atualmente na lixeira

O arquivo contém uma entrada de lixeira:

- **Título:** `Lançamento: Passagem Samuel - taxas de embarque`.
- **Detalhe:** `Excluído a partir da tela de detalhes.`
- **Quantidade de registros guardados:** 1 despesa.

Esse lançamento não faz parte da lista ativa de despesas enquanto permanecer na lixeira. Ele pode ser restaurado individualmente ou excluído definitivamente.

### 16.6 Observação sobre datas e valores mostrados na interface

O JSON incorporado é a fonte dos cadastros e lançamentos. A tela é atualizada ao iniciar e utiliza a data atual do computador/navegador como período inicial. Assim, valores que aparecem no HTML como resultado de uma renderização anterior não devem ser interpretados como totais fixos do sistema. As listas e os KPIs são calculados novamente a partir dos dados e do período ativo.

---

## 17. Glossário simples

| Termo | Significado neste painel |
|---|---|
| **Lançamento** | Registro de uma entrada, saída, meta, operação ou movimento de saldo. |
| **Origem/Pagamento** | De onde o valor sai: dinheiro, conta corrente ou cartão, conforme o registro. |
| **Destino** | Para onde um recebimento vai ou para onde uma transferência envia dinheiro. |
| **Fatura** | Soma das despesas associadas a um cartão e datadas no mês selecionado. |
| **Parcela** | Parte de uma compra dividida em cobranças, com uma data própria. |
| **Parcelamento** | Grupo de parcelas ligado à mesma compra. |
| **Recorrência** | Série de ocorrências de um lançamento, como semanal, mensal ou anual, nos formulários que permitem essa opção. |
| **Meta** | Valor que se espera receber no futuro. Só gera recebimento no painel quando confirmada com destino. |
| **Aporte** | Entrada de dinheiro em uma aplicação de investimento. |
| **Resgate** | Retirada de valor de uma aplicação de renda fixa. |
| **Compra/Venda** | Operação de compra ou venda de unidades/cotas em renda variável. |
| **Provento/Rendimento** | Valor registrado como provento, dividendo, juros ou rendimento recebido de um investimento. |
| **Custo médio** | Custo médio calculado pelo painel a partir das compras e vendas registradas. |
| **Cotação manual** | Preço de mercado digitado pelo usuário; o painel não o busca automaticamente. |
| **Limite usado** | Parte do limite do cartão comprometida por cobranças pendentes e parcelas futuras pendentes. |
| **Limite livre** | Limite total menos o limite usado calculado. |
| **Saldo líquido** | Indicador que combina dinheiro físico e determinados saldos de conta, descontando as faturas e parcelas pendentes do período. |
| **Drill down** | Abrir um nível mais detalhado ao clicar em um gráfico, indicador, barra, categoria ou linha. |
| **Lixeira** | Área temporária para itens excluídos, permitindo restauração antes da exclusão definitiva. |

## Histórico desta revisão da documentação

Esta revisão foi feita em cima de `Painel_Financeiro_Pessoal_com_Edicao_Completa.html`, não de uma versão anterior do painel. Foram acrescentadas instruções para os novos botões de edição e para o formulário de conta/cartão em modo de atualização; foi detalhado o recálculo ao editar movimentos de poupança/previdência; foram documentadas a edição de aplicações e operações de investimento, de tipos personalizados e o acesso à edição completa de metas; e foi registrada a rotina **Voltar ao início** para listas longas. Os dados descritos na seção 16 foram conferidos contra o bloco `window.INJETADO` do HTML atualizado.

A documentação descreve as rotinas presentes no código. Ela não substitui um teste manual de todos os caminhos com dados reais: quando uma alteração puder afetar muitos lançamentos (por exemplo, editar um parcelamento, mudar o nome de uma conta/cartão ou editar uma série recorrente), recomenda-se revisar o resultado na tela e salvar uma cópia atualizada do HTML depois da conferência.

---

## Resumo geral

O Painel Financeiro Pessoal é um aplicativo local baseado em um único HTML. Os registros financeiros ficam incorporados no próprio arquivo, e as telas são atualizadas por funções JavaScript que recalculam totais e gráficos a partir dos dados ativos e do período selecionado.

Para usar corretamente o painel, o fluxo mais seguro é: cadastrar contas, cartões e categorias; registrar despesas, recebimentos e metas com datas e origens/destinos corretos; utilizar o módulo de parcelamentos para compras parceladas; registrar aportes e operações diretamente no módulo Investimentos; conferir o efeito de transferências nos saldos; revisar status e valores nos detalhes; usar a Lixeira quando precisar desfazer uma exclusão; e salvar o HTML ao concluir alterações importantes.

A principal distinção a manter é entre **fluxo de caixa** (receitas, despesas e transferências), **parcelas/faturas** (cobranças datadas de cartão), **metas** (entradas esperadas até serem confirmadas) e **investimentos** (carteira calculada separadamente, com projeções e cotações manuais). Essa separação evita duplicidade de valores e ajuda a interpretar corretamente os indicadores exibidos.
