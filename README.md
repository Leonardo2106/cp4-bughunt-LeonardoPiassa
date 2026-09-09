# Checkpoint 4 — Bug Hunt StreamFIAP

## Identificação

**Grupo:** LeonardoPiassa

| Integrante | RM | Turma |
|---|---|---|
| Leonardo Piassa | 563663 | 2CCPW |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | `GET /api/conteudos/999` devolvia uma resposta vazia de sucesso, sem explicar que o conteúdo não existia. | `ConteudoController.java`, antigo método `buscarPorId`: um `catch (Exception)` vazio engolia `ConteudoNaoEncontradoException` e o método retornava `null`. | Removi o `try/catch` genérico e deixei a exceção específica chegar ao `GlobalExceptionHandler`, que responde 404 com a mensagem. | Tratamento de exceções específicas e propagação de erros (Aula 11). |
| bug02 | `GET /api/conteudos/categoria/FICCAO` podia retornar lista vazia mesmo havendo conteúdos nessa categoria. | `ConteudoController.java`, antigo `listarPorCategoria`: comparava objetos `String` com `==`, que verifica referência em vez do texto. | Passei a chamar `ConteudoRepository.findByCategoria`, consulta derivada que compara corretamente o valor persistido. | Comparação de objetos e Spring Data JPA (Aulas 1 e 13). |
| bug03 | Um `POST` de conteúdo com `duracaoMinutos` igual a zero ou negativa era aceito e persistido. | `Conteudo.java`, construtor e `setDuracaoMinutos`: o atributo era atribuído sem validar a regra. | Centralizei a validação no setter, usei-o no construtor e criei uma exceção específica tratada como HTTP 400 com mensagem clara. | Encapsulamento, validação de estado e exceções customizadas (Aulas 3, 4 e 11). |
| bug04 | O preço promocional de um filme ficava 20% mais caro (estreia: R$ 17,88) em vez de ter desconto (R$ 11,92). | `Filme.java`, `aplicarPromocao`: multiplicava o preço por `1.2`, aplicando acréscimo. | Alterei o fator para `0.8`, preservando 80% do preço e concedendo os 20% de desconto do contrato. | Interface e implementação de regra de negócio (Aula 9). |
| bug05 | Ao cadastrar uma série, título e categoria ficavam nulos, duração/classificação ficavam zero e ela sempre ficava indisponível. | `Serie.java`, construtor: não chamava `super(...)`; `ConteudoController` também não repassava `disponivel`. | O construtor agora encaminha todos os dados comuns à superclasse, e o controller repassa a disponibilidade recebida. | Herança e encadeamento de construtores (Aulas 4 e 6). |
| bug06 | Uma série de 5 temporadas custava R$ 9,90 em vez de R$ 24,50. | `Serie.java`, `calcularPrecoAluguel(double)`: o parâmetro extra criava uma sobrecarga, portanto chamadas polimórficas usavam a implementação de `Conteudo`. | Corrigi a assinatura para `calcularPrecoAluguel()` e adicionei `@Override`, fazendo o preço de R$ 4,90 por temporada ser usado. | Polimorfismo por sobrescrita versus sobrecarga (Aula 7). |
| bug07 | Um documentário era alugado por R$ 9,90, apesar de o contrato defini-lo como gratuito. | `Documentario.java`: não sobrescrevia `calcularPrecoAluguel` e herdava o preço genérico de `Conteudo`. | Implementei a sobrescrita retornando R$ 0,00; ele continua fora de promoções por não implementar `Promocionavel`. | Herança e polimorfismo por sobrescrita (Aulas 6 a 9). |
| bug08 | `POST /api/usuarios` tentava salvar um usuário sem ID, podendo falhar na persistência em vez de devolver o ID gerado. | `Usuario.java`, atributo `id`: tinha `@Id`, mas não declarava estratégia de geração. | Adicionei `@GeneratedValue(strategy = GenerationType.IDENTITY)`, deixando o banco gerar a chave como já ocorria em `Conteudo`. | Mapeamento JPA e persistência de entidades (Aula 13). |
| bug09 | O usuário era cadastrado com `nome: null`, embora o nome tivesse sido enviado no corpo da requisição. | `Usuario.java`, construtor: `nome = nome` atribuía o parâmetro a ele mesmo e não alterava o atributo. | Troquei por `this.nome = nome`, diferenciando o atributo da instância do parâmetro. | Estado de objetos, construtores e uso de `this` (Aulas 1 e 4). |
| bug10 | Usuários sem saldo conseguiam alugar e ficar com créditos negativos, enquanto usuários com saldo suficiente eram recusados. | `Usuario.java`, `temCreditosSuficientes`: a comparação estava invertida (`preco >= creditos`). | Corrigi a condição para verificar se `creditos >= preco` antes do débito. | Método de comportamento e regra de negócio no model (Aula 2). |
| bug11 | Um conteúdo marcado como indisponível ainda podia ser alugado e ter seu preço debitado. | `Usuario.java`, `alugar`: não verificava `Conteudo.isDisponivel()` antes das demais operações. | Incluí a validação no início do método e lancei a `ConteudoIndisponivelException` já tratada pela API. | Encapsulamento da regra de negócio no model e exceções (Aulas 2 e 11). |
| bug12 | O aluguel por usuário abaixo da classificação resultava em erro genérico do servidor, sem a mensagem útil da regra. | `ClassificacaoIndicativaException.java` e `GlobalExceptionHandler.java`: era checked e não possuía handler específico para converter a falha em resposta da API. | Tornei a exceção unchecked, removi os `throws` desnecessários e adicionei um handler que devolve HTTP 403 com sua mensagem. | Exceções checked/unchecked e tratamento global no Spring (Aulas 11 e 13). |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `Conteudo.duracaoMinutos` e acessos diretos no `ConteudoController`. | O atributo era público, quebrando o encapsulamento e permitindo alterações que ignorassem sua validação. | Tornei o atributo privado e substituí os acessos diretos pelo getter. |
| clean02 | `Usuario.alugar`, nos nomes `c` e `p`. | Nomes de uma letra escondiam o papel do conteúdo e do preço na principal regra de negócio. | Renomeei-os para `conteudo` e `precoAluguel` em todo o método. |
| clean03 | Final de `ConteudoController`. | Havia um método de desconto antigo nunca chamado e um bloco de cupom comentado, aumentando ruído e sugerindo regras que não pertencem ao contrato atual. | Removi o código morto; o histórico do Git continua disponível caso uma regra futura precise ser recuperada. |
| clean04 | `Conteudo.calcularPrecoAluguel`. | A classe abstrata fornecia um preço padrão de filme para tipos diferentes, embora cada subtipo tenha uma regra própria. | Transformei o método em abstrato, obrigando cada classe concreta a declarar seu preço e deixando o contrato polimórfico explícito. |
| clean05 | `Usuario.alugar`. | O método misturava validação e mudança do estado do aluguel com os detalhes de formatação e impressão do recibo. | Extraí a impressão para `emitirRecibo`, deixando o fluxo principal curto e em um único nível de abstração, sem mudar sua saída. |
| clean06 | Campos de repository dos três controllers. | A injeção em campos mutáveis com `@Autowired` escondia dependências obrigatórias e dificultava instanciar os controllers em testes. | Adotei injeção por construtor e campos `final`; o Spring usa automaticamente o único construtor, sem precisar de `@Autowired`. |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
`ConteudoRepository` é uma interface, portanto nem sequer existe uma implementação concreta que
o controller possa criar diretamente com `new`. Na inicialização, o Spring Data gera um proxy que
implementa essa interface, registra esse objeto como bean e configura nele o acesso JPA ao banco.
Ao construir `ConteudoController`, o contêiner encontra o único construtor e entrega o bean compatível.
O mesmo ocorre com os dois repositories exigidos por `AluguelController`. Com `new` no controller,
perderíamos esse proxy gerenciado, sua configuração, transações e a possibilidade de substituir a
dependência em testes. Os campos `final` ainda deixam claro que ela é obrigatória e não muda depois.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
No JDBC/DAO, nosso código abre `Connection`, prepara SQL, associa parâmetros, percorre o
`ResultSet`, converte cada linha em objeto e fecha recursos. O Spring Data JPA fornece tudo isso
para o CRUD ao fazer `ConteudoRepository` estender `JpaRepository<Conteudo, Long>` e usa as
anotações das entidades para mapear objetos e tabelas. `findByCategoria` funciona porque o Spring
interpreta o nome do método na inicialização, identifica a propriedade `categoria` e gera a consulta.
Essa abstração reduz repetição e atende bem às operações comuns do StreamFIAP. JDBC direto ainda
pode ser melhor quando precisamos de SQL muito específico, controle fino ou otimização particular.

### 3. Exceções checked vs unchecked (Aula 11)
Quando uma exceção estende `Exception`, ela é checked: quem chama precisa capturá-la ou declará-la
com `throws`. Isso obrigava `Usuario.alugar` e o endpoint a carregar uma preocupação de compilação,
mas não criava por si só uma resposta HTTP útil. Uma exceção que estende `RuntimeException` é
unchecked e pode atravessar naturalmente as camadas até o mecanismo global do Spring. Alterei
`ClassificacaoIndicativaException` para unchecked e criei um método específico no
`GlobalExceptionHandler`. Assim a mensagem montada pelo model chega no campo `erro` de uma resposta
HTTP 403, em vez de aparecer apenas como erro genérico 500.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Sobrescrita mantém a assinatura do método herdado e troca sua implementação no subtipo. Sobrecarga
cria outro método com o mesmo nome, porém com parâmetros diferentes. `Conteudo` declarava
`calcularPrecoAluguel()`, enquanto `Serie` tinha `calcularPrecoAluguel(double desconto)`; por isso o
código compilava, mas uma referência `Conteudo` chamava a versão herdada de R$ 9,90. Removi o
parâmetro que não era usado e marquei o método com `@Override`, fazendo uma série de cinco temporadas
custar R$ 24,50. Se a assinatura errada tivesse `@Override` desde o início, o compilador informaria
que não havia método correspondente na superclasse e impediria esse bug silencioso.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
A duração pertence ao estado de qualquer `Conteudo`, então sua validação foi centralizada em
`setDuracaoMinutos`; o construtor chama esse setter para que o objeto já nasça válido e mudanças
posteriores também sejam protegidas. Validar somente no controller deixaria outros pontos de criação
livres para gravar duração zero ou negativa. Os campos nulos da `Serie` tinham outra causa: seu
construtor não encaminhava os dados ao construtor de `Conteudo`, corrigido com `super(...)`.
Já créditos negativos eram consequência de uma regra de comportamento invertida; a defesa correta
fica em `Usuario.alugar`, antes de `debitarCreditos`. Assim cada invariável fica junto do estado ou
da operação que realmente a governa, sem depender exclusivamente da camada HTTP.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` representa a base comum da hierarquia: concentra identidade, título, categoria, duração,
classificação e disponibilidade, além de exigir o cálculo polimórfico do preço. `Promocionavel`
representa uma capacidade independente da herança: somente os tipos que assinam esse contrato têm
`aplicarPromocao`. Se documentários passassem a participar, bastaria declarar `implements
Promocionavel` em `Documentario` e implementar ali o desconto. `Conteudo.calcularPrecoPromocional`,
os controllers, os repositories, `Filme` e `Serie` permaneceriam intactos, pois o método já consulta
o contrato com `instanceof`. Isso mostra baixo acoplamento: uma capacidade pode ser adicionada a um
subtipo sem espalhar condicionais por toda a aplicação.

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
