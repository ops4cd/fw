## Operations for Composite Destination (Ops4CD)

### Resumo

Propomos uma interpretação técnico-operacional de **Operations for Composite Destination (Ops 4 CD)** aplicada à execução local de modelos de linguagem em formato **GGUF**. Como a expressão “Ops 4 CD” não aparece, nas fontes consultadas, como uma denominação consolidada de uma metodologia específica para modelos de linguagem, o conceito é tratado aqui como uma **arquitetura de operações para destinos compostos**: uma camada capaz de receber uma solicitação, decompor seus objetivos, encaminhar diferentes partes para modelos ou ferramentas locais e recompor os resultados em uma resposta única, auditável e contextualizada.

A ideia central é aproveitar a lógica de um *composite destination*: um destino lógico que representa vários destinos físicos e permite uma operação única sobre eles. Em sistemas de mensageria, por exemplo, uma única mensagem pode ser encaminhada para múltiplas filas ou tópicos; o ActiveMQ documenta essa possibilidade por meio de destinos compostos, inclusive com combinação de filas e tópicos em uma única operação lógica. No campo dos modelos locais, essa lógica pode ser reinterpretada como um **roteador-orquestrador** que distribui tarefas entre modelos GGUF especializados, ferramentas de recuperação documental, módulos de avaliação e sistemas de registro. [activemq.apache](https://activemq.apache.org/components/classic/documentation/composite-destinations)

O relatório defende que Ops 4 CD pode oferecer ganhos de privacidade, modularidade, resiliência, controle de custos e adequação pedagógica, especialmente em ambientes educacionais, institucionais e de pesquisa. Entretanto, sua adoção exige governança dos modelos, controle de contexto, avaliação de qualidade, segurança local e mecanismos explícitos de rastreabilidade.

**Palavras-chave:** modelos de linguagem locais; GGUF; llama.cpp; Ollama; orquestração; destinos compostos; agentes locais; educação; privacidade; avaliação.

***

## 1. Introdução

A expansão dos modelos de linguagem generativa modificou práticas de produção textual, programação, tutoria, análise documental e automação. Porém, grande parte das soluções disponíveis depende de APIs externas, cobrança por uso, conexão permanente e envio de dados para servidores de terceiros. Em contextos educacionais, administrativos, científicos e governamentais, essas condições podem ser incompatíveis com requisitos de privacidade, soberania tecnológica e proteção de dados.

O formato GGUF tornou-se importante nesse cenário por reunir, em um arquivo portátil, pesos do modelo, dados de tokenização, metadados arquiteturais e informações de quantização. O formato é utilizado por ferramentas como llama.cpp, Ollama, LM Studio, GPT4All, Jan e KoboldCpp. Sua principal contribuição não é apenas reduzir o tamanho do modelo, mas possibilitar uma cadeia de execução local relativamente padronizada. [datacamp](https://www.datacamp.com/tutorial/gguf-format-a-complete-guide)

Um modelo de linguagem local, contudo, não constitui sozinho um sistema inteligente completo. Ele precisa de:

- seleção de modelo;
- gerenciamento de memória;
- controle de contexto;
- acesso eventual a documentos;
- ferramentas externas;
- validação das respostas;
- registro das operações;
- mecanismos de segurança;
- interface de interação.

É nesse ponto que se situa a proposta de Ops 4 CD. A expressão é compreendida, neste relatório, como uma abordagem de **operações para um destino composto**, em que a solicitação do usuário não é necessariamente enviada a um único modelo, mas distribuída entre múltiplos componentes locais, cujas saídas são integradas posteriormente.

***

## 2. Delimitação conceitual

### 2.1. Destino composto

Em uma arquitetura tradicional, existe um fluxo simples:

\[
\text{usuário} \rightarrow \text{modelo} \rightarrow \text{resposta}
\]

Em uma arquitetura de destino composto, o fluxo é ampliado:

\[
\text{usuário} \rightarrow \text{orquestrador} \rightarrow
\{D_1,D_2,D_3,\ldots,D_n\}
\rightarrow \text{agregador} \rightarrow \text{resposta}
\]

Cada \(D_i\) representa um destino operacional. Ele pode ser:

- um modelo de linguagem GGUF;
- um modelo especializado em código;
- um mecanismo de embeddings;
- um banco vetorial;
- um analisador de documentos;
- um verificador de consistência;
- um sistema de conversão de voz;
- uma ferramenta de execução controlada;
- um módulo de avaliação.

A expressão “destino” não precisa significar apenas um endereço de rede. Ela pode representar qualquer componente que receba uma tarefa e devolva um resultado estruturado.

### 2.2. Operações

O termo *operations* designa o conjunto de procedimentos necessários para administrar o ciclo de vida da solicitação:

1. receber a solicitação;
2. identificar sua intenção;
3. decompor o problema;
4. selecionar os destinos apropriados;
5. executar as tarefas;
6. controlar falhas;
7. combinar os resultados;
8. avaliar a resposta;
9. registrar evidências e parâmetros;
10. devolver o resultado ao usuário.

Assim, Ops 4 CD não deve ser reduzido a uma simples técnica de roteamento. Trata-se de uma camada de **governança operacional da inferência distribuída localmente**.

### 2.3. Relação com composição

O conceito também possui uma analogia com operações de composição gráfica. Em computação visual, composição significa combinar um elemento com seu fundo, distinguindo fonte, destino, sobreposição, transparência e regras de mistura. A especificação do W3C descreve a composição como a combinação de um elemento gráfico com seu *backdrop* e apresenta operadores que determinam como fonte e destino participam do resultado. [w3](https://www.w3.org/TR/2012/WD-compositing-20120816/)

A analogia com LLMs é útil:

| Composição gráfica | Ops 4 CD para LLMs |
|---|---|
| Fonte | Saída de um modelo ou ferramenta |
| Destino | Contexto, documento ou resultado acumulado |
| Backdrop | Estado conversacional e evidências disponíveis |
| Operador de composição | Regra de seleção, fusão ou prioridade |
| Grupo isolado | Subtarefa com contexto restrito |
| Composição final | Resposta integrada ao usuário |

A analogia não é literal. Modelos de linguagem não combinam respostas como pixels. Porém, a distinção entre **resultado produzido**, **contexto de destino** e **regra de composição** ajuda a projetar sistemas mais previsíveis.

***

## 3. GGUF e a infraestrutura local

### 3.1. Características do formato

GGUF é um formato binário voltado à execução de modelos de linguagem locais, especialmente no ecossistema llama.cpp. Um arquivo GGUF normalmente contém:

- tensores do modelo;
- pesos quantizados ou não quantizados;
- vocabulário;
- configuração do tokenizador;
- metadados da arquitetura;
- parâmetros de contexto;
- dimensões de incorporação;
- informações sobre atenção;
- dados de quantização. [datacamp](https://www.datacamp.com/tutorial/gguf-format-a-complete-guide)

A estrutura autocontida facilita a distribuição e reduz a dependência de conjuntos fragmentados de arquivos. O formato também favorece o *memory mapping*, permitindo que o sistema operacional mapeie o arquivo de modelo sem necessariamente copiar todo o conteúdo para a memória de uma só vez.

### 3.2. Quantização

A quantização reduz a precisão numérica dos pesos para diminuir o uso de memória e ampliar a possibilidade de execução em computadores pessoais. Entre as opções mais comuns encontram-se:

- Q2_K;
- Q3_K_M;
- Q4_K_M;
- Q5_K_M;
- Q6_K;
- Q8_0.

De modo geral, Q4_K_M constitui um ponto de partida equilibrado para uso geral, enquanto Q5_K_M e Q6_K podem ser preferíveis quando se busca maior fidelidade em programação, raciocínio ou geração estruturada. A escolha não deve considerar apenas o tamanho do arquivo: também é necessário reservar memória para o sistema operacional, buffers temporários e cache de contexto, conhecido como KV cache. [datacamp](https://www.datacamp.com/tutorial/gguf-format-a-complete-guide)

Uma estimativa simplificada da memória necessária é:

\[
M_{\text{total}} \approx M_{\text{modelo}}+
M_{\text{KV}}+
M_{\text{buffers}}+
M_{\text{sistema}}
\]

O componente \(M_{\text{KV}}\) cresce com o tamanho do contexto, o número de camadas, a dimensão do modelo e a precisão utilizada no cache. Por isso, um modelo que funciona com 4.096 tokens pode apresentar lentidão ou falha com 32.000 tokens.

### 3.3. Runtimes locais

Os principais runtimes utilizados em arquiteturas Ops 4 CD são:

- **llama.cpp:** oferece execução por linha de comando, servidor local, quantização, avaliação e múltiplos backends de hardware.
- **Ollama:** simplifica o gerenciamento de modelos, a criação de configurações e a exposição de uma API local.
- **LM Studio:** oferece interface gráfica e servidor compatível com aplicações locais.
- **GPT4All, Jan e KoboldCpp:** oferecem experiências alternativas para interação e integração com modelos GGUF.

O llama.cpp pode executar modelos com CPU, CUDA, Metal, Vulkan, HIP/ROCm e outros backends, conforme a plataforma e a compilação utilizada. O ecossistema permite combinar CPU e GPU por meio do descarregamento de camadas.

***

## 4. Arquitetura proposta para Ops 4 CD

### 4.1. Visão geral

Uma implementação de referência pode ser organizada em oito camadas:

```text
[Usuário ou aplicação]
          |
          v
[Gateway local / API]
          |
          v
[Classificador e planejador]
          |
          +------------------+
          |                  |
          v                  v
 [Destino textual]     [Destino documental]
          |                  |
          v                  v
 [Modelo GGUF A]       [RAG / arquivos locais]
          |                  |
          +--------+---------+
                   |
                   v
          [Compositor / avaliador]
                   |
                   v
          [Resposta e auditoria]
```

### 4.2. Destinos possíveis

Um conjunto de destinos pode ser configurado da seguinte forma:

| Destino | Função | Exemplo |
|---|---|---|
| \(D_1\) | Conversação geral | Modelo instruído de 7B–14B |
| \(D_2\) | Programação | Modelo especializado em código |
| \(D_3\) | Documentos | Recuperação de trechos locais |
| \(D_4\) | Classificação | Modelo pequeno e rápido |
| \(D_5\) | Verificação | Modelo independente para crítica |
| \(D_6\) | Síntese | Modelo responsável pela resposta final |
| \(D_7\) | Ferramentas | Scripts, calculadoras e consultas locais |
| \(D_8\) | Auditoria | Registro de prompts, modelos e resultados |

O sistema não precisa executar todos os destinos em cada solicitação. O orquestrador deve acionar apenas os componentes necessários.

### 4.3. Tipos de operação

#### Roteamento seletivo

A solicitação é enviada a um único destino, conforme sua natureza.

Exemplo:

- pergunta pedagógica → modelo geral;
- geração de código → modelo de programação;
- consulta a legislação armazenada localmente → destino documental;
- transcrição → modelo de áudio, não um LLM textual.

#### Roteamento paralelo

A mesma solicitação ou subtarefas diferentes são enviadas simultaneamente a múltiplos destinos.

Exemplo:

- um modelo produz uma resposta;
- outro verifica possíveis erros;
- um sistema documental recupera evidências;
- o compositor gera a versão final.

#### Roteamento em cascata

O resultado de uma etapa alimenta a próxima:

\[
D_1 \rightarrow D_2 \rightarrow D_3
\]

Exemplo:

1. \(D_1\) identifica os conceitos centrais;
2. \(D_2\) pesquisa documentos locais;
3. \(D_3\) redige uma resposta fundamentada.

#### Roteamento com fallback

Se o destino principal falhar, o sistema aciona outro modelo ou reduz a complexidade da tarefa:

\[
D_{\text{principal}} \rightarrow
\begin{cases}
\text{sucesso} & \text{resposta normal}\\
\text{falha} & D_{\text{reserva}}
\end{cases}
\]

Isso é especialmente útil em computadores com memória limitada ou quando uma arquitetura específica não é suportada pelo runtime.

#### Roteamento por especialização

Modelos diferentes assumem papéis diferentes. Um modelo geral pode ser mais adequado para linguagem natural, enquanto outro apresenta melhor desempenho em código, matemática ou classificação.

A especialização não garante superioridade automática. Ela precisa ser verificada empiricamente em tarefas representativas do contexto de uso.

***

## 5. Protocolo operacional

### 5.1. Etapa 1 — Normalização

O sistema recebe a solicitação e converte seus elementos em uma estrutura interna:

```json
{
  "request_id": "2026-001",
  "language": "pt-BR",
  "task": "elaboração de relatório",
  "domain": "educação e tecnologia",
  "privacy": "local_only",
  "requested_format": "documento acadêmico",
  "context_limit": 12000
}
```

Essa etapa evita que cada modelo receba a solicitação em formato diferente.

### 5.2. Etapa 2 — Classificação

Um classificador leve determina:

- tipo de tarefa;
- nível de complexidade;
- necessidade de documentos;
- necessidade de ferramentas;
- sensibilidade dos dados;
- necessidade de validação adicional.

Uma regra simples poderia ser:

```javascript
function chooseDestinations(request) {
  const destinations = [];

  if (request.requiresDocuments) destinations.push("retrieval");
  if (request.task === "code") destinations.push("coder");
  if (request.task === "classification") destinations.push("small-classifier");
  destinations.push("general-model");

  return destinations;
}
```

Em aplicações educacionais, a classificação pode também considerar faixa etária, disciplina, objetivo de aprendizagem e nível de autonomia esperado do estudante.

### 5.3. Etapa 3 — Planejamento

O planejador converte a solicitação em subtarefas:

```json
{
  "subtasks": [
    {
      "id": "t1",
      "instruction": "Identificar conceitos fundamentais",
      "destination": "general-model"
    },
    {
      "id": "t2",
      "instruction": "Recuperar documentos relevantes",
      "destination": "retrieval"
    },
    {
      "id": "t3",
      "instruction": "Avaliar riscos e limitações",
      "destination": "critic-model"
    }
  ]
}
```

O planejamento deve conter critérios de conclusão. Sem esses critérios, a arquitetura pode gerar múltiplas respostas sem saber quando interromper o processo.

### 5.4. Etapa 4 — Execução

Cada destino recebe:

- identificador da tarefa;
- instrução;
- contexto autorizado;
- limite de tokens;
- temperatura;
- modelo GGUF selecionado;
- tempo máximo;
- política de retorno.

A execução precisa ser observável. Devem ser registrados:

- nome do modelo;
- quantização;
- runtime;
- parâmetros;
- duração;
- quantidade de tokens;
- erros;
- origem dos documentos;
- versão do prompt.

### 5.5. Etapa 5 — Composição

O compositor deve combinar os resultados sem apagar conflitos importantes. Há pelo menos quatro estratégias:

#### Seleção

Escolher a resposta que satisfaz melhor os critérios de avaliação.

#### Fusão

Combinar trechos provenientes de diferentes destinos.

#### Crítica e revisão

Solicitar a um segundo modelo que avalie a primeira resposta e proponha correções.

#### Síntese fundamentada

Gerar uma resposta final que preserve referências, incertezas e divergências.

Um prompt de composição poderia ser:

```text
Você é o compositor final.

Use os resultados abaixo para produzir uma resposta única.
Não invente informações ausentes.
Quando houver conflito, apresente a divergência.
Diferencie claramente:
1. evidência documental;
2. inferência;
3. recomendação;
4. limitação.

Resultado do modelo geral:
{{general_output}}

Resultado da recuperação documental:
{{retrieval_output}}

Crítica independente:
{{critic_output}}
```

### 5.6. Etapa 6 — Auditoria

A resposta final deve ser submetida a verificações mínimas:

- existem afirmações sem suporte?
- o modelo citou documentos que realmente foram recuperados?
- houve vazamento de dados?
- a resposta respeita o formato solicitado?
- há contradições internas?
- o grau de certeza está adequado?
- a resposta contém conteúdo potencialmente perigoso?
- os parâmetros de execução foram registrados?

***

## 6. Aplicações educacionais

### 6.1. Tutoria personalizada local

Em uma aplicação educacional, Ops 4 CD pode distribuir uma interação entre:

- um modelo conversacional;
- um perfil do estudante armazenado localmente;
- uma base curricular;
- um gerador de atividades;
- um avaliador de respostas;
- um módulo de adaptação de dificuldade.

O fluxo poderia ser:

\[
\text{pergunta do estudante}
\rightarrow
\text{perfil de aprendizagem}
\rightarrow
\text{modelo tutor}
\rightarrow
\text{verificador curricular}
\rightarrow
\text{resposta adaptada}
\]

O benefício principal é evitar que informações pessoais e registros de aprendizagem sejam enviados para serviços externos. Contudo, a execução local não elimina riscos: arquivos de log, interfaces web mal configuradas e extensões podem expor dados.

### 6.2. Geração de materiais didáticos

Uma solicitação como “produza uma sequência didática sobre mudanças climáticas para o 8º ano” pode ser decomposta em:

1. identificação dos objetivos de aprendizagem;
2. consulta ao currículo local;
3. escolha de conceitos;
4. adaptação da linguagem;
5. geração de atividades;
6. elaboração de critérios de avaliação;
7. verificação de coerência científica.

Cada etapa pode ser executada por um destino diferente ou por um mesmo modelo com contextos isolados.

### 6.3. Correção formativa

A correção automática não deveria limitar-se a atribuir nota. Um sistema composto pode:

- detectar aspectos conceituais;
- identificar dificuldades linguísticas;
- comparar a resposta com critérios previamente definidos;
- gerar feedback;
- sugerir uma nova tentativa;
- encaminhar casos duvidosos ao professor.

A decisão final deve permanecer sob supervisão humana, especialmente quando a avaliação tiver consequências acadêmicas.

### 6.4. Desenvolvimento de sistemas educacionais

Para prototipagem em Google Apps Script, JavaScript ou aplicações web locais, um destino especializado em código pode gerar funções, enquanto outro:

- analisa requisitos;
- verifica segurança;
- propõe testes;
- explica o código;
- adapta a interface para estudantes.

Esse arranjo é mais confiável do que solicitar a um único modelo que escreva e aprove o próprio código.

***

## 7. Requisitos técnicos

### 7.1. Seleção de modelos

A escolha dos modelos deve considerar:

- capacidade de raciocínio;
- aderência a português brasileiro;
- qualidade de saída estruturada;
- velocidade;
- tamanho;
- suporte à arquitetura pelo runtime;
- estabilidade do modelo;
- licença;
- qualidade da quantização;
- comportamento em contexto longo.

Uma política prática poderia ser:

| Perfil de máquina | Configuração inicial |
|---|---|
| Até 8 GB de RAM | Modelo pequeno, Q4 ou inferior, contexto moderado |
| 16 GB de RAM | Modelo de 7B–9B em Q4/Q5 |
| 24–32 GB de RAM | Modelo de 7B–14B em Q5/Q6 |
| GPU dedicada | Descarregamento de camadas e contexto ajustado |
| CPU בלבד | Modelos menores e menor número de tokens |
| Ambiente institucional | Servidor local com API, logs e controle de acesso |

Essas faixas são apenas referências. O desempenho real depende da arquitetura, do sistema operacional, do backend e do comprimento do contexto.

### 7.2. Servidor local

Um servidor local pode ser exposto somente à máquina:

```bash
llama-server \
  -m models/model.Q4_K_M.gguf \
  -c 8192 \
  -ngl 99 \
  --host 127.0.0.1 \
  --port 8080
```

A opção `127.0.0.1` restringe o acesso ao próprio computador. Para uso em rede institucional, é necessário configurar autenticação, firewall, TLS ou uma rede isolada. Expor diretamente um servidor de inferência à internet é uma prática inadequada sem controles adicionais.

### 7.3. API composta

Um orquestrador pode chamar diferentes destinos:

```javascript
async function callModel(endpoint, payload) {
  const response = await fetch(endpoint, {
    method: "POST",
    headers: {"Content-Type": "application/json"},
    body: JSON.stringify(payload)
  });

  if (!response.ok) {
    throw new Error(`Falha no destino: ${endpoint}`);
  }

  return response.json();
}

async function compositeOperation(request) {
  const [general, critic] = await Promise.all([
    callModel("http://127.0.0.1:8080/v1/chat/completions", {
      model: "general",
      messages: [{role: "user", content: request}]
    }),
    callModel("http://127.0.0.1:8081/v1/chat/completions", {
      model: "critic",
      messages: [{role: "user", content: request}]
    })
  ]);

  return {
    general,
    critic,
    status: "completed"
  };
}
```

Em uma implementação real, o código deveria incluir:

- limites de tempo;
- repetição controlada;
- validação do JSON;
- circuit breaker;
- fila de tarefas;
- cancelamento;
- controle de concorrência;
- logs estruturados;
- autenticação quando aplicável.

### 7.4. Contexto e cache

O contexto deve ser tratado como recurso limitado. Ao compor várias saídas, o sistema pode ultrapassar a janela disponível. Recomenda-se:

- resumir resultados intermediários;
- eliminar repetições;
- limitar o tamanho de cada saída;
- armazenar documentos fora do prompt;
- recuperar apenas trechos relevantes;
- utilizar identificadores de evidência;
- reservar tokens para a resposta final.

Uma política simples é reservar:

\[
C_{\text{total}} =
C_{\text{instrução}}+
C_{\text{evidência}}+
C_{\text{histórico}}+
C_{\text{saída}}
\]

Se \(C_{\text{total}}\) exceder o limite do modelo, o orquestrador deve reduzir ou resumir componentes antes da inferência.

***

## 8. Avaliação da arquitetura

### 8.1. Indicadores de desempenho

A avaliação de Ops 4 CD deve considerar mais do que velocidade. Indicadores relevantes incluem:

#### Qualidade

- precisão factual;
- completude;
- coerência;
- adequação linguística;
- fidelidade aos documentos;
- qualidade pedagógica.

#### Eficiência

- tokens por segundo;
- tempo até o primeiro token;
- tempo total;
- uso de RAM;
- uso de VRAM;
- consumo energético;
- quantidade de chamadas por solicitação.

#### Confiabilidade

- taxa de falha;
- taxa de respostas inválidas;
- recuperação após falhas;
- consistência entre execuções;
- disponibilidade dos destinos.

#### Governança

- rastreabilidade;
- preservação de logs;
- controle de versões;
- explicitação das fontes;
- conformidade com políticas institucionais.

Uma métrica composta pode ser representada por:

\[
Q_{\text{ops}} =
w_qQ+
w_eE+
w_rR+
w_gG-
w_cC
\]

em que:

- \(Q\) = qualidade da resposta;
- \(E\) = eficiência;
- \(R\) = robustez;
- \(G\) = governança;
- \(C\) = custo computacional;
- \(w_i\) = pesos definidos conforme o caso de uso.

### 8.2. Protocolo experimental

Um experimento comparativo pode utilizar três condições:

1. modelo único GGUF;
2. modelo único com recuperação documental;
3. arquitetura Ops 4 CD com modelo gerador, crítico e compositor.

Cada condição deve receber o mesmo conjunto de tarefas. Devem ser comparados:

- qualidade por avaliadores humanos;
- consistência factual;
- tempo total;
- uso de memória;
- número de tokens;
- taxa de falha;
- grau de satisfação do usuário.

No contexto educacional, o conjunto de avaliação deve conter tarefas reais ou representativas, como:

- produção de plano de aula;
- elaboração de questões;
- adaptação para diferentes níveis;
- correção comentada;
- geração de rubrica;
- explicação de conceitos;
- programação de protótipos.

### 8.3. Hipóteses

Podem ser formuladas as seguintes hipóteses:

- **H1:** uma arquitetura composta produz respostas mais completas do que um modelo isolado em tarefas multidimensionais.
- **H2:** o uso de um destino crítico reduz erros factuais e contradições.
- **H3:** a execução local reduz dependência de serviços externos, mas aumenta a responsabilidade de administração do ambiente.
- **H4:** a composição de múltiplos destinos aumenta o tempo e o consumo de memória.
- **H5:** a especialização de modelos produz ganhos apenas quando o roteamento é adequado.
- **H6:** sistemas compostos são mais úteis quando as tarefas têm etapas claramente separáveis.

***

## 9. Riscos e limitações

### 9.1. Complexidade operacional

Um modelo único é mais simples de instalar, atualizar e depurar. Ops 4 CD introduz:

- múltiplos modelos;
- mais processos;
- diferentes formatos de prompt;
- conflitos de versões;
- maior necessidade de observabilidade;
- maior consumo de armazenamento.

O ganho de qualidade precisa justificar essa complexidade.

### 9.2. Erros compostos

Se cada destino apresentar uma probabilidade \(p_i\) de erro e o sistema depender de todos eles, a probabilidade de pelo menos um erro cresce com o número de componentes. Em uma aproximação independente:

\[
P(\text{erro em algum destino}) =
1-\prod_{i=1}^{n}(1-p_i)
\]

Isso não significa que a arquitetura composta seja necessariamente menos confiável. Um verificador pode corrigir erros do gerador. Porém, a composição precisa ser projetada para reduzir, e não apenas acumular, incertezas.

### 9.3. Alucinação do compositor

O compositor pode produzir uma resposta aparentemente coerente, mas que não corresponde exatamente a nenhuma das respostas anteriores. Por isso, ele deve receber instruções explícitas para:

- não inventar evidências;
- manter as incertezas;
- indicar conflitos;
- distinguir inferência de fonte;
- preservar identificadores documentais.

### 9.4. Privacidade ilusória

“Local” não significa automaticamente “seguro”. Podem ocorrer exposições por:

- logs em texto aberto;
- diretórios compartilhados;
- APIs sem autenticação;
- interfaces acessíveis pela rede;
- extensões de navegador;
- telemetria de aplicativos;
- cópias automáticas em serviços de nuvem;
- scripts executados com privilégios elevados.

O ambiente deve adotar minimização de dados, controle de acesso, criptografia de armazenamento quando necessária e política clara de retenção.

### 9.5. Licenças

Modelos GGUF podem ser derivados de modelos com licenças distintas. Antes de uso institucional ou comercial, é necessário verificar:

- licença do modelo original;
- licença da quantização;
- obrigações de atribuição;
- restrições comerciais;
- uso de dados pessoais;
- compatibilidade com redistribuição.

***

## 10. Governança e ética

### 10.1. Supervisão humana

A arquitetura deve informar quando:

- não encontrou evidência suficiente;
- dois modelos divergiram;
- a confiança é baixa;
- o documento recuperado é incompleto;
- a tarefa exige decisão profissional;
- a resposta pode afetar avaliação ou tratamento de pessoas.

Em sistemas educacionais, o professor deve permanecer responsável por decisões avaliativas, disciplinares ou relacionadas ao acompanhamento de estudantes.

### 10.2. Transparência

O usuário deve poder saber:

- que a resposta foi produzida localmente;
- quais modelos foram utilizados;
- se documentos locais foram consultados;
- se houve crítica automática;
- quais limitações foram identificadas;
- quando a resposta é apenas uma sugestão.

Uma interface pode apresentar:

```text
Modelos utilizados:
- Tutor: modelo GGUF Q5_K_M
- Verificador: modelo GGUF Q4_K_M
- Documentos: currículo_local_2026.pdf

Status:
- resposta composta
- evidência documental encontrada
- revisão humana recomendada
```

### 10.3. Proteção de dados educacionais

Dados de estudantes devem ser minimizados. O sistema deve evitar registrar, sem necessidade:

- nome completo;
- matrícula;
- informações de saúde;
- histórico disciplinar;
- dados familiares;
- identificadores biométricos;
- texto integral de avaliações sensíveis.

Quando possível, deve-se utilizar pseudônimos, IDs internos e campos removidos antes da inferência.

***

## 11. Proposta de implementação progressiva

### Fase 1 — Modelo único

Objetivo: validar a execução local.

- instalar runtime;
- executar um modelo GGUF;
- medir velocidade;
- testar português;
- definir prompts;
- criar conjunto de tarefas.

### Fase 2 — API local

Objetivo: separar interface e inferência.

- executar llama-server ou Ollama;
- criar cliente JavaScript;
- testar chamadas HTTP;
- registrar parâmetros;
- implementar timeout.

### Fase 3 — Segundo destino

Objetivo: introduzir especialização.

- adicionar modelo de código, crítica ou classificação;
- comparar qualidade;
- controlar memória;
- implementar roteamento.

### Fase 4 — Compositor

Objetivo: integrar resultados.

- definir formato estruturado;
- preservar evidências;
- tratar conflitos;
- adicionar validação de saída;
- criar métricas de avaliação.

### Fase 5 — Aplicação educacional

Objetivo: inserir o sistema em um fluxo pedagógico real.

- selecionar público;
- estabelecer objetivos de aprendizagem;
- testar com professores;
- aplicar avaliação de usabilidade;
- revisar riscos;
- documentar limites.

### Fase 6 — Operação institucional

Objetivo: garantir sustentabilidade.

- controle de versões;
- atualizações dos modelos;
- backup;
- gestão de usuários;
- política de logs;
- monitoramento;
- plano de contingência;
- formação dos usuários.

***

## 12. Conclusão

Ops 4 CD pode ser compreendido como uma arquitetura de **orquestração de destinos compostos para inferência local**, na qual diferentes modelos GGUF, bases documentais, ferramentas e módulos de avaliação colaboram para atender uma solicitação. A metáfora dos destinos compostos é tecnicamente produtiva porque desloca o foco do modelo isolado para o conjunto de operações que transforma uma solicitação em uma resposta confiável.

No contexto dos modelos locais, a arquitetura apresenta quatro contribuições principais:

1. **modularidade**, ao permitir combinar modelos com funções distintas;
2. **soberania**, ao manter dados e inferência sob controle local;
3. **especialização**, ao encaminhar cada subtarefa ao componente mais apropriado;
4. **auditabilidade**, ao registrar modelos, parâmetros, documentos e etapas.

Sua principal limitação é o aumento da complexidade. Um sistema composto não é automaticamente melhor do que um modelo único. Ele se torna vantajoso quando a tarefa exige decomposição, consulta documental, verificação, múltiplas perspectivas ou integração com ferramentas externas.

Para uso educacional, a proposta é especialmente promissora em tutoria personalizada, produção de materiais, correção formativa, desenvolvimento de protótipos e apoio à pesquisa. Entretanto, a arquitetura deve ser subordinada a objetivos pedagógicos claros, supervisão docente, proteção de dados e avaliação empírica.

**No ecossistema GGUF, Operations for Composite Destination constitui uma abordagem operacional para transformar modelos locais isolados em sistemas compostos, especializados, verificáveis e pedagogicamente orientados, desde que o roteamento, a composição e a governança sejam tratados como partes centrais do projeto — e não como detalhes posteriores.**

### Referências

- ActiveMQ. *Composite Destinations*. Documentação sobre um destino lógico que representa múltiplos destinos físicos e permite encaminhamento composto. [activemq.apache](https://activemq.apache.org/components/classic/documentation/composite-destinations)
- W3C. *Compositing and Blending 1.0*. Fundamentação sobre fonte, destino, *backdrop*, composição, grupos e operadores de mistura. [w3](https://www.w3.org/TR/2012/WD-compositing-20120816/)
- DataCamp. *GGUF Format: A Complete Guide to Local LLM Inference*. Informações sobre estrutura, quantização, runtimes, memória e execução local de modelos GGUF. [datacamp](https://www.datacamp.com/tutorial/gguf-format-a-complete-guide)
