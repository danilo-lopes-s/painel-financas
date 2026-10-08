# Painel Financeiro Pessoal — Documentação integral da Versão 15

**Tipo:** manual de usuário, especificação funcional e referência técnica.  
**Versão documentada:** V15 — painel financeiro **sem** os módulos de Investimentos e Transferências.  
**Código-fonte analisado:** `Painel_Financeiro_v15.html`.  
**Idioma da aplicação:** português brasileiro.  
**Moeda de apresentação:** BRL (real brasileiro).  
**Critério:** descrição do comportamento encontrado no HTML/JavaScript, sem usar valores demonstrativos como evidência financeira.

> **Leitura importante:** este material documenta a implementação efetiva da V15, não funcionalidades desejadas ou planejadas. Situações em que a interface apresenta um botão sem a respectiva função, ou em que o cálculo é simplificado, são identificadas explicitamente.

## Sumário

1. Objetivo, escopo e modelo de funcionamento
2. Guia de operação básica: abertura, menu, período, tema, salvamento, dados locais, importação e zeramento
3. Visão Geral: KPIs, rosca, valores por categoria e drill-down
4. Calendário: valores, títulos e agenda
5. Despesas: cadastro e edição inline
6. Recebimentos
7. Recorrências: programação e ajuste de ocorrências
8. Status: opções, cores e edição direta
9. Metas a Receber
10. Contas e dinheiro físico
11. Cartões: indicadores, faturas, percentuais e limites coloridos
12. Parcelamentos
13. Categorias
14. Saldo líquido e evolução financeira
15. Edição, exclusão e navegação
16. Modelo de dados
17. Mapa técnico de funções
18. Limitações e premissas
19. Roteiro de verificação funcional

---

## 1. Objetivo, escopo e modelo de funcionamento

O **Painel Financeiro Pessoal** é uma aplicação local em **um arquivo HTML**, com CSS e JavaScript incorporados. Sua interface é em português brasileiro e apresenta valores em reais (R$). Não há, nesses arquivos, backend próprio, banco de dados remoto, autenticação de usuários, integração bancária automática ou cotações conectadas. A operação principal consiste em abrir o `.html` no navegador, consultar os módulos, efetuar alterações e **salvar uma nova cópia do HTML com o estado atualizado dos dados**.

O sistema reúne duas noções que não devem ser confundidas:

- **Movimentações registradas:** despesas, recebimentos, metas e parcelas com datas, categorias, valores e estados. Alimentam os demonstrativos por período.
- **Saldos atuais cadastrados:** dinheiro físico e os saldos das contas, informados/armazenados como valores atuais, e **não** reconstruídos retroativamente a partir de todas as movimentações.

O painel também tem uma arquitetura de **detalhamento progressivo (drill-down)**: o usuário vê um total ou gráfico, clica nele, consulta os lançamentos que o compõem, abre um registro específico e retorna à apresentação anterior.

### 1.1. Requisitos práticos

1. Manter o HTML como arquivo local; abrir em navegador moderno, preferencialmente em computador.
2. Habilitar JavaScript e permitir downloads de arquivos HTML para usar o salvamento.
3. Não presumir sincronização automática entre equipamentos, navegadores ou cópias do arquivo.
4. Preferir cópias de segurança do HTML antes de modificar ou zerar os dados.
5. Utilizar o próprio botão **Salvar Arquivo** após cada sessão de alterações que devam permanecer no arquivo.

### 1.2. Layout e orientação

| Área | Conteúdo | Uso |
|---|---|---|
| Cabeçalho superior | Nome do painel; alternância de tema; salvar; indicador de alteração; importar JSON; zerar | Comandos globais. |
| Coluna esquerda | Cards de navegação, com ícone, nome do módulo e valor/quantidade | Abrir módulos ou reordenar por arraste na alça lateral. |
| Painel central | Título, período, botões contextuais, tabelas, gráficos e detalhes | Principal área de trabalho. |
| Parte inferior | Composição do saldo líquido e Evolução Financeira Histórica | Consulta transversal, independente do módulo central aberto. |
| Janelas modais | Formulários de criação/edição e confirmações | Alterar entidades e operações. |
| Avisos transitórios | Mensagens de sucesso ou validação | Feedback de comandos. |

O layout possui classes responsivas para telas menores; tabelas e conjuntos de campos podem exigir rolagem. O tema usa variáveis CSS e a classe `body.dark`.

---

## 2. Guia de operação básica

### 2.1. Abrir e navegar

1. Abra o arquivo HTML.
2. Clique em um card lateral, como **Visão Geral**, **Calendário**, **Contas**, **Cartões**, **Despesas**, **Metas a Receber**, **Recebimentos**, **Parcelamentos** ou **Categorias**.
3. Selecione linhas, cartões de métricas, legendas ou barras quando indicarem interação.
4. Use **← Voltar** na parte superior direita para sair de um detalhamento. Para uma categoria/gráfico, o botão pode ser rotulado **← Voltar ao gráfico**.

O menu esquerdo utiliza um cursor de apontamento para **abrir** o card; o arraste usa somente a **alça de ordenação no canto direito**. Arrastar modifica a ordem dos módulos no menu; não altera seus valores.

### 2.2. Seleção global de período

Na barra de período do painel central:

- **Hoje:** muda a data de referência para o dia atual.
- **◀ / ▶:** retrocede/avança um mês em modo *Mês / ano*, ou um dia em modo *Dia*.
- **Exibir → Mês / ano:** usa um campo `month` (`AAAA-MM`).
- **Exibir → Dia:** usa um campo `date` (`AAAA-MM-DD`).
- **Identificador do período:** mostra mês/ano ou data selecionada.

O filtro global é usado por listagens e totais baseados em `emPeriodo()`. A navegação de calendário mostra **o mês inteiro**, ainda que o modo global esteja em *Dia*; a data selecionada recebe um realce. Certos módulos, como **Categorias**, ocultam esse seletor porque não dependem de período. A evolução financeira histórica tem seu **próprio seletor de intervalo**, descrito adiante.

**Atenção:** esse seletor **não recalcula saldos históricos de contas**; ele delimita os lançamentos exibidos e os totais correspondentes. O saldo de cada conta é atual.

### 2.3. Alternar entre tema claro e escuro

Clique em **🌙 Modo escuro** para ativar a aparência escura. O botão passa a oferecer **☀️ Modo claro**. A escolha é registrada no `localStorage` sob a chave `painelFinanceiro_tema` e reaplicada quando a página abre no mesmo contexto de navegador. Essa preferência visual não é, por si só, um lançamento financeiro.

### 2.4. Detectar alterações pendentes

Quando o estado é modificado por um comando que salva dados localmente, `marcarAlteracao()` ativa o indicador **● Alterações não salvas** junto ao botão de salvamento. O indicador é limpo por `limparIndicadorAlteracao()` após a geração/gravação bem-sucedida do HTML segundo o fluxo de implementação. **Detalhe de implementação:** o HTML é capturado em `document.documentElement.outerHTML` antes de o indicador ser limpo; por isso, a cópia gerada pode preservar a marca visual de pendência mesmo quando contém os dados já gravados. Isso não altera o objeto financeiro salvo. Alterar apenas um filtro ou navegar entre telas não é a mesma coisa que alterar lançamentos.

Ao tentar fechar ou recarregar uma sessão com `hasUnsavedChanges=true`, o evento `beforeunload` solicita ao navegador uma advertência sobre mudanças não gravadas. O texto e a decisão final da caixa são controlados pelo navegador.

### 2.5. Salvar a aplicação com seus dados

1. Clique em **💾 Salvar Arquivo**.
2. Se o navegador oferecer a API `showSaveFilePicker`, escolha o destino e confirme a gravação; o arquivo pode ser gravado ou sobrescrito.
3. Se essa API não estiver disponível, o sistema cria um *download* denominado, por padrão, `Painel_Financeiro_Atualizado.html`.
4. Na próxima sessão, **abra o HTML gerado**, não a cópia antiga, caso queira recuperar exatamente o estado que acabou de salvar.

**Detalhe técnico:** `salvarArquivo()` serializa o objeto global `dados` em JSON, escapa caracteres sensíveis ao contexto do script, substitui o conteúdo de `<script id="dados_injetados">` por uma atribuição a `window.INJETADO` e gera um `Blob` contendo o documento HTML inteiro. Assim, o arquivo salvo é **autocontido**: dados e aplicação viajam juntos. O carimbo `dados.timestamp` é atualizado na operação.

**Cuidado com downloads:** a mensagem de download gerado assinala que o fluxo de download foi acionado; confira se o arquivo foi realmente concluído e onde foi armazenado. O painel não possui controle de versões, comparação de arquivos, cópias de segurança automáticas nem sincronização na nuvem.

### 2.6. Armazenamento local versus arquivo salvo

O sistema também escreve no `localStorage` com a chave **`painelFinanceiro_v9`**, uma identificação legada mantida nas versões analisadas. Em `boot()` a prioridade é:

1. `window.INJETADO`, quando o HTML contém dados incorporados;
2. JSON do `localStorage`, se não há dados incorporados;
3. Inicialização de exemplos, quando as duas fontes estão ausentes.

Logo, **salvar alterações localmente não é equivalente a atualizar o HTML original no disco**. Se o arquivo que você abrir já tiver um objeto `window.INJETADO`, ele prevalece sobre alterações presentes apenas no `localStorage`. Para persistência transportável, use **Salvar Arquivo** e reabra a cópia gerada.

### 2.7. Importar JSON — limite da implementação

O cabeçalho exibe **📥 Importar JSON** e um seletor de arquivo `.json` com `onchange="importarDados(event)"`. Contudo, **não existe uma função `importarDados` definida no JavaScript das versões examinadas**. Portanto, a interface sinaliza a intenção de importar, mas **não é correto documentar a importação de JSON como funcional e testada**; no estado atual, selecionar o arquivo tende a resultar em erro de JavaScript. O mecanismo comprovado de preservação é **Salvar Arquivo**, não esse botão.

### 2.8. Zerar painel com confirmação

O botão **🗑️ Zerar painel** abre **duas confirmações sucessivas**. Se ambas forem aceitas:

- os conjuntos de registros e os saldos controlados pelo painel são zerados **em memória, nesta sessão**;
- o indicador de alterações pendentes acende;
- o painel volta para a Visão Geral;
- aparece um aviso de que é necessário **Salvar Arquivo** para tornar permanente o zeramento no HTML.

A operação foi deliberadamente desenhada para **não sobrescrever o `localStorage` imediatamente durante o comando de zerar**. Assim, fechar sem salvar deixa a cópia anterior do HTML intacta. A recuperação depende de **reabrir aquela cópia antiga**, sem substituí-la pelo arquivo zerado; não se trata de uma lixeira nem de um mecanismo de *undo* após salvamento. O campo `zerado:true` impede a reinserção automática de categorias e exemplos nos fluxos condicionados por essa marcação.

---

## 3. Visão Geral: indicadores, gráficos e detalhamento

A página **Visão Geral** apresenta quatro indicadores para o período global:

| Indicador | Cálculo realizado | Caminho ao clicar |
|---|---|---|
| Receitas no Período | Soma de `valor` de `dados.recebimentos` filtrados por período | Lista de recebimentos, datas e valores. |
| Despesas no Período | Soma de `valor` de `dados.despesas` filtradas por período | Lista de despesas. |
| Saldo do Período | `receitas_do_periodo - despesas_do_periodo` | Correspondências positivas e negativas. |
| Metas no Período | Soma de `valor` dos `planejamentos` filtrados por período | Lista de metas previstas. |

A visão traz duas visualizações adicionais:

**Distribuição de Gastos.** O gráfico de rosca utiliza as despesas agrupadas pela categoria. Cada fatia é proporcional a `despesa_da_categoria / despesas_totais × 100`. O centro permite consultar todas as despesas; clicar numa fatia abre a categoria correspondente. A legenda abaixo é igualmente clicável.

**Valores por Categoria.** À direita há linhas alinhadas com nome/ícone da categoria, valor monetário, porcentagem e barra proporcional. Clicar na linha ou barra navega aos lançamentos daquela categoria. Ambas as visualizações usam a cor definida na categoria.

O detalhamento mostra data, descrição, categoria/origem, **status diretamente editável** e valor. Clique num lançamento para abrir sua ficha. Use **Voltar ao gráfico** para retornar à síntese.

**Regra importante:** os indicadores somam registros conforme data e tipo, **sem excluir automaticamente registros `Pendente` ou `Cancelado`**. Alterar um status modifica sua classificação/visualização, mas não transforma o somatório de valores em demonstrativo de caixa efetivamente liquidado.

---

## 4. Calendário de lançamentos

### 4.1. O que aparece

O calendário mensal organiza despesas, recebimentos e metas por **dia de vencimento/lançamento**. As faixas coloridas distinguem saídas, entradas e metas. Acima há um gráfico de barras com **Movimentação no período**: recebimentos, despesas e metas.

### 4.2. Modos de visualização

O seletor do próprio calendário permite escolher:

- **Valores:** os eventos exibem montantes (despesas negativas e entradas positivas); metas mantêm seu marcador próprio.
- **Títulos dos itens:** os eventos mostram a descrição do lançamento, permitindo identificar o objeto da movimentação sem abrir cada detalhe.

Os elementos usam elipse/truncamento visual para caber nos quadrinhos. Acessar o lançamento permite consultar sua descrição completa.

### 4.3. Interações

1. Navegue até um mês.
2. Escolha **Valores** ou **Títulos dos itens**.
3. Clique numa faixa de evento para abrir a despesa, o recebimento ou a meta.
4. Clique na célula de um dia para alternar o período global para o dia selecionado.

Séries recorrentes e parcelas só aparecem nas datas em que **existem registros efetivamente cadastrados**. Não há conexão com calendário externo.

---

## 5. Despesas

### 5.1. Listagem e organização

O módulo **Despesas** exibe gráfico por categoria e uma tabela com colunas **Data, Despesa, Forma, Status e Valor**. A barra de ferramentas permite buscar no título/origem e filtrar por categoria. O código mantém uma configuração de ordenação (`filters.despesas.sort`, inicialmente `date_desc`), mas o HTML principal da barra não apresenta um seletor de ordenação adicional nessa versão.

### 5.2. Criar despesa

Use **+ Nova Despesa** e informe:

- Título;
- Valor positivo;
- Data;
- Categoria cadastrada;
- Origem/pagamento: conta (`conta_<nome>`), cartão (`cartao_<nome>`) ou dinheiro físico (`dinheiro`);
- Frequência: única, semanal, quinzenal, mensal ou anual;
- Quantidade de ocorrências, quando recorrente (1 a 600);
- Campo de quantidade de parcelas no cartão.

**Distinção operacional:** o número `parcelas` caracteriza uma despesa para os agrupamentos de **Parcelamentos**, enquanto os campos `recorrenciaId`, `frequenciaRecorrencia` e `numeroRecorrencia` permitem administrar uma **série recorrente**. Recorrência e parcelamento são mecanismos diferentes; a simples configuração de `parcelas > 1` num lançamento não equivale à criação automática de um cronograma completo no fluxo atual de cadastro.

### 5.3. Editar despesa

- Clique na linha para abrir o detalhe.
- Nos grupos **Dados Básicos** e **Pagamento**, clique no pequeno botão **✏️ ao lado do campo** para editar título, valor, data, categoria, origem ou número de parcelas.
- Digite/selecione o novo valor, confirme em **✓** ou cancele em **✕**.
- O status é modificado num seletor independente.
- Se a despesa pertencer a uma série recorrente, o ícone junto a **Repetição** abre o formulário para alterar frequência, início, quantidade e escopo das alterações.
- **Excluir Permanentemente** pede confirmação e remove o registro escolhido.

Editar isoladamente o campo de uma ocorrência não é sinônimo de reprogramar a série inteira: para isso, use a edição de **Repetição**.

---

## 6. Recebimentos

A tela lista **Data, Origem, Destino, Status e Valor**, além de um gráfico de recebimentos agrupados pela conta de destino.

**Novo recebimento:** informe nome da origem/cliente, valor, data, tipo de entrada (**Pix/conta** ou **Dinheiro físico**) e, se for Pix, a conta de destino. Também há recorrência **única, semanal, quinzenal, mensal e anual**, com quantidade definida pelo usuário.

**Detalhe do recebimento:** permite edição campo a campo, alteração direta de status e exclusão. Quando a entrada integra série recorrente, existe controle de **Repetição** para editar a série ou apenas a ocorrência. Alterar o tipo para dinheiro ajusta a referência de destino para `Dinheiro`; selecionar uma conta bancária identifica a entrada como `Pix`.

**Importante:** registrar um recebimento **não credita automaticamente** a propriedade `saldo` da conta. Os recebimentos alimentam os cálculos por período; saldos atuais de contas permanecem registrados separadamente.

---

## 7. Recorrências: regras completas

O mesmo mecanismo de séries é aplicado a **despesas** e **recebimentos**:

| Frequência | Regra para a ocorrência n |
|---|---|
| Única | Apenas uma ocorrência. |
| Semanal | Data inicial + 7 dias × índice. |
| Quinzenal | Data inicial + 15 dias × índice. |
| Mensal | Avanço de meses, limitado ao último dia válido do mês. |
| Anual | Avanço de anos, ajustando o dia inválido (ex.: 29 de fevereiro). |

Os registros de uma série compartilham `recorrenciaId` e mantêm `numeroRecorrencia`, `totalRecorrencias`, `frequenciaRecorrencia`, `inicioRecorrencia` e `tituloBaseRecorrencia`. Seus títulos são identificados como `Nome (n/total)` quando existe mais de uma ocorrência.

**Ao aumentar a quantidade**, são criadas apenas as ocorrências faltantes. **Ao diminuir**, são removidas as últimas ocorrências da própria série. O algoritmo procura conservar IDs e **status individuais** das ocorrências remanescentes, reordenadas por data. As novas datas surgem automaticamente nas listas e no calendário, que são redesenhados com o conjunto de dados atualizado.

**Ao editar uma série:** o formulário oferece **Toda a série (inclusive quantidade)** ou **Somente esta ocorrência**. No segundo caso, você altera apenas os campos da ocorrência escolhida; no primeiro, a série é recalculada a partir da data e frequência informadas.

**Limites:** quantidade de 1 a 600; a edição da série atualiza seus registros e não oferece simulação de impactos antes de salvar. Registros legados sem `recorrenciaId` não passam a constituir uma série administrável automaticamente apenas por terem nomes parecidos.

---

## 8. Status: tipos, cores e edição direta

Os estados disponíveis são:

| Registro | Valores selecionáveis |
|---|---|
| Despesa / parcela | **Pendente**, **Pago**, **Cancelado** |
| Recebimento | **Pendente**, **Recebido**, **Cancelado** |
| Meta | **Planejado**, **Confirmado**, **Cancelado** |

A visualização também incorpora o contexto temporal do registro não concluído:

- **Concluído (`Pago`, `Recebido`, `Confirmado`):** verde.
- **Atrasado:** vermelho quando a data anterior ao dia corrente e não concluído.
- **É Hoje:** laranja quando a data é hoje.
- **Em N d:** amarelo quando faltam até cinco dias.
- **Futuro / Planejado:** azul em situações futuras.
- **Cancelado:** cinza.

A lista de status mostra as opções de situação armazenada, mas a **opção atualmente selecionada pode aparecer com um rótulo temporal**, como *Atrasado*, sem substituir internamente `Pendente`. A cor é determinada por `statusInfo()`, que utiliza `getStatusData()` e a data de referência **de hoje**, não a data escolhida no seletor de período.

**Como alterar:** use o menu na própria linha de tabela ou na ficha individual, sem precisar entrar em outro formulário. Em parcelamentos, o menu do **grupo** muda o status das parcelas **contidas no período selecionado**; se forem mistos, aparece indicação *Misto*. Alterar o status não remove nem ajusta automaticamente o valor da despesa, receita ou meta nos totalizadores.

---

## 9. Metas a Receber

O módulo registra **recebimentos previstos/planejados**, não receitas efetivamente recebidas.

**+ Nova Meta:** descrição, valor esperado, data prevista e, opcionalmente, geração de lote com repetição única, semanal, quinzenal ou mensal, e quantidade. O lote cria itens individuais identificáveis (`1/N`, `2/N`, ...). A versão analisada **não oferece a mesma gestão de séries com expansão/redução** implementada para despesas e recebimentos; editar uma meta ajusta o item selecionado.

A lista utiliza as colunas **Data Prevista, Descrição da Meta, Status e Valor**. Seu status é alterável diretamente entre **Planejado, Confirmado, Cancelado**. A ficha permite editar **descrição, valor e data** campo a campo, ou excluir o registro.

No painel inicial, **Metas no Período** soma os valores cadastrados das metas naquele período, independentemente do status. Uma meta confirmada não cria automaticamente um lançamento de recebimento; caso queira contabilizar uma receita também no módulo **Recebimentos**, é necessário gerenciá-la separadamente.

---

## 10. Contas e dinheiro físico

A página **Contas** reúne:

- dinheiro físico atual;
- saldo atual somado das contas bancárias;
- entradas no período;
- saídas com origem `conta_...` no período;
- gráfico de **Saldos atuais por conta**;
- tabela de contas, incluindo linha de **Dinheiro**.

Clique diretamente na linha da conta ou de **Dinheiro** (sem botão intermediário) para abrir **Lançamentos da Conta**. O detalhe mostra saldo atual cadastrado, entradas e saídas filtradas, gráfico de movimentação e tabela de lançamentos, com acesso à ficha e ao editor do item.

**Semântica essencial:** `dados.contas[].saldo` e `dados.dinheiroFisico` são os **valores atuais registrados**. A barra mensal/dia filtra os lançamentos relacionados, mas não reconstrói a trajetória histórica de saldo. A interface analisada **não apresenta botão de criação/edição de conta bancária individual** no próprio módulo: ele atua como consulta dos dados de contas já existentes no arquivo.

---

## 11. Cartões

### 11.1. Painel consolidado

O módulo oferece quatro **cards interativos**:

1. **Limite total:** soma dos limites cadastrados dos cartões.
2. **Usado / faturas do mês:** soma das despesas associadas a cartões **no mês global**, independentemente do modo Dia.
3. **Livre no mês:** diferença entre limite total e fatura do mês.
4. **Lançamentos do período:** soma das despesas com origem `cartao_...` filtradas pela data global (mês ou dia).

Clicar em cada indicador abre a **composição por cartão**, com valores, percentuais e acesso ao cartão. As tabelas e gráficos trazem participação do cartão no **limite total**, no **total das faturas** e no **próprio limite**.

### 11.2. Fórmulas de limite e participação

```text
Limite total               = Σ limites dos cartões
Fatura do cartão no mês    = Σ despesas com origem = cartao_<nome>
                             e data no mês selecionado
Fatura total do mês        = Σ faturas de todos os cartões
Limite livre do cartão     = limite do cartão − fatura do cartão
Limite livre total         = limite total − fatura total
% livre do cartão          = 100 × limite livre / limite do cartão
% usado do cartão          = 100 × fatura do cartão / limite do cartão
% da fatura total          = 100 × fatura do cartão / fatura total
% do limite total          = 100 × limite do cartão / limite total
```

Divisões por zero são apresentadas como **0,0%**. Barras coloridas de limite livre são visualmente limitadas entre 0% e 100% de largura, mesmo que o valor monetário ultrapasse alguma faixa.

### 11.3. Cores da disponibilidade

| Disponibilidade do limite | Cor |
|---|---|
| **≥ 75% livre** | Verde |
| **≥ 40% e < 75%** | Amarelo |
| **≥ 25% e < 40%** | Laranja |
| **< 25%** | Vermelho |

O gráfico de faturas mostra barras com a cor da **disponibilidade do limite**, não a cor de status de pagamento. Sua legenda explica a escala.

### 11.4. Detalhes do cartão

Clique numa linha do cartão ou em sua barra para abrir a ficha. Você verá limite, fatura do mês, limite livre, vencimento, despesas por categoria e lançamentos do cartão com status direto e botão de edição por registro.

**Limitação:** o cálculo “livre” subtrai a fatura agregada do mês, **sem reservar automaticamente limite para todas as parcelas futuras**; também não há processamento completo de fechamento/pagamento de faturas ou conciliação com a operadora do cartão. Os cartões já existentes podem ser consultados, mas este arquivo não contém interface dedicada a cadastrar um novo cartão.

---

## 12. Parcelamentos

Parcelamento é um **agrupamento de despesas** cujo campo `parcelas` é maior que 1. O algoritmo `gruposParcelas()` utiliza, preferencialmente, um `parcelamentoId` explícito; quando ausente, agrupa por **origem + título sem sufixo `(n/N)`**. Isso permite reconhecer parcelas relacionadas mesmo em dados legados, mas recomenda-se atenção a títulos repetidos na mesma origem.

A visão principal traz quantidade de parcelamentos encontrados no período, quantidade de parcelas no período e valor total correspondente. A tabela informa compra, origem/cartão, número de parcelas, **status de grupo**, valor no período e caminho para o cronograma.

Ao selecionar um parcelamento, a ficha exibe:

- **Total cadastrado da compra:** soma dos registros do grupo;
- **Quantidade contratada:** campo `parcelas` do grupo;
- **No período selecionado:** soma apenas dos registros filtrados;
- **Período do cronograma:** primeira e última data da série;
- tabela de todas as parcelas, incluindo anteriores e futuras, com status e edição.

O cronograma é apresentado **sem quadro colorido redundante de detalhe selecionado**. O status de uma parcela pode ser alterado na linha; o status agregado no módulo de parcelamentos aplica uma alteração em lote somente às parcelas do período global selecionado. A edição da parcela usa o ícone de lápis na tabela.

---

## 13. Categorias

A página **Gerenciar Categorias** é um catálogo administrativo independente do período; **não exibe gráfico de despesas por categoria**, já apresentado na Visão Geral.

Cada card mostra **ícone, nome e botão ✏️**. **+ Nova** abre o formulário para nome, cor (seletor de cor hexadecimal) e emoji/ícone escolhido da biblioteca. Editar permite substituir os três atributos.

O uso dessas propriedades se reflete nas etiquetas dos lançamentos, nas legendas e nas barras/porções de gráficos. O código associa despesas à categoria **pelo nome**; ao renomear uma categoria, **não há rotina demonstrada de migração em massa** do campo `categoria` já gravado nas despesas. Registros antigos podem manter o nome anterior; planeje alterações de nomenclatura com cuidado.

---

## 14. Saldo líquido e Evolução Financeira Histórica

### 14.1. Composição do Saldo Líquido

O card inferior é clicável. A equação de `refresh()` é:

```text
Saldo líquido exibido = dinheiroFisico + Σ saldo das contas − totalFaturasMes()
```

`totalFaturasMes()` computa despesas lançadas com origens de cartões no mês selecionado. A expressão **não é um balanço contábil completo**: não inclui passivos não modelados, não tenta reconstruir saldo bancário a partir dos lançamentos nem trata pagamentos de cartões como conciliação externa. O título *Faturas Pendentes* é um rótulo visual; o cálculo não filtra necessariamente as despesas de cartão conforme o status `Pago`.

### 14.2. Evolução Financeira Histórica

Há duas métricas:

- **Saldo Projetado:** soma acumulada de `recebimentos − despesas` ao longo dos meses abrangidos pela janela, **começando em zero no início dessa janela**. Não representa o saldo bancário atual nem uma série contábil histórica verificada.
- **Entradas x Saídas:** duas séries mensais separadas, uma de recebimentos e outra de despesas.

O seletor admite **Últimos 3, 6 ou 12 Meses** e **Personalizado**, com **data inicial e final**. Nas opções predefinidas, a janela termina no mês de referência escolhido no painel central; no personalizado, usa precisamente as datas preenchidas. A agregação é mensal; meses extremos respeitam os limites de data informados. O código valida início ≤ fim e processa até **240 meses** no gráfico financeiro.

O gráfico é um **SVG** gerado por JavaScript, com eixos, grades, polilinhas, pontos informativos com `title` e legendas. Suporta valores negativos na escala. As cores das séries são convencionais (verde para entradas, vermelho para saídas e roxo/azulado para saldo projetado).

**Ponto crítico de interpretação:** a evolução usa o **valor dos lançamentos cadastrados**, não seus estados de quitação. Por isso, “Saldo Projetado” é uma progressão baseada no registro financeiro, não uma conciliação de fluxo realizado. Tampouco deve ser confundida com o **Saldo Líquido** do quadro inferior, cujo cálculo tem outra base.

---

## 15. Fluxo de edição, exclusão e navegação

A navegação é controlada pelo par `visao`/`itemAtual`. `selecionarVisao()` escolhe a tela e registra algumas transições de detalhamento em `historicoVisoes`; `renderVisao()` encaminha para o renderizador correto; `navegarVoltar()` utiliza o histórico e, quando necessário, retorna à Visão Geral.

A criação e a edição tradicionalmente usam **modais** (`openModal`/`closeModal`), enquanto campos nas fichas de despesas, recebimentos e metas usam **edição inline** (`campoDetalhe`/`iniciarEdicaoCampo`). Células de status apresentam o seletor e impedem que o clique acione involuntariamente a navegação da linha. Alterações válidas salvam o estado local, atualizam os resumos e redesenham a interface por `refresh()`.

A exclusão de despesa, recebimento ou meta chama `excluirItem()` após confirmação. A exclusão é uma ação sobre os dados da sessão; sua permanência no HTML depende do fluxo de **Salvar Arquivo**. Não há trilha de auditoria imutável, versionamento de alterações ou permissão por usuário.

---

## 16. Modelo de dados: estruturas e relacionamentos

O estado primário é um objeto mutável global chamado `dados`. Suas estruturas principais são:

| Estrutura | Campos essenciais | Relações |
|---|---|---|
| `contas[]` | `nome`, `tipo`, `saldo` | Destino de recebimento; origem de despesa. |
| `cartoes[]` | `nome`, `bandeira`, `limite`, `vencimento`, `faturaAtual` (legado) | Origem `cartao_<nome>` nas despesas. |
| `despesas[]` | `id`, `titulo`, `categoria`, `valor`, `data`, `origem`, `parcelas`, `status` | Categorias, contas, cartões, recorrências e grupos de parcelas. |
| `recebimentos[]` | `id`, `titulo`, `valor`, `data`, `categoriaRecebimento`, `conta`, `status` | Conta ou dinheiro físico; recorrências. |
| `planejamentos[]` | `id`, `descricao`, `valor`, `data`, `status` | Metas previstas. |
| `categorias[]` | `id`, `nome`, `icon`, `cor` | Etiquetas e gráficos de despesas. |
| `dinheiroFisico` | Número | Saldo registrado de dinheiro em mãos. |
| `navOrder[]` | Identificadores de visões | Ordem dos cards laterais. |
| `timestamp` | Momento de salvamento | Metadado temporal. |
| `exemplosV13`, `zerado` | Marcadores de estado | Controlam injeção de exemplos e inicialização após zeramento. |

Campos de recorrência: `recorrenciaId`, `numeroRecorrencia`, `totalRecorrencias`, `frequenciaRecorrencia`, `inicioRecorrencia`, `tituloBaseRecorrencia`. Campos de parcela: `parcelamentoId` e `numeroParcela`, quando disponíveis, além do total `parcelas`.

**Representação:** datas de lançamentos são armazenadas como `AAAA-MM-DD` em texto ISO, úteis para filtragem e ordenação; montantes são convertidos via `Number()` e mostrados por `Intl.NumberFormat('pt-BR', {style:'currency', currency:'BRL'})`. Valores inválidos na função de formatação genérica podem aparecer como zero; validações de criação evitam alguns desses casos.

O utilitário `today()` usa `new Date().toISOString().slice(0,10)` (data UTC), o que pode fazer o rótulo **Hoje** e a classificação temporal diferirem da data civil local em horários próximos à virada do dia.

**Não há chaves estrangeiras formais:** vínculos com contas, cartões e categorias usam nomes codificados em strings. Renomear entidades sem migrar os registros associados pode quebrar referências no modelo.

---

## 17. Mapa técnico das funções centrais

| Componente/função | Responsabilidade |
|---|---|
| `boot()` | Priorizar dados embutidos, armazenamento local ou inicialização; preparar IDs e ordem do menu. |
| `gerarDadosDeExemplo()` / `adicionarExemplosV13()` | Popular conjuntos demonstrativos conforme marcadores de inicialização; não fazem parte das regras contábeis. |
| `money()`, `sum()`, `formatarData()`, `uid()` | Formatação, soma, datas e criação de identificadores. |
| `salvarDadosLocal()`, `marcarAlteracao()`, `limparIndicadorAlteracao()` | Manter estado no navegador e aviso de pendências. |
| `salvarArquivo()` | Gerar um HTML autocontido com dados embutidos. |
| `zerarPainel()` | Zerar o estado da sessão mediante dupla confirmação. |
| `alternarTema()`, `aplicarTema()` | Mudar aparência e manter preferência local. |
| `mudarMes()`, `selecionarPeriodo()`, `alterarModo()`, `moverPeriodo()`, `emPeriodo()` | Navegação temporal e filtragem de itens. |
| `refresh()` | Recalcular menus, saldos e gráficos, e redesenhar a visão atual. |
| `selecionarVisao()`, `renderVisao()`, `navegarVoltar()`, `header()` | Roteamento interno, detalhe, retorno e título/ações. |
| `initDragAndDrop()`, `salvarOrdemCards()`, `restaurarOrdemCards()` | Ordenação do menu lateral. |
| `renderPrincipal()`, `abrirCategoriaPorGrafico()`, `renderCorrespondencias()` | KPIs, gráfico de rosca, barras de categoria e drill-down. |
| `renderCalendario()`, `alterarExibicaoCalendario()` | Grade mensal e alternância entre valores e títulos. |
| `renderSimple()` e renderizadores de detalhe | Construção dos módulos e de suas tabelas. |
| `statusInfo()`, `statusControle()`, `atualizarStatus()` | Cor, rótulo e atualização de status individuais. |
| `statusControleGrupo()`, `atualizarStatusGrupo()` | Situação mista e alteração de status de parcelas do período. |
| `dataOcorrencia()`, `recorrenciasDo()`, `salvarSerie()`, `alterarEscopoRecorrencia()` | Datas e administração de séries recorrentes. |
| `campoDetalhe()`, `iniciarEdicaoCampo()` | Editores inline dos lançamentos. |
| `salvarDespesa()`, `salvarRecebimento()`, `salvarPlanejamento()`, `excluirItem()` | Persistência lógica dos registros. |
| `renderCategorias()`, `salvarCategoria()` | Administração de nomes, ícones e cores. |
| `faturaCartao()`, `totalFaturasMes()`, `corDisponibilidade()`, `pctCartao()` | Faturas e percentuais/cores dos cartões. |
| `renderEvolution()`, `setMetric()` | Séries e desenho da evolução financeira histórica. |
| `graficoBarras()`, `agrupar()`, `graficoDaVisao()` | Gráficos horizontais, agrupamentos e métricas de módulo. |

---

## 18. Regras, premissas e limitações importantes

1. **Arquivo local, sem servidor:** a aplicação não sincroniza com extratos bancários ou operadoras de cartão.
2. **Saldos atuais não são reprocessados:** despesas e recebimentos registrados não atualizam automaticamente `saldo` de conta nem `dinheiroFisico`.
3. **Status não exclui somatórios:** *Pago*, *Recebido*, *Pendente* e *Cancelado* são exibidos, mas os filtros financeiros principais usam data e valor, não conciliação por status.
4. **Parcelamentos dependem dos registros existentes:** o campo `parcelas` não substitui um processo automático de geração de todas as mensalidades.
5. **Conta e cartão são entidades de consulta nesta interface:** a versão examinada não apresenta formulário específico para cadastrar/editar suas propriedades administrativas.
6. **Metas não viram recebimentos automaticamente** e a geração de metas repetidas não tem o mesmo gerenciador de séries das receitas e despesas.
7. **`Importar JSON` está visualmente presente, mas sem implementação de função no código**; não trate como alternativa ao salvamento HTML.
8. **Os exemplos automáticos pertencem à inicialização**, não a uma fonte de dados confiável; qualquer análise deve distinguir dados reais e demonstrativos.
9. **Armazenamento e segurança:** não há criptografia de dados no arquivo nem autenticação; quem recebe o HTML salvo pode ler os registros financeiros incorporados.
10. **Precisão e finalidade:** as métricas servem para gestão pessoal e visualização da estrutura registrada, não como demonstração auditada, extrato bancário conciliado ou apuração tributária automática.

---

## 19. Procedimentos de verificação funcional

Use este roteiro quando atualizar o HTML ou migrar dados:

- [ ] Abrir o arquivo e confirmar que a Visão Geral e os cards carregam.
- [ ] Trocar entre tema claro/escuro e reabrir no mesmo navegador.
- [ ] Navegar por mês e dia; usar **Hoje**; verificar os totais dos filtros.
- [ ] Mudar calendário entre **Valores** e **Títulos dos itens**; abrir um evento.
- [ ] Criar, editar e excluir uma despesa e um recebimento; verificar status e gráficos.
- [ ] Criar séries recorrentes; aumentar e diminuir ocorrências; conferir o calendário.
- [ ] Alterar o status de um item e de um grupo de parcelas do período.
- [ ] Consultar cartões: soma dos limites, faturas, percentuais, cores e detalhe por cartão.
- [ ] Abrir **Valores por Categoria**, selecionar uma categoria e retornar ao gráfico.
- [ ] Verificar evolução de 3, 6, 12 meses e intervalo personalizado.
- [ ] Alterar uma categoria e observar o comportamento dos lançamentos antigos com o nome anterior.
- [ ] Confirmar o indicador de alteração; salvar HTML; fechar e reabrir a cópia nova.
- [ ] Testar o zeramento **somente em cópia descartável**, antes/depois de salvar.

