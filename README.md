# Agente de Triagem para Políticas Internas

Este notebook implementa um agente de Service Desk para políticas internas da empresa Carraro Desenvolvimento, utilizando as bibliotecas LangChain e LangGraph. O agente é capaz de triar as mensagens dos usuários, responder automaticamente a perguntas frequentes sobre as políticas internas e, quando necessário, solicitar mais informações ou indicar a abertura de um chamado para o departamento responsável (RH/IT).

O projeto é dividido em três partes principais:

## Parte 1: Triagem da Mensagem

Nesta parte, a mensagem inicial do usuário é analisada para determinar a intenção e a urgência.

-   **Instalação de Bibliotecas**: São instaladas as bibliotecas `langchain`, `langchain-google-genai` e `google-generativeai` para interação com modelos de linguagem e ferramentas do Google.
-   **Importação da API Key**: A chave de API do Google Gemini é importada de forma segura utilizando `google.colab.userdata`.
-   **Conexão com o Gemini**: É estabelecida a conexão com o modelo `gemini-2.5-flash` utilizando a biblioteca `langchain-google-genai`.
-   **Definição do Prompt de Triagem**: Um prompt específico é definido para guiar o modelo Gemini na triagem das mensagens, classificando-as em `AUTO_RESOLVER`, `PEDIR_INFO` ou `ABRIR_CHAMADO`, além de determinar a urgência (`BAIXA`, `MEDIA`, `ALTA`) e identificar campos faltantes.
-   **Estrutura de Saída**: É definida uma estrutura Pydantic (`TriagemOut`) para garantir que a saída do modelo de triagem esteja em um formato JSON consistente.
-   **Criação da Cadeia de Triagem**: É criada uma cadeia de triagem utilizando o modelo Gemini e a estrutura de saída definida, facilitando a extração estruturada da decisão de triagem.
-   **Função de Triagem**: Uma função `triagem` é criada para invocar a cadeia de triagem com a mensagem do usuário e retornar o resultado em formato de dicionário.
-   **Testes de Triagem**: São realizados testes com diferentes mensagens para verificar o comportamento da função de triagem.

## Parte 2: Recuperação de Informação com RAG

Esta parte foca na recuperação de informações relevantes a partir de documentos PDF contendo as políticas internas, utilizando a técnica de Retrieval Augmented Generation (RAG).

-   **Instalação de Bibliotecas**: São instaladas bibliotecas adicionais como `langchain_community`, `faiss-cpu`, `langchain-text-splitters` e `pymupdf` para carregamento de documentos, divisão de texto e criação de vetores de embeddings.
-   **Carregamento de Documentos**: Documentos PDF localizados no diretório `/content/` são carregados utilizando `PyMuPDFLoader`.
-   **Divisão de Texto (Chunking)**: O texto dos documentos carregados é dividido em pedaços menores (chunks) utilizando `RecursiveCharacterTextSplitter` para otimizar a busca por semelhança.
-   **Criação de Embeddings**: São gerados vetores numéricos (embeddings) para cada chunk de texto utilizando o modelo `gemini-embedding-001`.
-   **Criação do Vector Store e Retriever**: Os embeddings são armazenados em um Vector Store utilizando FAISS para permitir buscas eficientes. Um Retriever é configurado para buscar chunks relevantes com base na similaridade semântica com a pergunta do usuário, com um limite de score de similaridade.
-   **Criação da Cadeia RAG**: É criada uma cadeia de documentos (`create_stuff_documents_chain`) que combina o modelo Gemini com um prompt específico para RAG, permitindo que o modelo responda à pergunta do usuário com base no contexto fornecido pelos chunks recuperados.
-   **Funções Auxiliares para Citações**: São definidas funções para limpar o texto (`_clean_text`), extrair trechos relevantes (`extrair_trecho`) e formatar as citações a partir dos documentos recuperados (`formatar_citacoes`).
-   **Função Principal de Perguntas RAG**: Uma função `perguntar_politica_RAG` é criada para executar o processo completo de RAG: buscar documentos relevantes, invocar a cadeia RAG para obter a resposta e formatar as citações.
-   **Testes de RAG**: São realizados testes com perguntas para verificar a capacidade do agente em encontrar e utilizar informações dos documentos para responder.

## Parte 3: Orquestração com LangGraph

Nesta parte, os diferentes componentes (triagem e RAG) são orquestrados em um fluxo de trabalho utilizando LangGraph, definindo os estados e as transições entre eles.

-   **Instalação da Biblioteca**: A biblioteca `langgraph` é instalada para construir e gerenciar o grafo de estados.
-   **Definição do Estado do Agente**: Um `TypedDict` (`AgentState`) é definido para representar o estado do agente durante o processamento de uma mensagem, incluindo a pergunta, o resultado da triagem, a resposta, as citações, o sucesso do RAG e a ação final.
-   **Definição dos Nós do Grafo**: São definidos nós para cada etapa do fluxo: `node_triagem` (executa a triagem), `node_auto_resolver` (tenta responder usando RAG), `node_pedir_info` (indica a necessidade de mais informações) e `node_abrir_chamado` (simula a abertura de um chamado).
-   **Definição das Arestas Condicionais**: São definidas funções para determinar a transição entre os nós com base no estado atual: `decidir_pos_triagem` (decide para qual nó ir após a triagem) e `decidir_pos_auto_resolver` (decide para onde ir após a tentativa de auto-resolução).
-   **Criação do Workflow com LangGraph**: Um grafo de estados (`StateGraph`) é construído, adicionando os nós e definindo as arestas, incluindo as condicionais, para criar o fluxo de processamento da mensagem.
-   **Compilação do Grafo**: O grafo é compilado para ser executado.
-   **Visualização do Grafo**: O grafo de estados é visualizado para entender o fluxo do agente.
-   **Testes do Grafo Completo**: São realizados testes com diferentes mensagens para verificar o comportamento do agente no fluxo completo, incluindo a triagem, a tentativa de RAG e as ações finais (responder, pedir info, abrir chamado).

Este notebook fornece uma estrutura completa para a criação de um agente de Service Desk inteligente, demonstrando a integração de modelos de linguagem, recuperação de informação e orquestração de fluxo de trabalho.

#Observações

-Necessário carregar os arquivos pdfs da pasta data no colab
-Necessário gerar uma chave de API do Google Gemini em: [Google Ai Studio](https://aistudio.google.com/app/u/1/apikey?utm_source=website&utm_medium=referral&utm_campaign=Alura-may-25&pli=1)
