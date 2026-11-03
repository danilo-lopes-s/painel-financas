# Painel Financeiro Pessoal — Manual integral, especificação funcional e documentação técnica

**Edição documental:** 2026-11-03 · **Base de referência:** última entrega com backup JSON e painel limpo · **Idioma:** pt-BR · **Moeda:** BRL.  
**Arquivo principal examinado:** `Painel_Financeiro_Backup_Funcional.html`.  
**Arquivo de instalação limpa examinado:** `Painel_Financeiro_Em_Branco_Importar.html`.  
**Documento anterior utilizado como histórico de referência:** `Documentacao_Completa_Painel_Financeiro_V17.md`.  
**Natureza:** documentação funcional, guia do usuário, dicionário de dados, contrato de interoperabilidade, referência de manutenção e roteiro de testes.

> **Status da documentação:** baseada na inspeção dos HTMLs efetivamente disponíveis, não apenas nas solicitações anteriores. Não pressupõe funcionalidades que não estejam identificadas no arquivo atual. A documentação não inclui saldos, números bancários, nomes de pessoas ou valores privados presentes nos registros incorporados.

## 0. Controle documental e rastreabilidade

| Campo | Conteúdo |
|---|---|
| Aplicação | Painel Financeiro Pessoal, executado localmente como HTML autocontido |
| Base funcional | HTML com dados incorporados e novas operações de backup/exportação/importação JSON |
| Instalação limpa | HTML com coleções financeiras vazias e indicadores de inicialização que evitam dados demonstrativos |
| Contrato de backup atual | `formato: "PainelFinanceiro"`, `versaoEstrutura: 1` |
| Biblioteca de dados | Objeto JavaScript global `dados`; dados carregados do `window.INJETADO` |
| Persistência | Salvar HTML para gravar o estado; exportar JSON para criar backup portável |
| Dependência de backend | Nenhuma evidenciada no arquivo analisado |
| Fonte da verdade | Código-fonte da entrega atual, não documentação antiga ou descrição verbal |
| Nível de verificação | Análise estática de estrutura, fluxos, constantes e funções; verificações pontuais de integridade descritas ao final |

**Identidade dos arquivos analisados (SHA-256):**

```text
eba4240d022b68870ed2c37282cbdb6a42351e4abe8b4b6ae91ec22d3472980e  Painel_Financeiro_Backup_Funcional.html
71718948e89ce6ef861e1e999e97e1f66576b6c44487f23eb1007e4ae3cb8594  Painel_Financeiro_Em_Branco_Importar.html
```

Se um dos HTMLs for modificado futuramente, esses identificadores também mudarão. Portanto, este manual descreve precisamente **esta entrega**. Os exemplos JSON e financeiros a seguir são fictícios e servem apenas para explicar os contratos.

## Sumário

1. Finalidade e escopo
2. Arquitetura e princípios de persistência
3. Preparação e primeiro acesso
4. Mapa de navegação e drill-down
5. Filtros de período e datas
6. Visão Geral e indicadores
7. Calendário e agrupamentos
8. Contas e dinheiro físico
9. Cartões e Parcelamentos
10. Despesas e débitos automáticos
11. Recebimentos e créditos
12. Metas a Receber
13. Recorrências e parcelamentos: diferenças
14. Categorias e ícones
15. Investimentos: visão de conjunto
16. Investimentos: renda fixa
17. Investimentos: renda variável
18. Investimentos: operações, históricos e tipos personalizados
19. Transferências entre contas
20. Lixeira e recuperação de registros
21. Status e lógica financeira de saldos
22. Fórmulas, métricas e limitações dos indicadores
23. Menus retráteis, gráficos e navegação de detalhe
24. Salvar o HTML com dados incorporados
25. Exportar backup JSON
26. Importar backup JSON e restaurar painel limpo
27. Compatibilidade, migração e versionamento de dados
28. Reversão de importação, zeramento e recuperação
29. Modelo de dados e dicionário de coleções
30. Relacionamentos, IDs e integridade referencial
31. Arquitetura de código, funções e elementos da interface
32. Procedimentos operacionais completos
33. Segurança, privacidade e limites de proteção
34. Manutenção e evolução segura do painel
35. Testes e critérios de aceite
36. Lacunas comprovadas e pontos de atenção
37. Glossário e convenções
38. Apêndices técnicos: inventário de funções e interface

---

## 1. Finalidade e escopo

O painel concentra acompanhamento financeiro pessoal em uma aplicação de arquivo único. O usuário pode cadastrar contas, cartões, despesas, recebimentos, metas, parcelamentos, categorias, investimentos, movimentações financeiras e transferências; consultar gráficos; navegar por registros; mover informações para a lixeira; e conservar o estado por meio de HTML autocontido.

A entrega acrescenta **portabilidade integral dos dados**. É possível exportar o objeto financeiro para JSON e importá-lo em outro HTML compatível, inclusive na versão sem registros, sem redigitar lançamentos. O layout, os gráficos e as telas pertencem ao código da aplicação; os registros e seus relacionamentos pertencem ao modelo de dados.

**Fora do escopo evidenciado:** integração direta com bancos, sincronização entre dispositivos, autenticação, banco de dados em servidor, atualização automática pela internet, serviço de cotação ao vivo, criptografia do JSON e conversão genérica de quaisquer formatos futuros. Não confundir estimativas financeiras produzidas pelo código com extratos bancários ou recomendações de investimento.

### 1.1. Princípios funcionais

1. O HTML atua como aplicativo e, quando salvo, como fotografia persistida dos dados.
2. As operações realizadas na tela alteram primeiramente o estado carregado em memória.
3. **Salvar Arquivo** incorpora o estado atual ao HTML gravado.
4. **Exportar dados** produz um JSON independente da interface.
5. **Importar dados** substitui o conjunto de registros em memória depois de conferência e confirmação.
6. O sistema conserva relacionamentos por identificadores e referências armazenados nos próprios registros.
7. O filtro de período altera as visualizações e cálculos periódicos, não reescreve retroativamente os saldos atuais.
8. Investimentos, contas e despesas representam conceitos financeiros distintos: não somar indiscriminadamente cada visualização como se fossem movimentações de caixa separadas.

## 2. Arquitetura e persistência

### 2.1. Arquivo único

O HTML inclui CSS, a estrutura da interface, janelas modais, dados incorporados e código JavaScript. Os dados são injetados em um script identificável:

```html
<script id="dados_injetados">
  window.INJETADO = JSON.parse("...");
</script>
```

No carregamento, `boot()` copia o objeto de `window.INJETADO` para `dados`; executa rotinas de compatibilidade, garante as coleções necessárias, restaura a ordem de navegação e renderiza a interface. O bloco de dados é substituído pela função de salvamento quando se grava uma nova versão do HTML.

### 2.2. Independência do navegador

Na entrega atual, `salvarDadosLocal(isBoot)` **não grava dados financeiros em `localStorage`**; apenas sinaliza alterações pendentes, exceto no carregamento inicial. A função `boot()` usa os registros incorporados no próprio HTML e não procura o antigo `painelFinanceiro_v9` para restaurar dados.

Isso impede que o zeramento de uma cópia do painel afete outra versão por compartilhamento de uma chave de armazenamento. Em contrapartida, alterações em uma cópia **não se propagam** automaticamente às outras. Para migrá-las, use JSON ou copie o HTML salvo.

### 2.3. Estado temporário e estado persistido

| Estado | Onde existe | Quando muda | Sobrevive ao fechamento? |
|---|---|---|---|
| Registros em memória | Objeto `dados` na aba aberta | Ao criar, editar, importar, restaurar ou excluir | Não, a menos que se salve o HTML |
| Dados incorporados | Script `dados_injetados` do arquivo no disco | Ao gravar um novo HTML | Sim |
| Backup JSON | Arquivo `.json` exportado | No instante da exportação | Sim, enquanto o arquivo for preservado |
| Navegação aberta/filtros | Variáveis de interface e DOM | Cliques, seleção de período | Não é um registro financeiro |
| Tema visual | Classe `dark` no elemento do documento | Alternância de tema | A aparência pode viajar com o HTML salvo |
| Desfazer importação | Snapshot em memória | Após importação confirmada | Não; recurso da sessão atual |

### 2.4. Estrutura de inicialização

- `garantirModulos()` assegura coleções necessárias.
- `garantirContaDinheiroPrincipal()` assegura a conta técnica que representa dinheiro físico e sincroniza seu saldo com o campo legado `dinheiroFisico`.
- `migrarCreditoRecebimentos()` e sinalizadores associados protegem transições entre formatos anteriores e a lógica de crédito.
- Sinalizadores `zerado`, `exemplosV13`, `exemplosInvestV17` e `demoContasV18` controlam a inserção de exemplos nas condições previstas pelo código.
- O HTML limpo fornece os sinalizadores corretos para **não preencher automaticamente dados demonstrativos** ao ser aberto.

**Nota operacional:** mesmo no painel limpo, pode existir **uma conta técnica de dinheiro físico, com saldo zero**, criada no início. Isso não significa que registros financeiros antigos tenham sido importados.

## 3. Preparação e primeiro acesso

### 3.1. Requisitos

- Navegador moderno com JavaScript habilitado.
- Permissão para abrir arquivos locais e descarregar arquivos gerados.
- Espaço para guardar o HTML e os backups JSON.
- Preferência por diretório de backup organizado, com cópias datadas e separadas.

O navegador pode oferecer uma caixa para escolher a localização ao usar **Salvar Arquivo**; em outros casos, o aplicativo realiza download de um HTML atualizado. Essas diferenças dependem do ambiente.

### 3.2. Sequência recomendada

1. Abra `Painel_Financeiro_Backup_Funcional.html`.
2. Confira os indicadores e os módulos.
3. Use **Exportar dados** para criar uma cópia JSON inicial.
4. Após cadastrar ou editar informações, confirme o indicador de alterações não salvas.
5. Clique em **Salvar Arquivo** e confira o arquivo efetivamente gravado ou baixado.
6. Na sessão seguinte, abra o arquivo **mais recente que foi salvo**, não uma cópia antiga inadvertidamente.

### 3.3. Diferença entre versões fornecidas

**Preenchida:** contém os registros financeiros já incorporados antes da inclusão do mecanismo de backup. **Limpa:** traz as coleções sem cadastros financeiros e está preparada para receber um JSON completo. Ambas contêm interface, cálculos e rotinas de importação/exportação equivalentes no escopo funcional desta entrega.

## 4. Navegação e drill-down

### 4.1. Módulos de navegação principal

| Card | Entrada principal | Subníveis comuns |
|---|---|---|
| Visão Geral | Indicadores e distribuição | Detalhes de indicador e categoria |
| Calendário | Mês e eventos | Detalhes de lançamento |
| Contas e Transferências | Lista de contas e resumos | Tipo de conta, conta individual, Transferências |
| Cartões e Parcelamentos | Áreas Cartões, Parcelamentos e Despesas únicas | Cartão individual, parcela, lançamento |
| Despesas | Listagem e filtros | Detalhe da despesa, edição |
| Metas a Receber | Metas previstas | Detalhe da meta e confirmação |
| Recebimentos | Entradas e respectivos estados | Detalhe do recebimento |
| Categorias | Cadastros e atributos | Criação, edição, exclusão |
| Investimentos | Patrimônio, classes e tipos | Renda fixa, variável, aplicação, operações e histórico |
| Lixeira | Registros excluídos logicamente | Restauração, exclusão definitiva |

### 4.2. Drill-down

Uma seleção secundária chama `selecionarVisao(view,param)` e a tela aberta é registrada em `historicoVisoes` quando elegível. `navegarVoltar()` recupera o destino anterior ou usa um destino de segurança definido por `destinosVoltarPadrao` quando não houver histórico interno.

Os detalhes têm **← Voltar** na barra do cabeçalho e uma barra contextual dentro do conteúdo, injetada por `injetarVoltarDrilldown()`. Assim, o usuário não depende do histórico do navegador nem precisa retornar ao topo manualmente.

### 4.3. Menus compactos e expansíveis

As áreas volumosas usam blocos retráteis. `expBloco()` cria um painel com botão `+` ou `−`; `expLinhaDe()` faz o mesmo com linhas de tabelas. O painel também possui recurso **Expandir tudo / Contrair tudo** em páginas que registram blocos expansíveis.

O sistema diferencia: **abrir uma conta/cartão** pelo item clicável (nova página interna) e **expandir detalhes de um lançamento** dentro da tabela. O estado de expansão pode ser preservado em redesenhos durante a sessão, mas não integra o contrato financeiro do backup JSON.

### 4.4. Ordem dos cards

A navegação lateral possui arraste para reordenar. A ordem fica em `dados.navOrder`, portanto pode acompanhar o JSON exportado e o HTML salvo. A ordenação dos registros em algumas tabelas possui controles próprios e não deve ser confundida com a ordem dos cards.

## 5. Períodos, datas e filtros

O filtro global utiliza `globalAno`, `globalMes`, `globalDia` e `modoPeriodo`, com modos **Mês/ano** e **Dia**. `emPeriodo(registro)` compara o campo `data` do registro ao dia exato ou ao prefixo `AAAA-MM` do mês selecionado. Os controles permitem avançar/voltar no período e retornar a Hoje.

**Convenções:** armazenamento de datas em `AAAA-MM-DD`; exibição normalmente em `DD/MM/AAAA`; valores monetários em reais com `Intl.NumberFormat('pt-BR', {style:'currency',currency:'BRL'})`.

O calendário trabalha com visão mensal própria, mesmo quando há seleção pontual por dia. Investimentos e categorias podem ocultar o seletor de período global quando utilizam seus próprios critérios. Os saldos atuais das contas não são automaticamente recalculados para representar o patrimônio exato do último dia de cada período.

### 5.1. Filtros de tabelas

Despesas e recebimentos possuem filtros de texto/categoria e opções de ordenação na interface. Parcelamentos possuem ordenação de colunas e comando para restaurar a ordem original. As métricas e os contadores de telas filtradas devem ser interpretados **na seleção em vigor**.

## 6. Visão Geral

É a página principal de consulta. Apresenta, conforme o período selecionado, somas de receitas/recebimentos, despesas, diferença entre entradas e saídas e metas planejadas. O gráfico de distribuição de gastos consolida despesas por categoria, e os componentes gráficos permitem abrir a lista dos itens que compõem cada resultado.

**Atenção:** somatórios históricos por período não equivalem necessariamente a entradas/saídas bancárias já liquidadas. Vários relatórios somam `valor` das coleções por `data` sem exigir que todos os registros estejam efetivamente pagos. É o **status**, combinado às regras específicas de crédito/débito, que controla o efeito no saldo das contas em determinados fluxos.

### 6.1. Indicadores agregados

- Despesas no período: `sum(dados.despesas.filter(emPeriodo))`.
- Recebimentos no período: `sum(dados.recebimentos.filter(emPeriodo))`.
- Resultado do período: recebimentos menos despesas selecionados.
- Metas: valores dos planejamentos elegíveis ao período.
- Composição do saldo líquido disponível: abordada no capítulo 22.

## 7. Calendário

O calendário organiza eventos por data, diferenciando despesas, recebimentos e metas. Oferece exibição por **valor** ou **título** do lançamento; os eventos abrem detalhamentos e são relacionados às coleções originais.

Para despesas em cartão, é a **data de vencimento da cobrança/parcela registrada** que determina o período da fatura exibida pelo código. A data da compra, quando informada, permanece como atributo adicional para consulta e ordenação em contextos adequados.

A função `renderCalendario()` é a referência principal de implementação; mudanças na forma de agrupar eventos devem preservar os IDs dos lançamentos e a distinção entre compra original, ocorrência recorrente e parcela.

## 8. Contas e dinheiro físico

### 8.1. Classes suportadas

| Tipo normalizado | Papel | Observações |
|---|---|---|
| `corrente` | Conta corrente | Saldo pode sofrer transferências, despesas debitadas e recebimentos creditados |
| `poupança` | Conta de poupança | Há lançamentos manuais de depósito, rendimento e saque |
| `previdência` | Conta de previdência | Há aportes, rendimentos e resgates; patrimônio separado do saldo líquido disponível |
| `investimento` | Conta vinculável a aplicações | Valor pode considerar saldo de caixa mais patrimônio das aplicações relacionadas |
| `dinheiro` | Dinheiro físico ou reserva em espécie | Opera por saldo próprio e pode participar de transferências e lançamentos |

`tipoConta()` normaliza nomes de tipos para essas classes. O cadastro registra nome, modalidade, saldo e, quando informados, instituição, agência, número e observações.

### 8.2. Listagem e acesso

Em **Contas e Transferências → Contas**, cada conta possui uma linha clicável com ícone, nome, modalidade/instituição e saldo correspondente. Não se deve usar o símbolo de expansão dos lançamentos para entrar na conta: o clique na própria linha abre `detalhe_conta`.

A tela geral exibe **saldo digital disponível**, **dinheiro físico**, **total disponível** e **patrimônio total em contas**, além de atalhos por tipo e gráficos gerais recolhidos. O gráfico total pode incorporar categorias patrimoniais que **não fazem parte do caixa imediatamente disponível**.

### 8.3. Detalhe individual

A página individual inclui controles para registrar novos recebimentos/despesas, transferir, editar e, quando permitido, excluir a conta. Os indicadores dependem da modalidade. Nas contas aplicáveis, **Entradas no período (n)** e **Saídas no período (n)** mostram valores e contagens dos registros relacionados ao período.

Abaixo ficam três blocos de consulta, inicialmente fechados:

1. **Lançamentos da conta (n)** ou **Aplicações da conta (n)** para contas de investimento;
2. **Transferências desta conta no período (n)**;
3. **Gráficos e desempenho**, com evolução, fluxo e distribuições pertinentes à modalidade.

Os ícones específicos de tipos de contas são parte da interface vigente e devem ser preservados durante futuras mudanças de layout.

### 8.4. Dinheiro físico e compatibilidade

A conta técnica identificada por `sistema:'dinheiroFisico'` representa o dinheiro em espécie legado. `garantirContaDinheiroPrincipal()` garante sua existência e sincroniza `dados.dinheiroFisico` com o respectivo saldo. A conta principal possui proteções especiais contra exclusão comum, pois pode ser referenciada por lançamentos antigos.

### 8.5. Lançamentos manuais de poupança e previdência

`dados.movContas` armazena movimentos de **depósito, rendimento e saque** (poupança) e **aporte, rendimento e resgate** (previdência). As operações podem ser criadas, editadas ou excluídas. Os saldos, capital aportado líquido e rendimentos exibidos decorrem das regras da modalidade e das movimentações registradas. O modelo admite rendimento negativo na opção correspondente.

## 9. Cartões e Parcelamentos

### 9.1. Menu do módulo

A entrada **Cartões e Parcelamentos** agrupa separadamente:

- **Cartões:** seleção de cartões cadastrados;
- **Parcelamentos:** compras em várias parcelas, agregadas por identificador;
- **Despesas únicas nesta fatura:** cobranças de cartão que não pertencem a um parcelamento.

A lista de cartões é propositalmente compacta: cada item é clicável e exibe **nome** e **identificação** (instituição/bandeira/final, quando disponíveis), além do ícone. Os indicadores de limite e fatura ficam **dentro do drill-down do cartão**, não na listagem externa.

### 9.2. Detalhe individual do cartão

Ao entrar em `detalhe_cartao`, o painel apresenta: instituição, bandeira, final, vencimento, fechamento, valor e percentual do limite disponível; limite total, limite usado, limite livre, valor dos lançamentos do período; valor da fatura do mês, parcelas futuras pendentes e quantidade de lançamentos; comandos de cadastro e edição; **Lançamentos do cartão (n)** e gráficos em menus retráteis.

O botão **← Voltar** retorna à navegação do módulo. Itens pagos ou cancelados são excluídos da composição de uso de limite em funções específicas; entretanto, o rótulo **fatura do mês** possui uma fórmula diferente, explicitada abaixo.

### 9.3. Fórmulas de limites

Sejam `L = limite cadastrado`, `F = soma de despesas do cartão no mês selecionado`, `P = soma de cobranças do mês não pagas/não canceladas` e `R = soma de parcelas futuras ainda não pagas`:

```text
faturaCartao(c) = F                        # soma nominal das cobranças do mês
reservaFuturaCartao(c) = R                 # parcelas futuras pendentes
usadoLimiteCartao(c) = P + R               # limite comprometido
limiteLivre = L - usadoLimiteCartao(c)
percentualLivre = limiteLivre / L × 100    # quando L > 0
```

O cálculo de `faturaCartao(c)` mostrado no código soma as despesas do cartão no mês **independentemente do status**; o compromisso de limite é mais restritivo porque utiliza o predicado `pendenteCartao`. Portanto **fatura nominal do período** não deve ser confundida com **valor pendente a quitar**. Essa distinção é importante para documentação, testes e futuras mudanças.

### 9.4. Cores de disponibilidade

| Limite livre | Cor |
|---|---|
| 75% ou mais | Verde |
| 40% a menos de 75% | Amarelo |
| 25% a menos de 40% | Laranja |
| Menos de 25% | Vermelho |

A função `corDisponibilidade()` gera a cor; `legendaLimites()` exibe a explicação.

### 9.5. Cadastro e edição

Um cartão pode incluir nome, bandeira, banco, quatro últimos dígitos, limite, dia de vencimento, dia de fechamento, conta de pagamento opcional e observações. O código valida nome único, limite não negativo, vencimento/fechamento entre 1 e 31, e final do cartão com quatro dígitos quando informado. Renomear o cartão atualiza as referências `origem:'cartao_'+nomeAntigo` nas despesas vinculadas.

## 10. Despesas e débitos automáticos

### 10.1. Cadastro e campos

A despesa registra descrição, valor, categoria, data e origem. Origens relevantes: `cartao_<nome>`, `conta_<nome>` e `dinheiro` (conta principal). Parcelas e recorrências adicionam identificadores e campos próprios. Os detalhes permitem edição de status e exclusão lógica.

### 10.2. Origem financeira

- **Conta corrente/dinheiro físico:** a função de cadastro pode marcar a despesa como paga e realizar débito do saldo quando chegar a data, ou agendar a execução para data futura, conforme origem e controle `debitoAuto`.
- **Cartão de crédito:** o lançamento integra a cobrança do mês de sua data de vencimento e pode comprometer limite. Não deve ser debitado da conta bancária apenas por ter sido cadastrado no cartão.
- **Origens antigas:** podem não ter débito automático, conforme regras de compatibilidade exibidas pelo próprio formulário.

O conjunto `debitoAuto`, `agendado`, `debitado`, `debitoValor` e `debitoConta` registra o estado da execução e ajuda a evitar aplicação dupla quando um lançamento muda de valor, origem ou status.

### 10.3. Status

Estados utilizados para despesas: **Pendente**, **Pago** e **Cancelado**. Alterar o status pode debitar ou estornar saldo, dependendo do tipo de origem e de como a despesa foi registrada. A interface também permite editar ou remover ocorrências individuais.

### 10.4. Filtros e listas

A listagem de despesas possui consulta por período, filtros e detalhamento de linhas. Em páginas abrangentes, as listas são agrupadas por menus retráteis; os botões de cadastro permanecem disponíveis independentemente de o histórico estar expandido.

## 11. Recebimentos e créditos

Os recebimentos possuem descrição, valor, data, categoria de recebimento e conta de destino, podendo vincular-se a uma meta por `metaId`. O status pode ser **Recebido**, **Pendente** ou **Cancelado**. Quando há confirmação de recebimento elegível, a rotina de crédito pode atualizar o saldo da conta correspondente; quando o status é revertido, o crédito anterior pode ser estornado.

Os campos técnicos `creditado`, `creditoConta` e `creditoValor` são fundamentais à idempotência do saldo: uma importação completa deve preservá-los, **não apenas recriar o recebimento visível**. Renomeações e alterações de destino exigem atenção aos vínculos.

As entradas podem gerar gráficos por conta de destino, categoria de recebimento e outras classificações; os detalhes permanecem acessíveis via tabelas e drill-down.

## 12. Metas a Receber

Uma meta em `dados.planejamentos` registra `descricao`, `valor`, `data`, `status` e destino opcional. A criação pode produzir diversas ocorrências segundo frequência/quantidade selecionadas. A confirmação, quando possui destino financeiro, aciona `sincronizarMeta()` e pode gerar ou atualizar um **recebimento associado por `metaId`**, de modo que a meta não dependa de um recebimento manual duplicado.

Se a meta deixa de estar Confirmada ou perde o destino, a rotina pode remover o recebimento que ela própria havia gerado e estornar seu crédito. O sistema pode abrir uma confirmação específica para escolher destino antes de confirmar a meta.

**Interpretação:** uma meta é planejamento/expectativa; o recebimento associado representa entrada efetivada segundo os controles de status. Não somar os dois como se fossem receitas independentes em qualquer relatório.

## 13. Recorrências e parcelamentos

### 13.1. Recorrência

Recorrência descreve repetição de uma receita, despesa ou meta de acordo com regras de frequência, sem implicar necessariamente dívida parcelada em cartão. O código contempla frequência única, semanal, quinzenal, mensal e anual nos fluxos pertinentes. A quantidade validada para recorrência é de **1 a 600 ocorrências**.

A função `dataOcorrencia(inicio,frequencia,indice)` avança 7 dias para semanal, 15 dias para quinzenal, meses/anos para frequências mensais/anuais e ajusta o dia ao último dia válido do mês quando necessário. Séries mantêm `recorrenciaId`, números, contagem e descrição-base. `salvarSerie()` procura preservar identificadores e status dos itens mantidos em uma edição.

### 13.2. Parcelamento de cartão

Um parcelamento tem **1 a 120 parcelas** conforme validação do formulário. Se a quantidade for **1**, o fluxo cria uma **despesa única** no cartão. Se for maior que 1, cria lançamentos individuais mensais em `dados.despesas`, ligados por `parcelamentoId`, com `numeroParcela` e `parcelas`.

O valor total é distribuído em centavos para que a soma das parcelas corresponda ao total contratado, inclusive quando a divisão não é exata. O grupo aparece na seção Parcelamentos e pode ser aberto, editado ou movido à lixeira.

### 13.3. Status agrupado

`statusControleGrupo()` reconhece situações homogêneas ou **Misto**; `atualizarStatusGrupo()` pode alterar os status das parcelas do grupo **dentro do período selecionado**. Essa operação é distinta de uma quitação de fatura inteira com histórico individual de reversão.

### 13.4. Datas relevantes

- `dataCompra`: quando a compra ocorreu, se fornecida;
- `data`: quando aquela cobrança/parcela vence ou pertence ao período;
- `numeroParcela` / `parcelas`: posição de cada parcela;
- `parcelamentoId`: vínculo do conjunto;
- `recorrenciaId`: vínculo da série de repetição, conceito separado de parcelamento.

## 14. Categorias e ícones

A interface permite cadastrar categorias com nome, ícone e cor. Os gráficos por categoria utilizam essas cores; as telas de cadastro exibem biblioteca de emojis/ícones. A exclusão normal integra a lixeira e suas regras de restauração.

**Cuidado:** despesas tipicamente referenciam categorias por `nome` (string), e não exclusivamente pelo `id` do cadastro. Renomear ou apagar categorias merece testes de associação para não deixar lançamentos com nomes sem correspondência visual.

## 15. Investimentos: visão geral

Investimentos são cadastrados em `dados.investimentos`, com classe **fixa** ou **variavel**, tipo e histórico de movimentos em cada aplicação. O painel oferece resumo de patrimônio, distribuição por classe, tipo, conta vinculada, histórico mensal, rendimentos/proventos e agenda de vencimentos conforme dados disponíveis.

O catálogo nativo contém:

| Renda fixa | Renda variável |
|---|---|
| Poupança | Ações |
| Tesouro Direto | FIIs |
| CDB | ETFs |
| LCI | BDRs |
| LCA | Criptomoedas |
| Debêntures | — |

Há também tipos personalizados em `dados.tiposInvestimento`. O patrimônio de investimentos é tratado separadamente do saldo líquido disponível, evitando o erro de considerar uma aplicação simultaneamente como dinheiro livre e ativo investido.

### 15.1. Hierarquia de navegação

Investimentos → Renda Fixa ou Renda Variável → Tipo → Lista de aplicações → Aplicação → Operações/indicadores. Há vistas de histórico, proventos e vencimentos. Agrupamentos são implementados com seções expansíveis, incluindo `<details>` em alguns casos.

## 16. Investimentos: renda fixa

### 16.1. Informações registráveis

Os campos variam por tipo e podem incluir: emissor, indexador, taxa anual, taxa-base, percentual do indexador, spread, data inicial, vencimento, tributação, valor atual informado, conta de investimento, carência, liquidez e informações descritivas. Os movimentos típicos são **aporte, resgate, juros/rendimento recebido**, conforme o controle do formulário.

### 16.2. Taxas e evolução

`taxaEfetivaInv()` calcula uma taxa anual de referência, usando as seguintes aproximações do código:

```text
CDI:         taxa-base × percentual-do-indexador + spread
IPCA+ / TR+: (1 + taxa-base) × (1 + spread) − 1
Selic:       taxa-base + spread
Demais:      taxa anual cadastrada
```

Os valores percentuais são transformados em frações antes das operações. `aplicarFluxosFixos()` capitaliza aportes e resgates até a data-alvo usando aproximadamente:

```text
valor_movimento_na_data = valor_original × (1 + taxa_anual)^(dias/365)
saldo_projetado = soma(aportes capitalizados) − soma(resgates capitalizados)
```

Quando existe `saldoAtual` informado, ele pode prevalecer sobre o valor estimado. A projeção até o vencimento é **estimativa determinística**, não cotação bancária ou promessa de retorno. `invAnalise()` também expõe principal, resultado, rendimentos recebidos e, quando aplicável, uma faixa ilustrativa de IR regressivo por tempo de aplicação.

### 16.3. Riscos de interpretação

As estimativas não substituem regras particulares de títulos, indexadores reais, impostos em todas as hipóteses, custos operacionais ou marcação a mercado. A documentação registra o algoritmo implementado, e não certifica precisão jurídica ou econômica da taxa efetiva.

## 17. Investimentos: renda variável

### 17.1. Cadastro e operações

Renda variável pode possuir código/ticker, preço atual informado, conta vinculada e campos particulares por classe. As operações típicas são **compra**, **venda** e **provento**. Cada compra registra quantidade, preço e possíveis taxas; uma venda também utiliza esses campos.

### 17.2. Custo médio e resultado

Em `invAnalise()` a lógica principal é:

```text
Ao comprar:
  custo += quantidade × preço + taxas
  quantidade_total += quantidade

Preço médio = custo / quantidade_total, quando quantidade_total > 0

Ao vender:
  custo_da_parcela_vendida = (custo / quantidade_total) × quantidade_vendida
  resultado_realizado += valor_bruto_da_venda − taxas − custo_da_parcela_vendida
  custo -= custo_da_parcela_vendida
  quantidade_total -= quantidade_vendida

Valor atual = quantidade_total × (cotação informada, se houver;
                                     caso contrário, custo médio)
```

Proventos recebidos são acumulados separadamente. Preços, quantidades e histórico são dados locais informados/registrados; não foi identificada integração de mercado em tempo real.

## 18. Investimentos: movimentos, relatórios e personalização

A interface permite inserir, editar e remover movimentos de investimentos, com validações de quantidade disponível antes de vendas e de saldo/valor aplicável antes de resgates. `resumoCarteira()` combina patrimônio, principal, aportes, resultados e proventos das aplicações.

Os históricos mensais e gráficos de evolução utilizam as datas dos movimentos e o modelo de avaliação disponível, podendo agrupar por tipo, classe ou conta. O painel também oferece agenda de vencimentos para aplicações com data de vencimento.

**Tipos personalizados:** `tiposInvestimento` guarda classe, nome, descrição e lista de campos; os tipos são agregados ao catálogo da classe correspondente. A remoção de um tipo personalizado pode incluir aplicações vinculadas, preservadas na lixeira segundo o fluxo de exclusão.

## 19. Transferências

Transferências em `dados.transferencias` movem saldos entre contas e/ou dinheiro físico **sem criar uma despesa ou um recebimento adicional**. O saldo do destino sobe e o da origem cai pelo mesmo valor; em operação simples, o total consolidado do conjunto de contas não muda.

### 19.1. Identificação de extremos

O modelo atual usa `origem` e `destino` com chaves de conta do tipo `c:<nome>` ou a chave especial `fisico` para dinheiro físico principal. Essas chaves são distintas das strings `conta_<nome>` e `cartao_<nome>` usadas em despesas.

### 19.2. Criar, editar e reverter

`salvarTransferencia()` verifica origens e destinos distintos, data e valor positivo; confirma saldos existentes e disponibilidade suficiente. Quando se edita, calcula a reversão da transferência antiga e aplica a nova, evitando acumular efeitos de duas versões de um mesmo registro. Excluir uma transferência pode estornar o saldo associado, sujeito a validações.

A seção exibe indicadores de saldos e um menu retrátil **Histórico de transferências (n)**, com edição e exclusão. Dentro de cada conta há outro menu independente para transferências daquela conta no período.

## 20. Lixeira e recuperação

A lixeira conserva registros retirados das coleções operacionais, juntamente com o contexto necessário à restauração. Uma entrada normalmente possui título, data de exclusão, descrição, `registros` e `meta`; cada registro informa sua coleção, posição original e objeto arquivado.

O sistema pode mover à lixeira lançamentos individuais, grupos de parcelas, contas com vínculos, cartões com despesas, investimentos com movimentos, tipos personalizados e categorias, de acordo com o fluxo implementado. Alguns tipos de exclusão são coletivos, pois preservar apenas o cadastro sem seus lançamentos quebraria relacionamentos.

**Distinção crítica:** mover para a lixeira não equivale a eliminar definitivamente o registro do JSON exportado. Como o backup copia o objeto `dados` inteiro, ele inclui a própria `lixeira`. Excluir definitivamente ou esvaziar a lixeira pode tornar o conteúdo não recuperável pelo painel atual, salvo backup anterior.

## 21. Estados financeiros e reconciliação

| Entidade | Estados de uso corrente | Efeito principal |
|---|---|---|
| Despesa | Pendente, Pago, Cancelado | Pode afetar saldo quando há débito automático; pode liberar ou reservar limite de cartão |
| Recebimento | Recebido, Pendente, Cancelado | Pode creditar ou estornar conta de destino |
| Meta | Planejado, Confirmado, Cancelado | Pode gerar/remover recebimento associado conforme destino |
| Parcelamento | Estado de cada parcela; grupo pode exibir Misto | A edição em grupo atua nas parcelas elegíveis ao período |

`atualizarStatus()` altera o estado; fluxos especializados como `efetuarDebitoDespesa`, `estornarDebitoDespesa`, `agendarOuDebitar`, `creditarRecebimento`, `estornarCreditoRecebimento` e `sincronizarMeta` registram o efeito financeiro correspondente.

**Regra de preservação:** os campos técnicos de execução devem migrar com os registros. Importar somente título, data e valor seria insuficiente para identificar quais débitos/créditos já foram realizados e poderia distorcer saldos.

## 22. Fórmulas e indicadores de saldo

### 22.1. Saldos disponíveis e patrimônio

- `saldoContasDisponiveis()` soma as contas normalizadas como **corrente** ou **poupança**.
- `saldoDinheiroTotal()` agrega contas de **dinheiro físico**, ou utiliza o valor legado quando necessário.
- `valorConta(c)` utiliza o saldo da conta e, para contas de investimento, pode adicionar patrimônio das aplicações vinculadas.
- `valorTipoConta(t)` agrega os valores das contas daquele tipo.
- `totalFaturasPendentesMes()` soma compras únicas de cartão **pendentes** no mês selecionado.
- `totalParcelasPendentesMes()` soma parcelas de cartão **pendentes** no mês selecionado.

O indicador **Saldo Líquido** exibido na implementação é, conceitualmente:

```text
saldo digital disponível
+ dinheiro físico total
− compras únicas de cartão pendentes no mês selecionado
− parcelas de cartão pendentes no mês selecionado
```

Esse número representa um **saldo disponível ajustado pelos compromissos do período** segundo regras específicas do código. Não é sinônimo de patrimônio total, fluxo de caixa de competência ou saldo bancário oficial.

### 22.2. Investimento versus disponibilidade

A interface pode mostrar patrimônio por tipo de conta ou aplicações, além do caixa disponível. Previdência e patrimônio investido não entram automaticamente no mesmo total operacional usado para despesas do dia a dia. Sempre conferir o rótulo do indicador antes de somar grupos.

### 22.3. Percentuais e gráficos

Percentuais são derivados dos valores agregados no contexto. Gráficos de rosca e barras podem abrir listas de origem; séries de linha são calculadas conforme a modalidade. Projeções de renda fixa dependem dos dados e taxas informados. Divisões por zero retornam percentuais nulos ou valores alternativos conforme cada função.

## 23. Menus retráteis e experiência de uso

A versão atual adota uma experiência compacta para evitar páginas longas. Sempre que possível, agrupamentos mostram título, quantidade de ocorrências e breve contexto, enquanto gráficos e tabelas permanecem recolhidos até a ação do usuário.

**Convenções de design existentes:** cards com ícones, detalhes abertos em páginas internas, títulos com contagem `(n)`, comandos de cadastro visíveis e botão Voltar em drill-down. Os gráficos das contas foram deslocados para depois de lançamentos e transferências; dentro do cartão, indicadores ficam antes dos grupos de lançamentos/gráficos.

### 23.1. Critério para `(n)`

O significado de `(n)` varia conforme o bloco: quantidade de lançamentos filtrados, transferências relacionadas no período, contas cadastradas, aplicações, cartões cadastrados etc. Não interpretar a contagem sem o título da seção. Alguns históricos exibem todos os registros, mesmo quando o KPI associado mostra somente o período filtrado.

### 23.2. Acessibilidade básica

Linhas clicáveis usam atributos `role="button"`, `tabindex="0"` e, em trechos importantes, ativação por Enter ou Espaço. Expansões atualizam `aria-expanded`; botões Voltar possuem rótulos. Há suporte a telas menores, mas tabelas largas podem exigir rolagem horizontal.

## 24. Salvar o HTML

1. Faça as alterações desejadas.
2. Observe `● Alterações não salvas`.
3. Clique em **💾 Salvar Arquivo**.
4. Quando disponível, a API `showSaveFilePicker` permite escolher/sobrescrever arquivo.
5. Caso contrário, a função oferece um arquivo HTML para download.
6. Confirme o arquivo no disco e reabra-o para validar persistência.

**Como funciona:** a função `salvarArquivo()` serializa o objeto `dados` em JSON, escapa caracteres perigosos para o script, substitui o conteúdo de `<script id="dados_injetados">` no HTML serializado e gera um `Blob`. O carimbo `dados.timestamp` é atualizado. O arquivo resultante contém código e dados.

O aviso de download iniciado não equivale à comprovação automática de conclusão do download pelo sistema operacional. Não feche a sessão com alterações pendentes sem verificar o arquivo salvo.

## 25. Exportar backup JSON

### 25.1. Fluxo de uso

1. Clique em **📤 Exportar dados**.
2. O navegador iniciará o download de um arquivo `Backup_Painel_Financeiro_<data-hora>.json`.
3. Guarde o arquivo em local seguro, preferencialmente com uma cópia adicional.
4. Não altere o conteúdo manualmente se não dominar o contrato de dados.

`exportarDadosJSON()` chama `criarConteudoBackup()` e serializa o estado completo em UTF-8, com identação. Não altera saldos, estados ou o HTML salvo: é uma fotografia do estado **atual em memória**, inclusive alterações ainda não gravadas no HTML.

### 25.2. Formato de envelope

```json
{
  "formato": "PainelFinanceiro",
  "versaoEstrutura": 1,
  "versaoPainel": "Painel HTML independente (backup v1)",
  "exportadoEm": "2026-11-03T14:00:00.000Z",
  "dados": {
    "contas": [],
    "cartoes": [],
    "despesas": [],
    "recebimentos": [],
    "planejamentos": [],
    "categorias": [],
    "investimentos": [],
    "transferencias": [],
    "movContas": [],
    "tiposInvestimento": [],
    "lixeira": [],
    "navOrder": [],
    "dinheiroFisico": 0,
    "timestamp": 0,
    "zerado": true
  }
}
```

**Exemplo de estrutura**, sem dados reais. Os campos `dados` do backup efetivo podem conter flags e propriedades adicionais, que o exportador preserva ao copiar integralmente o objeto.

### 25.3. Conteúdo preservado

São exportados registros ativos, campos auxiliares, IDs, vínculos de recorrência e parcelamento, status de pagamento, metadados de crédito/débito, movimentos de investimento, transferências, lixeira, informações cadastrais, ordem de navegação e flags existentes. O JSON **não contém o código HTML, CSS ou a lógica executável do painel**.

### 25.4. Limites

O backup não é criptografado e pode expor dados financeiros pessoais. Não é sincronização em nuvem, não compara versões e não valida, no ato da exportação, se cada relação financeira está correta. Ele preserva fielmente o estado disponível.

## 26. Importar backup JSON e restaurar painel vazio

### 26.1. Procedimento recomendado

1. Gere um backup JSON a partir do painel com dados.
2. Abra `Painel_Financeiro_Em_Branco_Importar.html` ou outro HTML compatível.
3. Clique em **📥 Importar dados** e selecione o arquivo `.json`.
4. Aguarde a verificação e examine a prévia com contagem das coleções.
5. Clique em **Confirmar e substituir os dados**.
6. Confira contas, cartões, despesas, recebimentos, investimentos, transferências e saldos.
7. **Clique em Salvar Arquivo** para incorporar a base restaurada no novo HTML.
8. Feche e reabra o HTML salvo para uma conferência final.

### 26.2. O que ocorre internamente

- `processarArquivoBackup(event)` lê o arquivo local e executa `JSON.parse`.
- `prepararImportacaoJSON(obj)` identifica o envelope e valida campos essenciais.
- `resumoImportacaoHTML()` apresenta as contagens por coleção, inclusive movimentos internos de investimentos.
- O botão de confirmação aciona `confirmarImportacaoJSON()`.
- A rotina cria uma fotografia dos dados anteriores, substitui `dados` pelo conjunto importado, garante módulos e conta técnica de dinheiro, reinicia a navegação e marca as alterações como pendentes.

**Substituição integral:** a importação não adiciona apenas registros que faltam nem tenta fundir dois painéis. O conjunto de destino é **substituído** pelo conjunto do backup. Para consolidar duas bases diferentes, seria necessário projeto específico de mesclagem e resolução de conflitos.

### 26.3. Validações implementadas

| Verificação | Resultado esperado |
|---|---|
| Arquivo maior que 30 MB | Rejeição antes de ler/processar |
| JSON sintaticamente inválido | Rejeição e mensagem de erro |
| Envelope com `formato` incompatível | Rejeição, salvo objeto legado reconhecido |
| `versaoEstrutura` futura ou inválida | Rejeição até existir conversor específico |
| Coleção obrigatória com tipo diferente de lista | Rejeição |
| Elemento não-objeto nas coleções de registros | Rejeição |
| Chaves `__proto__`, `constructor`, `prototype` | Rejeição |
| Profundidade superior ao limite de 45 níveis | Rejeição |
| Número não finito no objeto | Rejeição |
| Coleção ausente em formato legado | Criação da lista vazia correspondente |
| `dinheiroFisico` ausente | Inicialização com zero |

**Limitação importante:** essa validação é estrutural e de segurança básica. Não é uma verificação financeira exaustiva: não garante que todas as referências a contas/cartões existam, que cada parcela some ao total ou que os status de crédito/débito sejam economicamente coerentes. Faça conferência após restaurar.

### 26.4. Contagem de prévia

A prévia distingue as coleções `contas`, `cartoes`, `despesas`, `recebimentos`, `planejamentos`, `categorias`, `investimentos`, `transferencias`, `movContas`, `tiposInvestimento`, `lixeira`, além de somar as operações presentes no array `movimentos` de cada investimento. A quantidade de parcelas é refletida como lançamentos individuais na coleção `despesas`.

## 27. Versionamento e migração entre releases

### 27.1. Contrato atual

O importador reconhece oficialmente o envelope `PainelFinanceiro` versão **1**. Também reconhece um **objeto bruto legado** quando identifica arrays de coleções financeiras conhecidas, atribuindo-lhe classificação interna versão 0. Nesse caso, completa listas ausentes, ajusta flags de inicialização e conserva os demais campos existentes.

**Não significa compatibilidade garantida com qualquer versão futura.** A validação rejeita versões de estrutura acima de 1 até que seja implementado um conversor correspondente.

### 27.2. Regra de evolução

Alterações exclusivamente visuais — cor, layout, menu, gráfico, posição de cartões e drill-down — podem usar o mesmo backup sem migração, desde que o formato de `dados` permaneça compatível. Alterações estruturais — renomeação de campos, mudança de IDs, troca de chaves de relações ou mudança de semântica — exigem conversor explícito.

Protocolo sugerido para versões futuras:

```text
1. Congelar o contrato da estrutura anterior e documentá-lo.
2. Alterar BACKUP_VERSAO_ESTRUTURA somente quando o schema mudar.
3. Implementar migrarV1ParaV2(), migrarV2ParaV3() etc.
4. Validar integridade do objeto convertido antes de substituir dados.
5. Testar fixtures legadas e dados limpos.
6. Manter exportação do novo formato e testes de reabertura do HTML.
```

Essas funções de migração encadeadas são **proposta de manutenção futura**; não devem ser consideradas já implementadas no código atual.

### 27.3. Preservação de extensões

Como o backup copia o objeto integralmente, campos desconhecidos podem ser carregados juntamente com o restante. Isso não garante que a interface atual saiba **interpretar** tais campos; portanto, seu significado deve continuar documentado e acompanhado por conversões quando necessário.

## 28. Desfazer, zerar e recuperar

### 28.1. Desfazer a última importação

Após confirmar uma importação, aparece **↶ Desfazer importação**. A função usa uma fotografia anterior do objeto `dados`. Exige confirmação e restaura a situação imediatamente anterior àquela importação, também marcando alterações como pendentes.

**Escopo:** há apenas **um snapshot** da última importação. Ele vive na sessão e não constitui histórico ilimitado de desfazimentos. Alterações feitas após a importação também podem ser perdidas ao desfazer. Após salvar/reabrir o arquivo, não conte com esse snapshot como recuperação durável.

### 28.2. Zerar painel

O comando **🗑️ Zerar painel** possui duas confirmações. Zera contas, cartões, lançamentos, metas, investimentos, transferências, movimentos, lixeira e saldos **na memória**, mas **preserva as categorias cadastradas** no comportamento atual. O zeramento só entra no HTML quando o usuário usa **Salvar Arquivo**.

### 28.3. Versão limpa versus zeramento

A versão limpa fornecida separadamente começa sem registros ou categorias, com flags de inicialização apropriadas. O comando Zerar de uma versão preenchida, ao contrário, conserva suas categorias. Portanto, os dois recursos não são idênticos. **Para começar absolutamente do zero e importar depois**, prefira a versão limpa oficial.

### 28.4. Recuperação de emergência

Se uma importação for incorreta e ainda estiver na mesma sessão, use **Desfazer importação**. Se o HTML original não foi sobrescrito, feche sem salvar e reabra a cópia antiga. Se ele foi substituído, use outro HTML de backup ou um JSON previamente exportado. Evite depender exclusivamente do navegador para recuperação.

## 29. Modelo de dados: dicionário técnico

O objeto global `dados` é a unidade de persistência e portabilidade. Campos opcionais podem variar conforme a origem do registro, migrações e operações executadas. A tabela abaixo descreve **campos verificados no código e nos exemplares**, não um JSON Schema formal que o importador valide integralmente.

### 29.1. Raiz

| Campo | Tipo esperado | Função |
|---|---|---|
| `contas` | `Array<object>` | Contas e saldos cadastrados |
| `cartoes` | `Array<object>` | Cadastro de cartões de crédito |
| `despesas` | `Array<object>` | Compras, despesas, parcelas, recorrências e seu estado |
| `recebimentos` | `Array<object>` | Entradas de dinheiro, confirmação e crédito |
| `planejamentos` | `Array<object>` | Metas e expectativas de recebimento |
| `categorias` | `Array<object>` | Classificações, ícones e cores |
| `investimentos` | `Array<object>` | Aplicações e seus movimentos internos |
| `transferencias` | `Array<object>` | Transferências que movem saldos |
| `movContas` | `Array<object>` | Movimentação manual de modalidades de conta |
| `tiposInvestimento` | `Array<object>` | Tipos personalizados de investimento |
| `lixeira` | `Array<object>` | Registros removidos logicamente e dados de restauração |
| `navOrder` | `Array` | Ordenação dos módulos no menu |
| `dinheiroFisico` | `number` | Saldo legado sincronizado com conta física principal |
| `timestamp` | `number` | Carimbo atualizado ao salvar HTML |
| `zerado` | `boolean` | Sinalização de inicialização sem reinserir exemplos |
| `exemplosV13` | `boolean` | Controle de exemplos antigos |
| `exemplosInvestV17` | `boolean` | Controle de exemplos de investimento |
| `demoContasV18` | `boolean` | Controle de exemplos de contas |
| `migracaoRecebV1` | `boolean` | Controle da migração de recebimentos/créditos |

### 29.2. Contas (`contas[]`)

| Campo | Uso |
|---|---|
| `nome` | Identificador textual exibido e referenciado por vários vínculos |
| `tipo` | Modalidade: corrente, poupança, investimento, previdência ou dinheiro |
| `saldo` | Saldo cadastrado/atual da conta |
| `banco`, `agencia`, `numero` | Dados institucionais opcionais |
| `notas` | Observações livres |
| `id`, `sistema` | Marcadores técnicos opcionais; `sistema:'dinheiroFisico'` identifica a conta principal em espécie |

### 29.3. Cartões (`cartoes[]`)

| Campo | Uso |
|---|---|
| `nome` | Identificador textual, empregado em `cartao_<nome>` |
| `banco`, `bandeira`, `final` | Identificação visual do cartão |
| `limite` | Limite nominal |
| `vencimento` | Dia do mês do vencimento |
| `fechamento` | Dia do mês de fechamento, quando informado |
| `contaPagamento` | Conta sugerida/opcional no cadastro |
| `notas` | Observações |
| `faturaAtual` | Campo legado encontrado em registros, não a fonte única do cálculo atual de fatura |

### 29.4. Despesas (`despesas[]`)

| Campo | Uso |
|---|---|
| `id` | ID do lançamento individual |
| `titulo`, `categoria`, `valor`, `data` | Dados básicos |
| `origem` | `cartao_<nome>`, `conta_<nome>`, `dinheiro` ou valor legado |
| `status` | Pendente / Pago / Cancelado; ausência pode usar default lógico |
| `dataCompra` | Data da compra, distinta da data de cobrança |
| `parcelas`, `numeroParcela`, `parcelamentoId` | Vínculo e posição do parcelamento |
| `recorrenciaId`, `numeroRecorrencia`, `totalRecorrencias` | Vínculo da série recorrente |
| `frequenciaRecorrencia`, `inicioRecorrencia`, `tituloBaseRecorrencia` | Metadados de repetição |
| `debitoAuto`, `agendado`, `debitado` | Estado da automação de débito |
| `debitoValor`, `debitoConta` | Referência ao débito aplicado, útil ao estorno |

### 29.5. Recebimentos (`recebimentos[]`)

| Campo | Uso |
|---|---|
| `id` | ID do recebimento |
| `titulo`, `valor`, `data` | Dados básicos |
| `categoriaRecebimento` | Categoria/modalidade da entrada |
| `conta` | Destino financeiro |
| `status` | Recebido / Pendente / Cancelado |
| `metaId` | Meta que gerou o recebimento, quando aplicável |
| `creditado`, `creditoValor`, `creditoConta` | Controle de crédito efetuado e possível estorno |
| Campos de `recorrencia*` | Contexto de recorrência, se houver |

### 29.6. Metas (`planejamentos[]`)

| Campo | Uso |
|---|---|
| `id` | ID da meta |
| `descricao` | Descrição da expectativa |
| `valor`, `data` | Valor esperado e data |
| `status` | Planejado, Confirmado ou Cancelado |
| `destino` | Conta que receberá o valor quando confirmada |

### 29.7. Categorias (`categorias[]`)

`{id, nome, icon, cor}`: ID do cadastro, nome de exibição/referência, emoji/ícone e cor CSS.

### 29.8. Investimentos (`investimentos[]`)

| Grupo de campos | Uso |
|---|---|
| `id`, `nome`, `classe`, `tipo` | Identidade, classe fixa/variável e tipo |
| `codigo`, `contaInv`, `descricao` e campos específicos | Caracterização e conta vinculada |
| `taxaAnual`, `indexador`, `taxaBase`, `percentualIndexador`, `spread` | Parâmetros de estimativa de renda fixa |
| `tributacao`, `vencimento`, `saldoAtual` | Tributação, agenda e valor observado de renda fixa |
| `cotacaoAtual` | Cotação digitada para avaliação da renda variável |
| `movimentos` | Lista aninhada de operações da aplicação |
| Campos de tipo personalizados | Atributos especiais definidos para classe/categoria |

**Movimento de investimento:** objeto com campos usuais `{id, data, tipo, valor, quantidade, preco, taxas, nota}`; o significado de quantidade/preço depende de compra/venda versus aporte/resgate.

### 29.9. Transferências (`transferencias[]`)

`{id, data, origem, destino, valor, nota}`. `origem` e `destino` referenciam a conta por `c:<nome>` ou, para dinheiro principal, `fisico`.

### 29.10. Movimentos de contas (`movContas[]`)

`{id, conta, data, tipo, valor, nota}`. A semântica do tipo é definida pela modalidade da conta e pela lista de opções do formulário correspondente.

### 29.11. Tipos de investimento (`tiposInvestimento[]`)

Campos comuns: `classe`, `nome`, `descricao`, `campos` (lista de chaves de atributos habilitados).

### 29.12. Lixeira (`lixeira[]`)

| Campo | Uso |
|---|---|
| `id`, `titulo`, `detalhe` | Identidade e descrição do arquivamento |
| `dataExclusao` | Data/hora do arquivamento |
| `registros` | Lista `{colecao, indice, item}` de objetos removidos |
| `meta` | Dados auxiliares, por exemplo instruções de reaplicação de transferência |

## 30. Relacionamentos e integridade

```text
Cartão(nome)
   └── Despesa(origem = "cartao_" + nome)
         ├── Parcelamento(parcelamentoId, numeroParcela, parcelas)
         └── Recorrência(recorrenciaId, numeroRecorrencia, ...)

Conta(nome)
   ├── Despesa(origem = "conta_" + nome; ou dinheiro principal)
   ├── Recebimento(conta / creditoConta = nome)
   ├── Meta(destino = nome)
   │     └── Recebimento(metaId = ID da meta)
   ├── MovimentoManual(conta = nome)
   ├── Investimento(contaInv = nome)
   └── Transferência(origem/destino = "c:" + nome; ou fisico)

Lixeira
   └── Registros removidos + índice original + metadados de recuperação
```

Os registros dependem tanto de **IDs próprios** quanto de **chaves textuais** (nome do cartão/conta). É por isso que renomeações precisam atualizar referências, e por que conversões de versões devem manter vínculos de forma transacional.

### 30.1. Invariantes recomendados para importação e manutenção

- IDs de lançamentos não devem ser reaproveitados para entidades diferentes.
- Todo `parcelamentoId` deve identificar um grupo coerente, e `numeroParcela` deve estar dentro da quantidade contratada.
- `metaId` deve apontar para a meta correta e não produzir recebimentos duplicados.
- `origem`/`destino` de transferências devem referenciar contas existentes ou `fisico` válido.
- Os campos de crédito/débito devem corresponder à operação já refletida no saldo.
- Investimentos não devem perder o array `movimentos` ao atualizar apenas propriedades cadastrais.
- `lixeira` precisa conservar os objetos originais para que restaurações sejam possíveis.

Esses são **critérios de integridade para testes e futuras melhorias**; não declarar que todos sejam verificados integralmente pelo importador v1.

## 31. Arquitetura de código e elementos de interface

### 31.1. Responsabilidades principais

| Domínio | Entradas técnicas verificadas |
|---|---|
| Inicialização | `boot`, `gerarDadosDeExemplo`, `garantirModulos`, `garantirContaDinheiroPrincipal` |
| Persistência HTML | `salvarArquivo`, `salvarDadosLocal`, `marcarAlteracao`, `limparIndicadorAlteracao` |
| Backup JSON | `criarConteudoBackup`, `contagensBackup`, `conferirSegurancaBackup`, `prepararImportacaoJSON`, `resumoImportacaoHTML`, `processarArquivoBackup`, `confirmarImportacaoJSON`, `desfazerUltimaImportacao` |
| Navegação | `selecionarVisao`, `renderVisaoBase`, `renderVisao`, `navegarVoltar`, `injetarVoltarDrilldown`, `header` |
| Expansões | `expBloco`, `expLinhaDe`, `preencherExp`, `definirExp`, `alternarExp`, `alternarTudo` |
| Período | `emPeriodo`, `mudarMes`, `selecionarPeriodo`, `moverPeriodo`, `atualizarLabelMes` |
| Contas | `renderContasPagina`, `renderContasTipo`, `renderDetalheConta`, `valorConta`, `saldoContasDisponiveis` |
| Cartões | `renderCartoesPagina`, `renderListaCartoes`, `renderDetalheCartao`, `usadoLimiteCartao`, `reservaFuturaCartao` |
| Parcelamentos | `gruposParcelas`, `renderListaParcelamentos`, `renderDetalheParcelamento`, `salvarParcelamento` |
| Despesas | `renderDetalheDespesa`, `salvarDespesa`, `efetuarDebitoDespesa`, `estornarDebitoDespesa` |
| Recebimentos | `renderDetalheRecebimento`, `salvarRecebimento`, `creditarRecebimento`, `estornarCreditoRecebimento` |
| Metas | `renderDetalhePlan`, `sincronizarMeta`, `abrirConfirmacaoMeta` |
| Investimentos | `renderInvestimentos`, `renderDetalheInvestimento`, `invAnalise`, `resumoCarteira`, `salvarMovInvest` |
| Transferências | `renderTransferencias`, `salvarTransferencia`, `excluirTransferencia` |
| Exclusões | `criarEntradaLixeira`, `restaurarEntradaCore`, `renderLixeira`, `restaurarLixeira` |

**Localização:** o apêndice apresenta inventário de declarações de função com as linhas correspondentes no HTML principal identificado por SHA-256.

### 31.2. Elementos de interface relevantes

| Elemento DOM | Finalidade |
|---|---|
| `dados_injetados` | Script contendo o snapshot financeiro em `window.INJETADO` |
| `contextContent` | Conteúdo principal dinâmico |
| `contextTitle`, `contextSubtitle` | Títulos e contexto da página |
| `contextBack` | Voltar do cabeçalho |
| `periodSelector` | Filtro de período global |
| `navCards` | Cards de navegação |
| `unsavedIndicator` | Indicador de alterações pendentes |
| `backupImportInput` | Seletor de arquivo JSON |
| `backupUndoBtn` | Reversão da última importação da sessão |
| `backupModal`, `backupResumo` | Confirmação e prévia do backup |

### 31.3. Modais

A interface possui formulários/modais próprios de despesa, recebimento, meta, categoria, conta, cartão, parcelamento, edição de parcelamento, confirmação de meta, quadro/gráfico, movimento de conta, investimento, operação de investimento, tipo personalizado, transferência e backup. Eles são abertos/fechados por `openModal()` e `closeModal()`, além de funções de cada domínio.

### 31.4. Estratégia de evolução

O código é extenso e combina funções tradicionais com atribuições no objeto `window`, além de rotinas acrescentadas em etapas sucessivas. Alterações devem ser localizadas, acompanhadas de testes de navegação, dados e persistência. Evite substituir o HTML inteiro por uma reconstrução de interface que omita funções ou coleções existentes.

## 32. Procedimentos operacionais (roteiros)

### 32.1. Migrar base preenchida para nova versão

1. **Antes de editar:** faça uma cópia do HTML antigo e exporte um JSON.
2. Confirme que o novo HTML possui **Importar dados**, e que reconhece o formato/versão exportados.
3. Abra o novo HTML e importe o JSON.
4. Compare quantidades por coleção com a prévia do backup.
5. Verifique pelo menos uma conta, um cartão, uma despesa, um recebimento, uma transferência, um parcelamento e uma aplicação (quando existirem).
6. Verifique saldos, status e vínculos de pagamentos.
7. Salve o HTML novo e reabra-o.
8. Guarde o HTML antigo e o JSON até concluir a conferência.

### 32.2. Preencher painel limpo de uma só vez

1. Abra o HTML limpo.
2. Confirme que as coleções não possuem cadastros prévios (a conta técnica de dinheiro físico pode existir).
3. Importe o JSON exportado de um painel preenchido.
4. Confirme a substituição e confira os dados.
5. Salve e reabra o novo HTML.
6. Passe a trabalhar no arquivo recém-salvo, descartando apenas cópias dispensáveis **depois** de verificar o resultado.

### 32.3. Recuperar lançamento excluído

1. Abra **Lixeira**.
2. Localize o registro ou grupo correspondente.
3. Verifique se há vínculos (conta, cartão, parcelas, investimentos).
4. Restaure pelo comando disponível.
5. Confirme o retorno do dado e o efeito sobre o saldo.
6. Salve o HTML.

### 32.4. Desfazer uma importação incorreta

1. Não sobrescreva o HTML se ainda estiver em dúvida.
2. Clique em **↶ Desfazer importação** na mesma sessão.
3. Confirme a reversão.
4. Confira a restauração do estado anterior.
5. Decida conscientemente se deseja salvar esse estado ou simplesmente reabrir o HTML antigo.

### 32.5. Atualizar somente o layout em uma release futura

1. Preserve a identidade e os nomes das coleções.
2. Modifique somente as funções de renderização e estilos previstos.
3. Importe um JSON de teste com contas, cartões, recorrência, parcelas, créditos, débitos e investimentos.
4. Teste os drill-downs e botões Voltar.
5. Exporte novamente e compare a integridade do conteúdo dos dados.
6. Valide salvamento e reabertura do HTML sem regressões.

## 33. Segurança, privacidade e limites

O HTML preenchido **e o JSON de backup** podem conter dados pessoais e financeiros, inclusive observações livres, nomes de contas, identificações de cartão e históricos. Na entrega analisada:

- Não existe autenticação própria antes da abertura do arquivo local.
- O backup JSON é texto legível, **sem criptografia**.
- Não há banco remoto nem conexão externa necessária às operações examinadas.
- O navegador pode armazenar arquivos em Downloads, pastas sincronizadas ou locais compartilhados, conforme configurações do usuário.
- O HTML salvo embute os dados no código-fonte: qualquer pessoa com acesso ao arquivo pode consultá-los.
- A validação de importação é necessária, mas não substitui um backup preservado fora do aplicativo.

**Boas práticas:** guardar cópias em local seguro, proteger o dispositivo, evitar compartilhar arquivos preenchidos, manter backups datados, testar restauração periodicamente e não inserir credenciais ou senhas bancárias nos campos livres do painel.

## 34. Manutenção segura e contrato para futuras atualizações

### 34.1. O que não deve ser alterado inadvertidamente

- Identificação dos registros e suas referências por ID/nome;
- Lógica de contabilização de `debitado`/`creditado` e dados auxiliares;
- Chaves `origem`/`destino` das transferências;
- `parcelamentoId` e `recorrenciaId`;
- Cálculo de saldos disponíveis versus patrimônio investido;
- Persistência independente por HTML;
- Envelope de exportação e versão de estrutura;
- Importação confirmada, proteção contra arquivo inválido e desfazer na sessão;
- Botões Voltar e expansão dos blocos;
- Dados reais já incorporados no arquivo.

### 34.2. Procedimento de alteração

1. Criar cópia de trabalho do HTML.
2. Exportar JSON antes de qualquer edição.
3. Definir escopo da mudança: visual, regra financeira ou schema.
4. Revisar funções e vínculos que dependem do trecho alterado.
5. Atualizar apenas as regiões necessárias; não eliminar coleções por não estarem visíveis na interface.
6. Executar verificações proporcionais ao risco.
7. Importar backup de teste e salvar/reabrir HTML.
8. Atualizar este MD, a versão do schema quando aplicável e o registro de alterações.

### 34.3. Sugestão de política de releases

Distinguir `versaoPainel` (release da aplicação e layout) de `versaoEstrutura` (contrato dos dados). Uma mudança de estilo não requer mudança de `versaoEstrutura`. A renomeação de um campo persistido ou a mudança de semântica de débito pode requerê-la e sempre exige migração/teste.

## 35. Testes e critérios de aceite

### 35.1. Testes funcionais de navegação

- [ ] Abrir cada card principal sem erro.
- [ ] Abrir Contas → selecionar conta → voltar à lista.
- [ ] Abrir Cartões e Parcelamentos → Cartões → selecionar cartão → voltar.
- [ ] Expandir/recolher lançamentos, transferências e gráficos separadamente.
- [ ] Verificar contador `(n)` de cada menu contra sua lista correspondente.
- [ ] Alterar período e confirmar que as listas/KPIs se atualizam sem alterar saldos históricos indevidamente.
- [ ] Abrir detalhes de lançamentos, investimentos, parcelas e outros níveis; voltar corretamente.
- [ ] Testar navegação e tabelas em viewport estreito e largo.

### 35.2. Testes financeiros

- [ ] Criar despesa pendente em cartão e conferir limite usado.
- [ ] Marcar despesa como paga/cancelada e verificar o impacto na composição correspondente.
- [ ] Cadastrar despesa em conta corrente/dinheiro e validar débito único.
- [ ] Reverter status e verificar estorno sem duplicidade.
- [ ] Confirmar recebimento; reverter e verificar crédito/estorno.
- [ ] Confirmar meta com destino e validar recebimento vinculado `metaId`.
- [ ] Criar parcelamento com divisão monetária não exata e conferir centavos.
- [ ] Transferir entre contas; editar e excluir; validar conservação de saldos.
- [ ] Criar renda fixa/variável, registrar movimentos e verificar projeções/custo médio.
- [ ] Excluir e restaurar registros pela lixeira; validar vínculos.

### 35.3. Testes de backup e restauração

- [ ] Exportar JSON e confirmar `formato` e `versaoEstrutura`.
- [ ] Comparar todas as coleções antes/depois de exportar, inclusive campos não exibidos na tela.
- [ ] Importar para painel limpo e validar totais/relacionamentos.
- [ ] Cancelar a prévia sem substituir os dados.
- [ ] Confirmar uma importação e desfazê-la na mesma sessão.
- [ ] Importar JSON inválido, versão futura e coleção com tipo errado: deve rejeitar.
- [ ] Importar backup legado reconhecido, com coleções faltantes: preencher arrays vazios.
- [ ] Verificar limite de 30 MB.
- [ ] Salvar o HTML importado e reabri-lo sem usar `localStorage`.
- [ ] Comparar quantidades, IDs, estados e saldos antes/depois de reabrir.
- [ ] Testar caracteres especiais nas observações e descrições.
- [ ] Exportar novamente após importar e comparar a equivalência do conjunto de dados, descontadas flags de compatibilidade, timestamp e normalizações explicitamente previstas.
- [ ] Verificar que zerar uma cópia HTML não afeta outra cópia independente.

### 35.4. Critérios de aceite de uma futura versão

Uma release está apta a receber backups existentes quando: (a) reconhece o envelope anunciado, (b) converte quaisquer formatos anteriores prometidos, (c) não perde chaves nem operações financeiras, (d) preserva saldos e vínculos após salvar/reabrir, (e) não duplica créditos/débitos, (f) fornece navegação de retorno nas páginas de detalhe e (g) apresenta erros de importação de modo compreensível.

### 35.5. Limite dos testes desta documentação

A documentação foi preparada por leitura do código e inspeção estrutural dos arquivos fornecidos. Os checklists desta seção são **roteiros recomendados**; sua presença não equivale a execução integral de testes em navegador ou auditoria contábil. Testes executados especificamente na geração deste MD, quando houver, serão resumidos no rodapé técnico final.

## 36. Lacunas comprovadas e pontos de atenção

Esta seção evita que o manual perpetue funcionalidades citadas em conversas anteriores, mas não encontradas na versão de HTML efetivamente analisada.

1. **Quitação de fatura agrupada com restauração dos estados anteriores:** não foi localizado, no HTML principal desta entrega, um comando identificado como **“Marcar como Fatura paga”** nem uma rotina específica de cancelamento que guarde individualmente e restaure os status anteriores de todos os itens de uma fatura. Há edição de status de despesas/parcelas e controle de débito por origem, mas isso **não comprova** a presença do fluxo de pagamento de fatura inteira anteriormente solicitado. Deve ser reavaliado antes de documentar como entregue.
2. **Indicadores coloridos de vencimento “Sem Faturas”, “Falta 1 dia” etc.:** esses rótulos e seu helper correspondente não foram identificados no HTML principal usado como fonte deste MD. A tela atual identifica dias de vencimento e fechamento, porém a lógica de badges de prazo anteriormente discutida não deve ser afirmada como presente sem uma nova verificação de código.
3. **Migração automática de schemas futuros:** a aplicação só reconhece formalmente `versaoEstrutura:1` e estrutura bruta legada; `migrarV1ParaV2()` é uma sugestão, não código existente.
4. **Importação com mesclagem:** não implementada; a operação substitui o conjunto completo.
5. **Validação referencial/contábil exaustiva:** o importador verifica forma e propriedades básicas, não todas as regras financeiras e relações.
6. **Backups criptografados, agendados ou na nuvem:** não identificados.
7. **Histórico persistente de desfazer importações:** existe apenas a reversão da última importação na mesma sessão.
8. **Fonte de dados externa para investimentos:** não identificada; avaliações dependem de parâmetros e cotações registradas localmente.

Se alguma dessas funcionalidades for adicionada em release posterior, atualizar esta seção **somente após verificar o novo código**.

## 37. Glossário e convenções

| Termo | Significado neste painel |
|---|---|
| **Backup JSON** | Fotografia versionada do objeto `dados` sem a interface |
| **HTML autocontido** | Aplicação e snapshot de dados no mesmo arquivo |
| **Drill-down** | Navegação da visão agregada para página interna de detalhes |
| **Bloco retrátil** | Menu expansível que esconde ou mostra tabelas/gráficos |
| **Fatura nominal** | Soma das cobranças do cartão no mês, independentemente de status no cálculo `faturaCartao` atual |
| **Pendente** | Lançamento ainda não marcado Pago/Cancelado, conforme contexto |
| **Saldo disponível** | Valores em corrente/poupança mais dinheiro físico conforme indicador |
| **Patrimônio investido** | Valor estimado/registrado das aplicações, distinto de caixa livre |
| **Recorrência** | Série temporal de ocorrências |
| **Parcelamento** | Compra de cartão desdobrada em cobranças mensais vinculadas |
| **Importação por substituição** | Carregar JSON em memória descartando a base de destino da sessão |
| **Migração de schema** | Conversão explícita da estrutura de dados entre versões |
| **Lixeira** | Repositório local de registros removidos logicamente, passível de restauração |
| **BRL / pt-BR** | Moeda e convenções de formatação do painel |

---

## 38. Apêndices técnicos

Os inventários abaixo foram extraídos do HTML principal da versão documentada, com linhas relativas ao arquivo identificado no controle documental. Uma função listada no inventário é uma **entrada de código existente**, não uma garantia automática de que todos os caminhos de execução foram testados.

### Apêndice A — Funções JavaScript declaradas

**Quantidade identificada:** 226 declarações `function` (nomes únicos: 226).

| Função | Linha do HTML |
|---|---:|
| `boot` | 692 |
| `gerarDadosDeExemplo` | 733 |
| `marcarAlteracao` | 774 |
| `limparIndicadorAlteracao` | 775 |
| `salvarDadosLocal` | 778 |
| `mudarMes` | 779 |
| `emPeriodo` | 781 |
| `alterarModo` | 782 |
| `selecionarPeriodo` | 783 |
| `moverPeriodo` | 784 |
| `atualizarLabelMes` | 785 |
| `getStatusData` | 786 |
| `statusInfo` | 795 |
| `statusControle` | 802 |
| `statusControleGrupo` | 822 |
| `recorrenciasDo` | 838 |
| `semSufixoRecorrencia` | 841 |
| `dataOcorrencia` | 842 |
| `atualizarCamposRecorrencia` | 854 |
| `quantasRecorrencias` | 860 |
| `salvarSerie` | 867 |
| `identificarSerie` | 918 |
| `getCatDetails` | 924 |
| `initDragAndDrop` | 930 |
| `getDragAfterElement` | 959 |
| `salvarOrdemCards` | 967 |
| `restaurarOrdemCards` | 971 |
| `garantirTopoVisivel` | 986 |
| `renderVisaoBase` | 1002 |
| `rotuloDestinoVoltar` | 1060 |
| `injetarVoltarDrilldown` | 1080 |
| `header` | 1093 |
| `renderPrincipal` | 1105 |
| `renderCorrespondencias` | 1152 |
| `renderCalendario` | 1164 |
| `renderSimple` | 1186 |
| `campoDetalhe` | 1291 |
| `grupoParcelamentoDaDespesa` | 1340 |
| `localizarParcelaNoPeriodo` | 1347 |
| `rotuloParcelaDespesa` | 1357 |
| `renderDetalheDespesa` | 1364 |
| `abrirModalDespesa` | 1389 |
| `renderDetalheRecebimento` | 1422 |
| `abrirModalRecebimento` | 1439 |
| `renderDetalhePlan` | 1478 |
| `abrirModalPlan` | 1484 |
| `fecharERecarregar` | 1503 |
| `renderCategorias` | 1508 |
| `abrirModalCategoria` | 1512 |
| `selecionarIcone` | 1513 |
| `showToast` | 1519 |
| `setFilter` | 1527 |
| `openModal` | 1528 |
| `closeModal` | 1529 |
| `refresh` | 1549 |
| `contagensBackup` | 1587 |
| `criarConteudoBackup` | 1594 |
| `conferirSegurancaBackup` | 1618 |
| `prepararImportacaoJSON` | 1629 |
| `resumoImportacaoHTML` | 1667 |
| `nomeArquivoAtual` | 1730 |
| `graficoBarras` | 1762 |
| `agrupar` | 1766 |
| `graficoDaVisao` | 1767 |
| `faturaCartao` | 1776 |
| `reservaFuturaCartao` | 1779 |
| `usadoLimiteCartao` | 1780 |
| `totalUsadoLimite` | 1781 |
| `totalFaturasMes` | 1782 |
| `corDisponibilidade` | 1783 |
| `legendaLimites` | 1787 |
| `pctCartao` | 1790 |
| `graficoFaturasCartoes` | 1791 |
| `cartoesKpi` | 1798 |
| `renderListaCartoes` | 1802 |
| `renderDetalheIndicadorCartoes` | 1816 |
| `renderListaContas` | 1832 |
| `tabelaLancamentos` | 1834 |
| `renderDetalheCartao` | 1839 |
| `gruposParcelas` | 1878 |
| `renderDetalheParcelamento` | 1881 |
| `adicionarExemplosV13` | 1887 |
| `aplicarTema` | 1893 |
| `alternarTema` | 1894 |
| `garantirModulos` | 1935 |
| `retirarColecao` | 1945 |
| `retirarMovimentoInvestimento` | 1952 |
| `criarEntradaLixeira` | 1958 |
| `adicionarRegistroLixeira` | 1965 |
| `ativoContemRegistro` | 1971 |
| `chaveContidaNaEntrada` | 1982 |
| `restaurarEntradaCore` | 1988 |
| `renderLixeira` | 2040 |
| `tiposDeInv` | 2095 |
| `tipoDef` | 2100 |
| `taxaEfetivaInv` | 2108 |
| `aplicarFluxosFixos` | 2115 |
| `invAnalise` | 2123 |
| `resumoCarteira` | 2143 |
| `botaoInv` | 2150 |
| `agruparInvPorTipo` | 2157 |
| `linhaTabelaInv` | 2166 |
| `tabelaInv` | 2170 |
| `renderInvestimentos` | 2171 |
| `renderInvestClasse` | 2181 |
| `renderInvestTipo` | 2197 |
| `renderInvestLista` | 2205 |
| `renderDetalheInvestimento` | 2206 |
| `renderInvExtras` | 2224 |
| `popularTiposInv` | 2233 |
| `abrirInvestimento` | 2240 |
| `salvarInvestimento` | 2253 |
| `abrirNovoTipo` | 2266 |
| `salvarNovoTipo` | 2272 |
| `atualizarCamposMovInvest` | 2283 |
| `calcularValorMovInvest` | 2294 |
| `abrirMovInvest` | 2298 |
| `saldoDisponivelMov` | 2309 |
| `salvarMovInvest` | 2314 |
| `excluirMovInvest` | 2327 |
| `excluirInvestimento` | 2332 |
| `contasTransferencia` | 2338 |
| `lerSaldo` | 2343 |
| `definirSaldo` | 2344 |
| `mudarSaldoTransfer` | 2345 |
| `renderTransferencias` | 2346 |
| `abrirTransferencia` | 2354 |
| `salvarTransferencia` | 2364 |
| `excluirTransferencia` | 2382 |
| `transferenciasDaConta` | 2390 |
| `tabelaTransferenciasConta` | 2391 |
| `invSerieHistorica` | 2398 |
| `nMes` | 2430 |
| `renderHistoricoInvestimentos` | 2431 |
| `renderInvestMes` | 2444 |
| `renderInvestProventos` | 2449 |
| `renderVencimentosInvestimentos` | 2454 |
| `adicionarDemonstracaoInvestV17` | 2462 |
| `despesasCartaoPendentesMes` | 2477 |
| `totalFaturasPendentesMes` | 2483 |
| `totalParcelasPendentesMes` | 2484 |
| `popularDestinoMeta` | 2487 |
| `infoDestinoMeta` | 2494 |
| `infoConta` | 2498 |
| `sincronizarMeta` | 2505 |
| `abrirConfirmacaoMeta` | 2516 |
| `proximoVencimentoCartao` | 2585 |
| `garantirContaDinheiroPrincipal` | 2651 |
| `resetRegistros` | 2677 |
| `abrirQuadro` | 2680 |
| `quadroTabela` | 2683 |
| `graficoLinhas` | 2694 |
| `graficoColunas` | 2718 |
| `graficoRosca` | 2730 |
| `mesesContas` | 2748 |
| `itensConta` | 2760 |
| `patrimonioInvEm` | 2780 |
| `serieConta` | 2783 |
| `agregarSeries` | 2798 |
| `desempenho` | 2803 |
| `quadroContaMes` | 2807 |
| `quadroTotalMes` | 2811 |
| `graficoEvolucaoContas` | 2815 |
| `graficoFluxoContas` | 2821 |
| `itensInvestimentoQuadro` | 2826 |
| `roscasInvestimentoContas` | 2829 |
| `roscaContasPorValor` | 2835 |
| `roscaSaidasCategoria` | 2839 |
| `roscaCapitalRendimento` | 2845 |
| `cardTipoConta` | 2851 |
| `linhaMenu` | 2856 |
| `renderContasMenu` | 2859 |
| `renderCartoesMenu` | 2866 |
| `renderContasLista` | 2876 |
| `renderContasTipo` | 2882 |
| `renderDetalheConta` | 2918 |
| `popularContasInv` | 3063 |
| `patrimonioPorContaInv` | 3068 |
| `adicionarDemonstracaoContasV18` | 3086 |
| `roscaGrupos` | 3255 |
| `graficosDespesas` | 3265 |
| `graficosRecebimentos` | 3271 |
| `graficosMetas` | 3276 |
| `graficosContasLista` | 3280 |
| `roscasCarteira` | 3290 |
| `dataCompraGrupo` | 3303 |
| `renderListaParcelamentos` | 3304 |
| `movimentarSaldoOrigem` | 3353 |
| `efetuarDebitoDespesa` | 3357 |
| `estornarDebitoDespesa` | 3362 |
| `agendarOuDebitar` | 3367 |
| `processarDebitosAgendados` | 3371 |
| `despesasUnicasCartao` | 3443 |
| `renderCartoesUnicas` | 3444 |
| `expBloco` | 3465 |
| `expLinhaDe` | 3471 |
| `preencherExp` | 3482 |
| `definirExp` | 3487 |
| `atualizarRotuloExpTudo` | 3503 |
| `injetarToolbarExp` | 3512 |
| `renderVisao` | 3517 |
| `capturarHtml` | 3524 |
| `toolbarAcoes` | 3532 |
| `corpoCapturado` | 3539 |
| `redirecionarParaCard` | 3551 |
| `rolarParaExp` | 3565 |
| `resumoCabecalho` | 3571 |
| `renderContasPagina` | 3572 |
| `renderCartoesPagina` | 3625 |
| `cabecalhoTipoConta` | 3633 |
| `primeiraDataNoDia` | 3649 |
| `movimentarSaldoDestino` | 3750 |
| `creditarRecebimento` | 3754 |
| `estornarCreditoRecebimento` | 3761 |
| `sincronizarCreditoRecebimento` | 3766 |
| `migrarCreditoRecebimentos` | 3768 |
| `novoControle` | 3824 |
| `voltarAoInicio` | 3841 |
| `paiVisualDaTabela` | 3863 |
| `jaTemControleDepoisDo` | 3875 |
| `inserirDepois` | 3879 |
| `precisaRolagemDentroDeModal` | 3886 |
| `inserirEmListasRepetidas` | 3895 |
| `atualizarVisibilidade` | 3907 |
| `atualizarRaiz` | 3929 |
| `agendarAtualizacao` | 3960 |

### Apêndice B — Funções/ações expostas em `window`

Além de declarações tradicionais, o código usa atribuições diretas no objeto global para interações da interface. Inventário de atribuições com função de seta:

| Ação global | Linha do HTML |
|---|---:|
| `window.atualizarStatus` | 808 |
| `window.atualizarStatusGrupo` | 830 |
| `window.alterarEscopoRecorrencia` | 893 |
| `window.selecionarVisao` | 994 |
| `window.navegarVoltar` | 1051 |
| `window.abrirCategoriaPorGrafico` | 1143 |
| `window.alterarExibicaoCalendario` | 1185 |
| `window.setMetric` | 1235 |
| `window.renderEvolution` | 1237 |
| `window.iniciarEdicaoCampo` | 1295 |
| `window.atualizarContasDestinoRecebimento` | 1430 |
| `window.salvarRecebimento` | 1458 |
| `window.salvarPlanejamento` | 1491 |
| `window.excluirItem` | 1497 |
| `window.salvarCategoria` | 1514 |
| `window.excluirLancamento` | 1530 |
| `window.zerarPainel` | 1540 |
| `window.exportarDadosJSON` | 1605 |
| `window.processarArquivoBackup` | 1677 |
| `window.fecharBackupModal` | 1696 |
| `window.confirmarImportacaoJSON` | 1701 |
| `window.desfazerUltimaImportacao` | 1718 |
| `window.salvarArquivo` | 1737 |
| `window.restaurarLixeira` | 2015 |
| `window.excluirDefinitivamenteLixeira` | 2030 |
| `window.esvaziarLixeira` | 2035 |
| `window.excluirConta` | 2052 |
| `window.excluirCartao` | 2071 |
| `window.excluirGrupoInvestimentosIds` | 2079 |
| `window.excluirTipoInvestimento` | 2087 |
| `window.invToggle` | 2153 |
| `window.cancelarConfirmacaoMeta` | 2523 |
| `window.confirmarMeta` | 2524 |
| `window.abrirModalConta` | 2533 |
| `window.salvarConta` | 2538 |
| `window.abrirModalCartao` | 2552 |
| `window.abrirModalEditarCartao` | 2566 |
| `window.salvarCartao` | 2567 |
| `window.ajustarDataParcelamento` | 2592 |
| `window.previewParcelamento` | 2597 |
| `window.abrirModalParcelamento` | 2601 |
| `window.salvarParcelamento` | 2614 |
| `window.abrirQuadroReg` | 2679 |
| `window.fecharQuadro` | 2681 |
| `window.cliqueGrafico` | 2692 |
| `window.cliqueRosca` | 2693 |
| `window.alternarSerie` | 2711 |
| `window.abrirMovConta` | 2962 |
| `window.salvarMovConta` | 2971 |
| `window.excluirMovConta` | 2983 |
| `window.atualizarCamposTipoConta` | 2992 |
| `window.abrirModalConta` | 3003 |
| `window.abrirModalEditarConta` | 3021 |
| `window.salvarConta` | 3022 |
| `window.cliqueEvolucao` | 3077 |
| `window.previewParcelamentoEd` | 3181 |
| `window.abrirModalEditarParcelamento` | 3185 |
| `window.salvarEdicaoParcelamento` | 3199 |
| `window.excluirParcelamento` | 3224 |
| `window.editarParcelamentoDaDespesa` | 3234 |
| `window.barraDragStart` | 3243 |
| `window.barraDragOver` | 3244 |
| `window.barraDragLeave` | 3245 |
| `window.barraDragEnd` | 3246 |
| `window.barraDrop` | 3247 |
| `window.ordenarParcelamentos` | 3301 |
| `window.resetarOrdemParcelamentos` | 3302 |
| `window.previewParcelamento` | 3314 |
| `window.salvarParcelamento` | 3318 |
| `window.excluirCategoria` | 3340 |
| `window.despesaMudouFrequencia` | 3376 |
| `window.atualizarFormularioDespesa` | 3381 |
| `window.salvarDespesa` | 3407 |
| `window.alternarExp` | 3502 |
| `window.alternarTudo` | 3507 |
| `window.header` | 3527 |
| `window.executarAcao` | 3531 |
| `window.despesaMudouFrequencia` | 3655 |
| `window.atualizarFormularioDespesa` | 3661 |
| `window.salvarDespesa` | 3695 |
| `window.salvarRecebimento` | 3777 |

### Apêndice C — Identificadores de janelas modais

| Modal | Uso geral |
|---|---|
| `despesaModal` | Cadastro/edição de despesa |
| `recebimentoModal` | Cadastro/edição de recebimento |
| `planModal` | Meta a Receber |
| `categoriaModal` | Categoria e ícone |
| `contaModal` | Cadastro de conta |
| `cartaoModal` | Cadastro de cartão |
| `parcelamentoModal` | Cadastro de parcelamento |
| `confirmarMetaModal` | Confirmar meta com destino |
| `parcelamentoEdModal` | Edição de parcelamento |
| `quadroModal` | Detalhes de gráficos e indicadores |
| `movContaModal` | Lançamento manual de conta |
| `investimentoModal` | Aplicação financeira |
| `movimentoInvestimentoModal` | Operação de investimento |
| `tipoInvestimentoModal` | Tipo personalizado de investimento |
| `transferenciaModal` | Transferência entre contas |
| `backupModal` | Prévia/confirmação da importação JSON |

### Apêndice D — Constantes e chaves do contrato de backup

```javascript
const BACKUP_FORMATO = 'PainelFinanceiro';
const BACKUP_VERSAO_ESTRUTURA = 1;
const BACKUP_LISTAS = ['contas','cartoes','despesas','recebimentos','planejamentos','categorias',
'investimentos','transferencias','movContas','tiposInvestimento','lixeira','navOrder'];
```

### Apêndice E — Registro de alterações documentais

| Data | Natureza | Resultado |
|---|---|---|
| 2026-11-03 | Revisão total da antiga documentação V17 | Manual reescrito para base real atual, incluindo menus e drill-down, conta/cartão individual, lixeira, status e efeitos em saldos, esquema JSON v1, importação em HTML limpo, salvamento, limitações e critérios de teste. |

**Nota final:** atualizações subsequentes da aplicação exigem revisão das seções funcionais, dos contratos de dados e das lacunas, com novo checksum para os HTMLs.
