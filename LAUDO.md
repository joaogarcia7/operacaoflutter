# Laudo Técnico - Operação Resgate

## Bug 1
* **Bug:** 1
* **Arquivo:** `lib/pages/central_page.dart`
* **Sintoma:** O aplicativo não compila/executa pois há falha ao instanciar o widget de detalhe.
* **Causa:** O construtor da classe `DetalhePage` requer o parâmetro `ocorrencia`, porém estava sendo instanciado vazio.
* **Correção:** Alterado para passar o argumento `DetalhePage(ocorrencia: o)`.
* **Teste:** Confirmado que o app compila corretamente e abre a tela de detalhes sem erros.

## Bug 2
* **Bug:** 2
* **Arquivo:** `lib/pages/central_page.dart`
* **Sintoma:** Erro de sintaxe na formatação do subtítulo da ocorrência.
* **Causa:** Quebra de linha literal no meio de uma string simples. No Dart, aspas simples não suportam quebra de linha visual.
* **Correção:** Trocada a quebra de linha visual pelo caractere `\n` dentro da string.
* **Teste:** Execução bem-sucedida da listagem das ocorrências.

## Bug 3
* **Bug:** 3
* **Arquivo:** `lib/pages/central_page.dart`
* **Sintoma:** Permitia salvar ocorrências vazias (apenas com o título ou apenas com o local), burlando a intenção de ser obrigatório.
* **Causa:** A lógica `if (titulo.isEmpty && local.isEmpty)` barrava apenas se os DOIS estivessem vazios (AND em vez de OR).
* **Correção:** Trocado `&&` por `||` para obrigar o preenchimento de ambos.
* **Teste:** Tentativa de preencher apenas o local exibiu o Snackbar de aviso corretamente.

## Bug 4
* **Bug:** 4
* **Arquivo:** `lib/pages/central_page.dart`
* **Sintoma:** O botão switch para filtrar as ocorrências travava e não alternava a posição.
* **Causa:** O método `onChanged` do `SwitchListTile` estava aplicando `!value` na atribuição do `setState`, invertendo o novo estado do botão, e essencialmente impedindo sua mudança.
* **Correção:** Removida a negação: `somenteAbertos = value;`.
* **Teste:** O switch agora anima e mantém o estado selecionado.

## Bug 5
* **Bug:** 5
* **Arquivo:** `lib/pages/central_page.dart`
* **Sintoma:** Ao marcar "Somente não resolvidos", o sistema mostrava apenas os itens RESOLVIDOS.
* **Causa:** O filtro `.where` inspecionava se a ocorrência estava resolvida (`o.resolvido`), buscando um resultado oposto ao título do filtro.
* **Correção:** Inserida a negação lógica para filtrar: `!o.resolvido`.
* **Teste:** Criar um ticket fechado e outro aberto, alternar o filtro e validar que apenas o aberto aparece.

## Bug 6
* **Bug:** 6
* **Arquivo:** `lib/models/ocorrencia.dart`
* **Sintoma:** Permissões baseadas em conta não funcionavam no longo prazo porque a identidade não era corretamente obtida do documento (Firebase).
* **Causa:** No factory `fromDoc`, o atributo `autorId` recebia o dado de `d['autorNome']` em vez da chave correta correspondente ao ID.
* **Correção:** Substituído `d['autorNome']` para mapear de `d['autorId']`.
* **Teste:** Feito logout e login e visualizado que apenas o dono vê o botão para alterar o próprio status.

## Bug 7
* **Bug:** 7
* **Arquivo:** `lib/models/ocorrencia.dart`
* **Sintoma:** Um ticket gravado no Firebase como "em aberto" (false) aparecia como resolvido na interface e vice-versa.
* **Causa:** Ao construir o objeto, havia uma negação lógica incorreta atribuindo o status: `!(d['resolvido'])`.
* **Correção:** Removida a negação `!` na inicialização da propriedade `resolvido`.
* **Teste:** Observado a correta paridade entre os dados do Firebase e a cor/etiqueta na listagem.

## Bug 8
* **Bug:** 8
* **Arquivo:** `lib/services/central_service.dart`
* **Sintoma:** O autor verdadeiro do ticket esbarrava na barreira de segurança ao tentar alterar o status (lançava exceção StateError).
* **Causa:** A validação para alterar status faz checagem contra o UID da conta (`u.uid`), porém a função `registrar()` estava salvando o Email (`u.email`) como identificador do autor.
* **Correção:** Na função `registrar`, o dicionário que vai para o Firebase agora recebe `'autorId': u.uid`.
* **Teste:** Um novo chamado criado na conta A e posteriormente solucionado na mesma conta validou a alteração com sucesso.

## Bug 9
* **Bug:** 9
* **Arquivo:** `lib/services/central_service.dart`
* **Sintoma:** Marcar uma ocorrência como resolvida não surtia nenhum efeito de mudança real.
* **Causa:** A chamada `.update` para o Firestore atribuía novamente o status atual (`item.resolvido`) ao invés do status desejado.
* **Correção:** A chamada `update` agora envia a chave `!item.resolvido` para aplicar a alternância.
* **Teste:** Ocorreu a atualização na UI após clicar em "Marcar como resolvido".

## Bug 10
* **Bug:** 10
* **Arquivo:** `lib/services/central_service.dart`
* **Sintoma:** Uma pista enviada para a ocorrência X era vinculada e mostrada na ocorrência Y ou simplesmente não aparecia em lugar algum correlato (mistura total de conversas).
* **Causa:** A rota do Firebase na função `enviarPista` setava o ID do documento da coleção estaticamente como `'pistas'`, ou seja, sobrescrevia sempre no mesmo lugar.
* **Correção:** O documento referenciado passou a ser `ocorrenciaId` passado no argumento da função.
* **Teste:** Comentário enviado apenas apareceu para o item correspondente.

## Bug 11
* **Bug:** 11
* **Arquivo:** `lib/pages/detalhe_page.dart`
* **Sintoma:** Não era possível enviar pistas ao digitar um texto (a operação não ocorria). Só era ativado com o campo vazio.
* **Causa:** Condicional invertida que cancelava o fluxo caso o input não estivesse vazio (`if (pista.text.trim().isNotEmpty) return;`).
* **Correção:** A interrupção mudou para ocorrer quando o texto estivesse `.isEmpty`.
* **Teste:** Ao digitar algo, a pista passa da validação e dispara pro backend.

## Bug 12
* **Bug:** 12
* **Arquivo:** `lib/pages/detalhe_page.dart`
* **Sintoma:** Mesmo com a validação arrumada, ao digitar "Vi na quadra", o Firebase recebia um comentário em branco.
* **Causa:** Na chamada para `service.enviarPista()`, o texto enviado estava hardcoded como `''` (vazio).
* **Correção:** Passado o valor capturado pelo controller: `pista.text.trim()`.
* **Teste:** Visualizado o texto corretamente listado na aba da comunidade usando uma Conta B.
