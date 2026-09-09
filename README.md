# Checkpoint 4 — Bug Hunt StreamFIAP

## Identificação

**Grupo:** LeonardoPiassa

| Integrante | RM | Turma |
|---|---|---|
| Leonardo Piassa | 563663 | 2CCPW |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 1 / 6 |

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
| clean02 | | | |
| clean03 | | | |
| clean04 | | | |
| clean05 | | | |
| clean06 | | | |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
