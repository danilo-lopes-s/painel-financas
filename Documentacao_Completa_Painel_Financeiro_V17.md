# Painel Financeiro Pessoal — Documentação integral da Versão 17

**Tipo:** manual de usuário, especificação funcional, modelo de dados e referência técnica.  
**Versão documentada:** V17 — painel financeiro **com** Investimentos e Transferências, mantendo os recursos da V15.  
**Código-fonte analisado:** `Painel_Financeiro_v17_Demonstracao.html`.  
**Idioma da aplicação:** português brasileiro.  
**Moeda de apresentação:** BRL (real brasileiro).  
**Critério:** regras e estrutura verificadas no código. **Todos os preços, taxas, nomes de posições e valores colocados no HTML apenas para demonstração foram deliberadamente desconsiderados.**

> **Leitura importante:** a V17 usada como base contém rotinas de inicialização de uma carteira fictícia. Essas rotinas são descritas como arquitetura de demonstração, sem transformar seus dados ilustrativos em métricas de mercado. Este manual é autossuficiente: também explica as funções herdadas da V15.

## Sumário

**Núcleo financeiro, navegação e armazenamento:**

1. Objetivo, escopo e modelo de funcionamento
2. Guia de operação básica (inclusive salvar, modo escuro e zerar)
3. Visão Geral, KPIs e gráficos
4. Calendário
5. Despesas
6. Recebimentos
7. Recorrências
8. Status
9. Metas a Receber
10. Contas e dinheiro físico
11. Cartões e métricas de limite
12. Parcelamentos
13. Categorias
14. Saldo líquido e evolução financeira
15. Fluxo de edição e navegação
16. Modelo de dados-base
17. Mapa de funções-base
18. Regras e limitações gerais
19. Checklist de testes do núcleo

**Extensão Investimentos e Transferências:**

20. Separação patrimonial e visão dos módulos
21. Investimentos: indicadores e distribuição
22. Renda Fixa: catálogo, campos, fórmulas, resgates e projeções
23. Renda Variável: catálogo, operações, preço médio e resultados
24. Evolução de investimentos, drill-down, proventos e vencimentos
25. Tipos de investimento personalizados
26. Transferências: cadastro, cálculos, histórico e reversão
27. Dados adicionais da V17
28. Funções técnicas da V17
29. Limitações particulares
30. Testes específicos
31. Conclusão arquitetural

---

## 1. Objetivo, escopo e modelo de funcionamento

O **Painel Financeiro Pessoal** é uma aplicação local em **um arquivo HTML**, com CSS e JavaScript incorporados. Sua interface é em português brasileiro e apresenta valores em reais (R$). Não há, nesses arquivos, backend próprio, banco de dados remoto, autenticação de usuários, integração bancária automática ou cotações conectadas. A operação principal consiste em abrir o `.html` no navegador, consultar os módulos, efetuar alterações e **salvar uma nova cópia do HTML com o estado atualizado dos dados**.

O sistema reúne duas noções que não devem ser confundidas:

- **Movimentações registradas:** despesas, recebimentos, metas e parcelas com datas, categorias, valores e estados. Alimentam os demonstrativos por período.
- **Saldos atuais cadastrados:** dinheiro físico e os saldos das contas, informados/armazenados como valores atuais, e **não** reconstruídos retroativamente a partir de todas as movimentações.

Na V17, **investimentos formam patrimônio separado do saldo bancário**, enquanto **transferências alteram diretamente saldos atuais, sem criar receita ou despesa** (capítulos 20 a 26).

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
2. Clique em um card lateral, como **Visão Geral**, **Calendário**, **Contas**, **Cartões**, **Despesas**, **Metas a Receber**, **Recebimentos**, **Parcelamentos**, **Categorias**, **Investimentos** ou **Transferências**.
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

- os conjuntos de registros, incluindo **investimentos, transferências e tipos personalizados**, e os saldos controlados pelo painel são zerados **em memória, nesta sessão**;
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

**Semântica essencial:** `dados.contas[].saldo` e `dados.dinheiroFisico` são os **valores atuais registrados**. Na V17, o módulo **Transferências** também altera esses saldos atuais em pares de débito e crédito; diferentemente das transferências, a simples criação de despesa ou recebimento não os altera. A barra mensal/dia filtra os lançamentos relacionados, mas não reconstrói a trajetória histórica de saldo. A interface analisada **não apresenta botão de criação/edição de conta bancária individual** no próprio módulo: ele atua como consulta dos dados de contas já existentes no arquivo.

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

O card inferior é clicável. **O patrimônio investido da V17 não está incluído nesse saldo**. A equação de `refresh()` é:

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

O estado primário é um objeto mutável global chamado `dados`. A tabela abaixo mostra as estruturas **herdadas do núcleo V15**; os novos objetos de investimento e transferência constam no capítulo 27:

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
2. **Saldos atuais não são reprocessados a partir de receitas/despesas:** despesas e recebimentos registrados não atualizam automaticamente `saldo` de conta nem `dinheiroFisico`. **Transferências da V17**, em contraste, debitam e creditam esses saldos.
3. **Status não exclui somatórios:** *Pago*, *Recebido*, *Pendente* e *Cancelado* são exibidos, mas os filtros financeiros principais usam data e valor, não conciliação por status.
4. **Parcelamentos dependem dos registros existentes:** o campo `parcelas` não substitui um processo automático de geração de todas as mensalidades.
5. **Conta e cartão são entidades de consulta nesta interface:** a versão examinada não apresenta formulário específico para cadastrar/editar suas propriedades administrativas.
6. **Metas não viram recebimentos automaticamente** e a geração de metas repetidas não tem o mesmo gerenciador de séries das receitas e despesas.
7. **`Importar JSON` está visualmente presente, mas sem implementação de função no código**; não trate como alternativa ao salvamento HTML.
8. **Os exemplos automáticos pertencem à inicialização**, não a uma fonte de dados confiável; qualquer análise deve distinguir dados reais e demonstrativos.
9. **Armazenamento e segurança:** não há criptografia de dados no arquivo nem autenticação; quem recebe o HTML salvo pode ler os registros financeiros incorporados.
10. **Precisão e finalidade:** as métricas servem para gestão pessoal e visualização da estrutura registrada, não como demonstração auditada, extrato bancário conciliado ou apuração tributária automática.

---

## 19. Procedimentos de verificação funcional do núcleo financeiro

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


---

## 20. V17: separação financeira e novos módulos

A V17 incorpora **Investimentos** e **Transferências** como áreas próprias no menu lateral, preservando Visão Geral, Calendário, Contas, Cartões, Despesas, Metas, Recebimentos, Parcelamentos e Categorias.

A separação implementada é intencional:

| Domínio | Onde os valores ficam registrados | Consequência operacional |
|---|---|---|
| **Contas / dinheiro físico** | `dados.contas[].saldo`, `dados.dinheiroFisico` | Valores monetários de disponibilidade bancária/física. |
| **Receitas/despesas** | `dados.recebimentos[]`, `dados.despesas[]` | Influenciam totais e séries da movimentação financeira por período. |
| **Transferências internas** | `dados.transferencias[]` + alteração direta dos saldos de origem/destino | Mover recursos entre saldos, sem criar receita/despesa. |
| **Investimentos** | `dados.investimentos[]` com seu histórico interno `movimentos[]` | Carteira própria, com avaliação e relatórios independentes dos saldos. |

**O patrimônio de investimentos não entra no card “Composição do Saldo Líquido”** e seus aportes/resgates não são automaticamente debitados/creditados em contas bancárias. Uma eventual transferência entre conta e corretora precisa ser gerenciada com cuidado pelo usuário; o catálogo de investimentos não cria, por si, uma transação bancária de compensação.

### 20.1. Mapa de navegação da área nova

```text
Investimentos
├── Resumo consolidado da carteira
│   ├── Patrimônio investido → lista dos investimentos
│   ├── Renda Fixa → catálogo e aplicações
│   ├── Renda Variável → catálogo e aplicações
│   ├── Rendimentos / proventos → histórico consolidado
│   ├── Distribuição entre classes (clicável)
│   ├── Consolidação de principal, valor e resultado
│   ├── Evolução do patrimônio (pontos mensais clicáveis)
│   └── Vencimentos previstos de renda fixa
├── Catálogo Renda Fixa → Tipo → Aplicações → Detalhe / Operações
├── Catálogo Renda Variável → Tipo → Ativos → Detalhe / Operações
└── Cadastrar investimento ou novo tipo

Transferências
├── Indicadores de dinheiro físico, saldos e total disponível
├── Transferido no período
├── Histórico por data
├── Nova transferência
└── Editar / excluir e reverter transferências
```

---

## 21. Investimentos: visão geral e indicadores

Clique no card **📈 Investimentos** para abrir a visão principal. Os dados são consolidados pela função `resumoCarteira()`, que chama `invAnalise()` para analisar cada aplicação.

### 21.1. Indicadores do painel

| Card ou linha | Origem do valor | Interação |
|---|---|---|
| **Patrimônio investido** | Soma dos valores atuais/estimados das aplicações | Abre a lista completa. |
| **Renda Fixa** | Soma das aplicações da classe `fixa` | Abre o catálogo de renda fixa. |
| **Renda Variável** | Soma dos ativos da classe `variavel` | Abre o catálogo de renda variável. |
| **Rendimentos / proventos registrados** | Soma das operações `rendimento` e `provento` | Abre o histórico desses recebimentos. |
| **Total histórico aportado** | Soma de operações do tipo `aporte` e `compra` | Linha da consolidação, sem descontar resgates e vendas. |
| **Custo / principal em aberto** | Somatório do principal vigente na renda fixa e do custo remanescente na renda variável | Linha da consolidação. |
| **Valor atual / estimado** | Avaliação somada de todos os investimentos | Linha da consolidação. |
| **Variação + lucro realizado** | Soma dos resultados de cada aplicação e dos lucros realizados nas vendas | Indicador que depende de informação de cotação e do modelo simplificado adotado. |
| **Proventos / juros recebidos** | Soma dos fluxos classificados como distribuição de rendimentos | Informativo separado do valor atual da carteira. |

### 21.2. Distribuição da carteira

O bloco **Distribuição da carteira** contém linhas/barras da renda fixa e da variável, cada uma com valor e percentual do total. A fórmula é:

```text
Participação de uma classe (%) = 100 × valor da classe / valor total da carteira
```

As linhas são interativas e levam ao catálogo da classe. **As barras não representam rentabilidade:** representam a composição percentual do patrimônio atualmente avaliado.

### 21.3. Lista de aplicações

A tabela reúne **Ativo**, **Tipo**, **Cotas / principal**, **Valor atual / estimado**, **Variação** e acesso ao detalhe. Para renda fixa, o campo de posição mostra o **principal vigente**; na renda variável, mostra a **quantidade de unidades**. A cor da variação distingue valores positivos e negativos. Clique num ativo para acessar suas características, seus lançamentos e suas projeções.

---

## 22. Catálogo de Renda Fixa: tipos, explicações e campos

O catálogo é formado por itens definidos em `CATALOGO_INV.fixa`, acessíveis em **Investimentos → Renda Fixa**. Clique no tipo para ler o texto de **Como funciona** e consultar/adicionar suas aplicações.

| Tipo | Funcionamento resumido exibido no sistema | Campos adicionais específicos |
|---|---|---|
| **Poupança** | Aplicação bancária com regras legais de remuneração e aniversário. | Banco/emissor; dia de aniversário. |
| **Tesouro Direto** | Títulos públicos, inclusive Selic, Prefixado e IPCA+, sujeitos a marcação a mercado na venda antecipada. | Subtipo/título, preço unitário de referência, quantidade de títulos de referência, custódia/corretora. |
| **CDB** | Certificado bancário prefixado ou indexado; emissor, liquidez, carência e enquadramento no FGC importam. | Banco/emissor; liquidez/resgate. |
| **LCI** | Título bancário do setor imobiliário, com regras próprias de prazo e tributação. | Banco/emissor; carência. |
| **LCA** | Título bancário do agronegócio, com regras próprias de prazo e tributação. | Banco/emissor; carência. |
| **Debêntures** | Dívida de empresa, sujeita a risco do emissor e, conforme o caso, cupons, amortização e variação de preço. | Rating, garantias, amortizações, pagamento de cupons. |

Os textos e campos são informativos e de cadastro. **O motor matemático usa uma fórmula geral de projeção**, não reproduz todas as características contratuais, calendários de cupons, regras específicas de cada emissor, condições de liquidez ou marcação a mercado de cada título.

### 22.1. Dados gerais da aplicação de renda fixa

Ao clicar em **+ Novo investimento** ou **+ Adicionar aplicação**, preencha:

1. **Classe:** Renda Fixa.
2. **Tipo de investimento:** Poupança, Tesouro Direto, CDB, LCI, LCA, Debêntures ou tipo personalizado.
3. **Nome da aplicação:** identificação legível para uso cotidiano (**obrigatória**).
4. **Ticker / identificação:** código ou referência livre.
5. **Banco / emissor / corretora:** instituição associada.
6. **Data inicial:** data de referência da aplicação.
7. **Indexador:** Prefixado, CDI, Selic, IPCA+, TR+ ou Outro.
8. **Vencimento:** data final, quando aplicável.
9. **Taxa anual contratada (% a.a.):** parâmetro do modo prefixado.
10. **Percentual do indexador (%):** multiplicador, útil em CDI.
11. **Taxa base anual hipotética (%):** hipótese para CDI, Selic, inflação ou TR.
12. **Spread anual (% adicional):** margem adicional sobre o indexador, na fórmula específica.
13. **Tributação estimada:** Isento, IR regressivo ou outra opção de tratamento disponível no formulário.
14. **Saldo atual real informado (R$):** campo opcional que **substitui a avaliação calculada atual** quando preenchido.
15. **Observações:** notas livres.
16. **Campos extras do tipo:** por exemplo, emissor, carência, garantias ou subtipo.

O registro da aplicação é a **ficha-mãe**, não necessariamente um aporte. Após criá-la, entre em seu detalhe e use **+ Novo lançamento** para registrar aporte(s), resgate(s) ou juros recebidos.

### 22.2. Registrar operações na renda fixa

No detalhe, clique **+ Novo lançamento**. As opções são:

| Operação | Campos financeiros usados | Interpretação |
|---|---|---|
| `aporte` | Data, valor, observação | Aumenta o principal modelado e inicia a capitalização daquele aporte. |
| `resgate` | Data, valor, observação | Reduz o saldo modelado na data do resgate, respeitando a verificação de disponibilidade. |
| `rendimento` | Data, valor, observação | Registra juros efetivamente recebidos **separadamente**; não altera automaticamente o principal. |

Há validação de datas e valores positivos; um resgate superior ao saldo modelado disponível é bloqueado. O sistema suporta múltiplos aportes com datas distintas, permitindo que cada fluxo acumule rendimento por um período próprio.

### 22.3. Taxas e fórmulas da renda fixa

Os percentuais digitados são transformados em frações decimais. A **taxa efetiva anual presumida** segue estas fórmulas do código:

```text
Prefixado/Outro: r = taxaAnual / 100
CDI:            r = (taxaBase/100) × (percentualIndexador/100) + spread/100
Selic:          r = taxaBase/100 + spread/100
IPCA+ ou TR+:   r = (1 + taxaBase/100) × (1 + spread/100) − 1
```

A simulação do montante em uma data-alvo soma, para cada aporte/resgate, o valor capitalizado por uma **taxa anual constante**:

```text
Montante projetado(t) =
  Σ [sinal(operação) × valor(operação) × (1 + r) ^ (dias(data_operação,t) / 365)]

sinal = +1 para aporte; −1 para resgate
```

O algoritmo restringe a data-alvo ao vencimento se houver um vencimento anterior e impede que o resultado exibido seja negativo. O **principal em aberto** é a soma de aportes menos resgates considerados; o **rendimento estimado** é a diferença positiva entre saldo projetado e principal. Trata-se de uma **hipótese matemática**, sem evolução real do CDI, IPCA, TR, taxas de mercado, amortizações/cupom ou efeitos de tributação de cada fluxo.

**Cuidado especial:** para um CDB de “110% do CDI”, o cálculo utiliza uma hipótese de taxa-base anual informada manualmente; não acessa o CDI atual ou passado. Em IPCA+/TR+, o número digitado como **spread** é composto com o índice base. Na Selic, o spread é adicionado linearmente à taxa base; a opção **Outro** utiliza, no código atual, o mesmo cálculo direto de `taxaAnual` que o prefixado. Essas opções não equivalem a um motor contratual universal.

### 22.4. Valor atual, projeção e tributação

No detalhe de renda fixa, a interface exibe:

- Principal em aberto;
- Valor atual/estimado;
- Resultado sobre principal;
- Juros recebidos;
- Indexador, taxa presumida, taxas digitadas, vencimento e tributação;
- **Projeção até vencimento**, quando existe data futura;
- Principal projetado, rendimento bruto, montante bruto e líquido aproximado.

Se houver **saldo atual real informado**, o indicador de valor atual utiliza esse saldo em vez da projeção. A **projeção de vencimento**, no entanto, continua baseada nos fluxos e taxas hipotéticos. Sem vencimento futuro, não é exibida uma projeção final de vencimento.

No modo **IR regressivo**, o percentual aproximado de IR selecionado pelo código é:

| Prazo usado para a estimativa | Alíquota |
|---|---:|
| Até 180 dias | 22,5% |
| De 181 a 360 dias | 20% |
| De 361 a 720 dias | 17,5% |
| Acima de 720 dias | 15% |

O prazo é medido da **primeira operação de aporte** até o vencimento/data-alvo: não há cálculo segregado de IR por lote. Em modo **Isento**, a alíquota estimada é zero; em enquadramentos não reconhecidos, o líquido pode aparecer como **Não calculado**.

```text
Rendimento bruto estimado = max(0, montante_bruto − principal_projetado)
Valor líquido aproximado   = montante_bruto − rendimento_bruto × alíquota_IR
```

Não há apuração de IOF, taxa de custódia, imposto específico por emissor/título ou amortização periódica. O líquido é, portanto, **simulação orientativa**, e não valor de resgate garantido ou cálculo fiscal definitivo.

---

## 23. Catálogo de Renda Variável: ativos e campos

Acesse **Investimentos → Renda Variável**. Os tipos são:

| Tipo | O que controla | Campos específicos informativos |
|---|---|---|
| **Ações** | Participações em empresas; compras, vendas, cotas, custo médio, proventos. | Setor; P/L; P/VP; ROE (%); dividend yield (%). |
| **FIIs** | Fundos imobiliários; cotas, cotação, distribuições e custo médio. | Segmento/tipo de FII; P/VP; dividend yield (%); vacância (%). |
| **ETFs** | Fundos de índice listados em bolsa. | Índice de referência; gestora; taxa de administração (% a.a.). |
| **BDRs** | Recibos representativos de ativos estrangeiros. | Ativo subjacente; mercado de origem; moeda de referência. |
| **Criptomoedas** | Ativos digitais, inclusive unidades fracionárias. | Rede blockchain; plataforma/exchange; carteira/custódia; descrição de staking/rendimentos. |

Os indicadores fundamentais (P/L, P/VP, ROE, DY, vacância), taxa de administração, identificação do índice e descrições de custódia são **campos manuais informativos**. O sistema não obtém nem recalcula automaticamente esses índices a partir de demonstrações financeiras, preços oficiais ou bases externas.

### 23.1. Criar e caracterizar um ativo

1. Entre na classe **Renda Variável** e escolha o tipo.
2. Clique **+ Novo investimento** ou **+ Adicionar aplicação**.
3. Informe nome, ticker, instituição/corretora, data inicial e campos complementares do produto.
4. Se houver, informe **Cotação atual unitária (R$)** e **Data da cotação**.
5. Confirme o cadastro e abra sua ficha.
6. Registre as compras/vendas/proventos individualmente no histórico.

Uma aplicação pode possuir múltiplas compras a preços diferentes. O **preço médio** é recalculado com base nas operações existentes. É importante atualizar periodicamente a cotação manual para ter uma avaliação mais útil.

### 23.2. Operações de renda variável

| Operação | Campos | Efeito principal |
|---|---|---|
| `compra` | Data, quantidade, preço unitário, taxas e observação | Aumenta quantidade e custo da posição. |
| `venda` | Data, quantidade, preço unitário, taxas e observação | Diminui quantidade e custo pelo preço médio; registra lucro/prejuízo realizado estimado. |
| `provento` | Data, valor e observação | Registra rendimento recebido, sem aumentar automaticamente a quantidade nem o saldo da conta. |

Nas compras/vendas, o campo **Valor total** é calculado como **quantidade × preço unitário**, e as taxas são registradas em campo separado. Em ações e FIIs, “quantidade” são ações/cotas; em criptomoedas podem ser frações decimais. A formatação de quantidade admite até **10 casas decimais** na apresentação.

O sistema bloqueia uma **venda maior que a quantidade calculada como disponível**. Se for preciso alterar um lançamento, clique em **✏️** ao lado da operação na tabela do ativo; use **🗑️** para excluir uma operação após confirmação. As posições são recalculadas a partir do histórico reordenado por data.

### 23.3. Cálculo da quantidade, do custo médio e do lucro

O algoritmo processa as compras e vendas por data, inicialmente com quantidade e custo iguais a zero:

```text
Na compra:
  custo_novo = custo_anterior + quantidade_comprada × preço_compra + taxas_compra
  qtd_nova   = qtd_anterior + quantidade_comprada

Preço médio = custo_em_aberto / quantidade_em_aberto

Na venda:
  custo_removido = (custo_em_aberto / qtd_em_aberto) × quantidade_vendida
  lucro_realizado += quantidade_vendida × preço_venda − taxas_venda − custo_removido
  custo_novo = custo_anterior − custo_removido
  qtd_nova   = qtd_anterior − quantidade_vendida

Valor de mercado atual = quantidade_em_aberto × cotação_unitária_informada
Variação não realizada = quantidade_em_aberto × (cotação_unitária_informada − preço_médio)
```

Se a posição ficar praticamente zerada, o motor redefine quantidade e custo como zero. O custo médio é um **custo médio ponderado em carteira**, não a média aritmética simples dos preços de compras. Vendas removem custo pelo médio anterior; outras convenções e eventos corporativos não são simulados automaticamente.

**Sem cotação informada:** para o `valor` do ativo, o código usa o **custo remanescente** como aproximação. Mas a propriedade interna `resultado` de renda variável continua sendo calculada a partir da cotação numérica, que vira zero se estiver ausente. Assim, **a variação exibida pode ficar incorretamente negativa quando a cotação não é informada**. Informe uma cotação válida e sua data; não interprete essa variação como desempenho real quando estiver faltando cotação.

### 23.4. Proventos, valorização e resultado

São conceitos distintos:

- **Proventos**: operações registradas com tipo `provento`; mostram dinheiro recebido, mas não são reinvestidas automaticamente nem somadas à posição atual.
- **Valorização/desvalorização não realizada**: diferença entre cotação manual e preço médio aplicada à quantidade atual.
- **Lucro/prejuízo realizado**: resultado das operações de venda, considerando custo médio proporcional e taxas de venda.
- **Valor atual/custo**: quantidade atual × cotação registrada ou, na falta dela, custo remanescente.

O quadro de carteira soma `resultado` de todas as aplicações e `realizado` das vendas, enquanto os proventos aparecem em linha própria. **Não confunda “total histórico aportado” com “custo em aberto”**: o primeiro acumula aportes brutos e compras feitas ao longo do tempo; o segundo reflete a posição remanescente depois de resgates e vendas.

### 23.5. Restrições do modelo de renda variável

Não há consulta automática à B3, bolsas internacionais, exchanges ou corretoras; não há séries históricas de cotações reais, cálculo de split/grupamento, bonificações, conversões de moeda ou tratamento tributário de vendas. ETFs e BDRs usam o mesmo motor de quantidades/preço médio das ações. O preço cadastrado deve estar **em reais** no campo de cotação, inclusive quando o ativo tem exposição externa. O campo de staking das criptomoedas é **descritivo**, não um cálculo autônomo de recompensas em unidades.

---

## 24. Histórico mensal dos investimentos e agenda de vencimentos

### 24.1. Gráfico de evolução do patrimônio investido

Na tela principal de Investimentos, existe um **SVG com linha de patrimônio mensal**, além de um seletor:

- últimos **3 meses**;
- últimos **6 meses**;
- últimos **12 meses**;
- **Personalizado**, com data inicial e data final.

O histórico considera até **60 meses**, sendo calculado a partir dos registros presentes. Em cada mês, a função `invSerieHistorica()`:

1. identifica a data-alvo (último dia do mês, ou o limite selecionado);
2. avalia a renda fixa com aportes, resgates e taxa presumida até essa data;
3. avalia a renda variável sobre as quantidades adquiridas até a data e a cotação manual mais recente, quando disponível;
4. soma as duas classes para desenhar a linha.

**Limite crucial:** no segmento variável, isso **não é uma curva de preços históricos**. A mesma cotação manual atual pode ser aplicada às quantidades de vários meses. A inclinação do gráfico pode refletir sobretudo aportes, compras e vendas, e não uma valorização passada de mercado.

### 24.2. Drill-down por mês

Cada ponto desenhado no gráfico é clicável: abre **Movimentações: AAAA-MM**, contendo quantidade de operações, somatório de aportes/compras, somatório de vendas/resgates e a relação dos lançamentos. Clique no ativo de uma linha para consultar ou editar as operações na sua própria ficha.

### 24.3. Agenda de vencimentos

Se houver renda fixa com data de vencimento igual ou posterior ao dia atual, o módulo inclui **Próximos vencimentos de renda fixa**, com vencimento, nome e **projeção bruta de resgate**. A lista exibe até **15 aplicações** ordenadas pela data de vencimento. A previsão usa a taxa modelada, não um evento de crédito automático no caixa. Clicar abre a aplicação.

### 24.4. Histórico de rendimentos

O card **Rendimentos / proventos registrados** leva à visão consolidada de operações `provento` e `rendimento`, com data, ativo, tipo, valor e acesso ao detalhe. **Esses registros não criam automaticamente receitas no módulo Recebimentos** e não entram nos saldos de contas.

---

## 25. Tipos personalizados de investimento

Os dois catálogos disponibilizam **+ Cadastrar outro tipo** ou **+ Novo tipo**. O formulário aceita:

- **Classe** (`fixa` ou `variavel`);
- **Nome** do novo tipo;
- **Descrição** de como funciona;
- **Campos adicionais**, separados por vírgula, ponto e vírgula ou quebra de linha.

São permitidos até **15 nomes de campos adicionais** por tipo. Há bloqueio de nomes de tipo duplicados na mesma classe, ignorando maiúsculas/minúsculas. Ao salvar, o catálogo passa a exibir o novo tipo na classe escolhida. O cadastro é persistido em `dados.tiposInvestimento[]` e as informações particulares de cada aplicação ficam em `investimento.extras`.

**O que é implementado:** criação de tipo com descrição e rótulos de campos, para organização, entrada manual e leitura no detalhe. **O que não é implementado:** criação de uma fórmula financeira específica por tipo, regras tributárias próprias programáveis, escolha livre de componente de formulário por campo, exclusão/edição administrativa dos tipos personalizados pelo catálogo ou integração com provedores externos. O motor financeiro continua sendo o genérico da classe fixa ou variável.

---

## 26. Transferências: circulação entre contas e dinheiro físico

O módulo **🔄 Transferências** destina-se exclusivamente a **movimentos internos**. Casos típicos: banco A → banco B; dinheiro físico → banco; banco → dinheiro físico. Ao contrário dos investimentos, **transferências alteram efetivamente os saldos registrados** das contas e do dinheiro em mãos.

### 26.1. Cadastro de transferência

1. Clique em **Transferências → + Nova transferência**; alternativamente, abra o detalhe de uma conta e use **+ Transferir**.
2. Selecione **Origem (debitar)** entre dinheiro físico e contas cadastradas.
3. Selecione **Destino (creditar)**, diferente da origem.
4. Informe valor positivo, data e, opcionalmente, uma descrição.
5. Salve e confira as duas disponibilidades atualizadas.

O sistema verifica se as duas contas existem, se são diferentes e se o valor não ultrapassa o saldo calculado como disponível na origem. Não há taxa de transferência automática nem conversão de moeda.

### 26.2. Fórmula financeira e invariância do total

Seja `T` o valor transferido:

```text
saldo_origem_novo  = saldo_origem_antigo − T
saldo_destino_novo = saldo_destino_antigo + T

Δ(soma de saldos de contas + dinheiro físico) = −T + T = 0
```

Os saldos são arredondados para duas casas decimais ao serem ajustados. A transferência é **neutra na soma de dinheiro físico + saldos de contas**, mas muda a distribuição entre fontes. Ela **não cria linhas em `despesas[]` nem em `recebimentos[]`**; portanto, não altera os KPIs de receita/despesa por si só.

### 26.3. Dashboard e histórico

O painel traz:

- **Dinheiro físico** (saldo atual);
- **Saldo nas contas** (soma atual dos bancos);
- **Total disponível** (dinheiro + contas);
- **Transferido no período** (soma bruta de transferências com data no mês/dia selecionado).

A tabela histórica mostra **Data, Origem, Destino, Valor, Observações e ações de edição/exclusão**. O indicador “Transferido no período” **não representa lucro, renda nem crescimento patrimonial**, apenas volume movimentado internamente; transferir o mesmo dinheiro várias vezes faz esse volume crescer.

No detalhe individual da conta (incluindo dinheiro físico), existe também uma tabela **Transferências desta conta no período**, com data, direção da movimentação, contraparte, valor e acesso à edição. Essa listagem usa o filtro global de período, enquanto os saldos apresentados continuam sendo os saldos atuais.

### 26.4. Editar transferência e reverter efeitos

Ao editar uma transferência já registrada, o algoritmo opera nesta ordem:

1. recupera e valida a transferência anterior;
2. calcula se há saldo suficiente para **desfazer** a operação anterior, sem gerar saldo negativo;
3. devolve o valor original à origem anterior e o remove do destino anterior;
4. debita a nova origem e credita o novo destino;
5. atualiza o registro com data, valor, contas e observação;
6. recalcula a interface.

Isso permite corrigir valor, data e contas sem duplicar o efeito. **A edição pode ser bloqueada** quando o saldo atual do destino original não permite desfazer a transferência anterior.

### 26.5. Excluir transferência

O botão **🗑️** pede confirmação. A exclusão tenta **reverter os saldos** antes de retirar o item do histórico: devolve dinheiro à origem e debita o destino. Caso o destino não possua saldo suficiente, a reversão é recusada e o registro é mantido.

**Regra de integridade:** transferências trabalham sobre **saldos atuais**, não sobre um *ledger* histórico capaz de reconstruir saldos por data. Alterar manualmente saldos em fonte externa ou mover recursos após uma transferência antiga pode tornar impossível revertê-la até que haja saldo compatível.

---

## 27. Dados e entidades acrescentados na V17

### 27.1. Investimento

```text
investimentos[]
  id, classe (fixa|variavel), tipo, nome, codigo, instituicao,
  dataInicio, vencimento, indexador, taxaAnual, taxaBase,
  percentualIndexador, spread, tributacao, saldoAtual,
  cotacaoAtual, cotacaoData, extras{}, notas, movimentos[]

movimentos[]
  id, data, tipo, valor, quantidade, preco, taxas, nota
```

As operações reconhecidas são `aporte`, `resgate`, `rendimento` na renda fixa e `compra`, `venda`, `provento` na renda variável. O campo `classe` de uma aplicação **não pode ser trocado** depois que ela possui operações, evitando reinterpretar aquisições de cotas como aplicações de renda fixa ou vice-versa.

### 27.2. Tipos personalizados

```text
tiposInvestimento[]
  id, classe, nome, descricao, campos[]
```

`garantirModulos()` inicializa arrays ausentes, prepara `movimentos[]` e `extras{}` de registros antigos, preservando compatibilidade básica entre arquivos.

### 27.3. Transferência

```text
transferencias[]
  id, data, origem, destino, valor, nota

Identificadores de conta:
  fisico            → dinheiro físico
  c:<nome da conta> → conta bancária
```

A vinculação da transferência à conta usa **nome de conta dentro da chave**, e não um ID imutável de conta. Renomeações da entidade sem migração dos registros podem impedir localização e reversão.

### 27.4. Estado de demonstração

A V17 de demonstração possui uma rotina `adicionarDemonstracaoInvestV17()` invocada durante `boot()`. Ela é condicionada aos marcadores `zerado` e `exemplosInvestV17`, evita IDs duplicados de exemplo e identifica aplicações de demonstração por `exemploInvest`. Esse comportamento explica o aparecimento de dados ilustrativos na distribuição e nos gráficos quando o arquivo é aberto.

**Esta documentação descreve somente a arquitetura, os campos e a lógica. Não utiliza taxas, preços, nomes de posições, cotações nem resultados simulados para afirmar qualquer rentabilidade real.** O estado de demonstração exige atenção antes de começar um acompanhamento financeiro verdadeiro: não confunda posições ilustrativas com patrimônio real.

---

## 28. Mapa técnico: funções novas da V17

| Função | Papel |
|---|---|
| `garantirModulos()` | Cria/prepara coleções adicionais e estruturas internas faltantes. |
| `tiposDeInv()`, `tipoDef()` | Recupera catálogo fixo e tipos personalizados da classe. |
| `renderInvestimentos()` | Painel consolidado, distribuição, histórico e vencimentos. |
| `renderInvestClasse()`, `renderInvestTipo()`, `renderInvestLista()` | Catálogos de classe/tipo e listagens da carteira. |
| `renderDetalheInvestimento()` | Ficha, características, métricas, projeções e histórico do ativo. |
| `abrirInvestimento()`, `popularTiposInv()`, `renderInvExtras()`, `salvarInvestimento()` | Formulário e gravação do cadastro principal. |
| `abrirNovoTipo()`, `salvarNovoTipo()` | Tipo próprio, descrição e campos adicionais. |
| `abrirMovInvest()`, `atualizarCamposMovInvest()`, `calcularValorMovInvest()`, `salvarMovInvest()` | Formulário de operações de investimento e sua validação. |
| `excluirMovInvest()`, `excluirInvestimento()` | Exclusão confirmada de operação ou aplicação com histórico. |
| `taxaEfetivaInv()` | Converte campos de taxa/indexador em taxa anual de projeção. |
| `aplicarFluxosFixos()` | Capitaliza aportes e resgates de renda fixa até a data-alvo. |
| `invAnalise()` | Avalia aplicação, principal/custo, resultado, preço médio e proventos. |
| `resumoCarteira()` | Consolida posições, montantes, custo, classes e proventos. |
| `invSerieHistorica()` | Gera valores mensais estimados da carteira. |
| `renderHistoricoInvestimentos()`, `renderInvestMes()` | SVG da evolução e drill-down mensal. |
| `renderInvestProventos()`, `renderVencimentosInvestimentos()` | Listagens de rendimentos e vencimentos. |
| `contasTransferencia()`, `lerSaldo()`, `definirSaldo()`, `mudarSaldoTransfer()` | Identificação e movimentação de saldos físicos/bancários. |
| `renderTransferencias()`, `abrirTransferencia()`, `salvarTransferencia()`, `excluirTransferencia()` | Painel, edição, verificação e reversão de transferências. |
| `transferenciasDaConta()`, `tabelaTransferenciasConta()` | Extrato das transferências internas de cada conta. |
| `adicionarDemonstracaoInvestV17()` | Inicialização controlada dos registros demonstrativos. |

---

## 29. Limitações adicionais e cuidados de uso da V17

1. **Investimento separado de saldo:** operação de investimento não debita/credita automaticamente contas. Isso evita efeitos inesperados, mas exige conciliação manual de recursos.
2. **Transferência não é rendimento:** ela apenas redistribui os saldos atuais e não afeta receitas/despesas.
3. **Renda fixa é projeção simplificada:** taxa anual presumida constante; não estima curva efetiva histórica/futura dos indexadores, cupons ou marcação a mercado.
4. **Renda variável requer cotação manual válida:** sem ela, o valor pode usar o custo, mas a variação interna pode tornar-se incoerente.
5. **Histórico patrimonial variável não usa preços históricos de mercado:** utiliza quantidade por data com a cotação mais recente.
6. **Taxas e impostos não são completos:** a calculadora de renda fixa usa tabela regressiva aproximada e não substitui extratos, escrituração ou declaração fiscal.
7. **Proventos não são reinvestidos nem creditados em conta automaticamente.**
8. **Sem conectores externos de corretora, bancos ou bolsa:** preços, vencimentos, indexadores e métricas fundamentais dependem do que o usuário digita.
9. **Transferências antigas podem não ser reversíveis** se o destino já consumiu o saldo recebido ou se a conta foi removida/renomeada.
10. **Campos personalizados são metadados**, não scripts de novas fórmulas de avaliação.
11. **O HTML de demonstração inclui rotina de população automática**, identificada por marcadores; não trate seus ativos de exemplo como dados financeiros reais.
12. **Salvar Arquivo é a forma de persistir estado no documento**; o botão Importar JSON permanece com a limitação documentada no capítulo 2.
13. **Zerar painel na V17** limpa também `investimentos[]`, `transferencias[]` e `tiposInvestimento[]`; deve ser testado apenas em cópia descartável.

---

## 30. Guia de verificação específico da V17

Além do roteiro do capítulo 19:

- [ ] Confirmar presença dos dois cards novos no menu lateral.
- [ ] Consultar as duas classes e as descrições de todos os tipos pré-cadastrados.
- [ ] Criar um tipo personalizado e uma aplicação desse tipo.
- [ ] Registrar aportes de renda fixa em datas diferentes e conferir a projeção de vencimento.
- [ ] Alternar entre taxa prefixada, CDI, Selic, IPCA+ e TR+ utilizando hipóteses explícitas.
- [ ] Informar/remover o saldo atual manual e notar a diferença na avaliação.
- [ ] Criar ativo de renda variável, registrar compras com preços e taxas distintas e verificar preço médio.
- [ ] Registrar uma venda parcial e conferir quantidade, custo e lucro realizado.
- [ ] Registrar proventos/juros e confirmar que aparecem no histórico sem aumentar automaticamente saldos bancários.
- [ ] Atualizar a cotação manual e conferir valor de mercado, variação e data da última cotação.
- [ ] Alternar histórico de investimentos entre 3, 6, 12 meses e personalizado; clicar num ponto do gráfico.
- [ ] Conferir a lista de vencimentos de renda fixa e suas projeções.
- [ ] Criar uma transferência entre contas, conferir débito/crédito e invariância do total.
- [ ] Fazer depósito de dinheiro físico em conta e confirmar a alteração nos dois saldos.
- [ ] Editar e excluir a transferência, conferindo reversão e tratamento de saldo insuficiente.
- [ ] Reabrir o HTML salvo e verificar investimentos, operações, tipos próprios e transferências.
- [ ] Distinguir os dados demonstrativos de qualquer cadastro real; antes de usar em produção pessoal, trabalhar numa cópia de arquivo preparada para isso.

---

## 31. Conclusão arquitetural da V17

A V17 mantém a base de **movimentação financeira e saldos** da V15 e acrescenta dois subdomínios: **carteira de investimentos com avaliação separada** e **transferências internas que preservam a soma dos saldos disponíveis**. O mesmo sistema visual é reaproveitado: módulos em cards, tabelas navegáveis, formulários locais, métricas clicáveis e salvamento autocontido em HTML.

A arquitetura atende ao acompanhamento pessoal de registros manuais e à exploração de métricas sob premissas declaradas. Ela não equivale a um sistema bancário integrado, custodiante, *market-data terminal*, razão contábil auditado ou calculadora fiscal oficial. A distinção entre **valor inserido**, **valor calculado**, **valor estimado** e **valor de mercado manual** é essencial para interpretar os relatórios corretamente.
