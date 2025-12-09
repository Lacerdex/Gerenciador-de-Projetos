# 🤖 Project Manager AI Chatbot

Este projeto é um sistema de gerenciamento de tarefas e projetos potencializado por um **Chatbot de Inteligência Artificial**. Ele é capaz de ler e analisar dados de planilhas (como Excel/CSV), cruzar informações com documentos de referência (PDFs) e manter o contexto da conversa, tudo em tempo real.

# 🎯 Objetivo Principal

Centralizar a consulta e a análise de grandes volumes de dados de projetos (como status de tarefas, porcentagens de conclusão e documentos de referência) através de uma interface de chatbot amigável e inteligente.

# 🚀 Tecnologias Utilizadas
| Categoria |	Tecnologia |	Função no Projeto |
| :--- | :--- | :--- |
| **Backend** |	**Flask** |	Micro-framework Python que hospeda a API REST. |
| **Análise de Dados** |	**Pandas** |	Leitura, tratamento, concatenação (`pd.concat`) e filtragem dos dados de planilhas (`.xlsx, .csv`). |
| **IA/LLM** |	**Chatbots (IA Primária e Secundária)** |	Geração de respostas, análise de dados e **resumo de histórico de conversas.** |
| **Banco de Dados** |	**MySQL** |	Armazenamento de dados persistentes (Usuários e Tarefas). |
| **Recuperação de Info** |	**Lógica de Busca em PDF** |	Função `cruzarDadosPDF` para encontrar trechos de referência em documentos. |
| **Segurança** | **Flask-Bcrypt** |	Hashing de senhas de usuários. |
| **Autenticação** |	**Flask-CSRF** |	Proteção contra ataques CSRF na API. |

# 💾 Arquitetura do Processamento de Mensagens

O endpoint principal (`/enviar`) é o núcleo do sistema, realizando uma complexa sequência de operações a cada mensagem do usuário.
Fluxo de Requisição (Chatbot)

1. **Recepção de Dados:**
   * Recebe a pergunta do usuário e uma lista de arquivos (`lista_arquivos`) a serem analisados.

2. **Tratamento de Dados** (`Pandas`):
   * Lê e padroniza as colunas de até **2 planilhas.**
   * **Concatena** os DataFrames e os classifica.
   * **Filtra** e prepara os dados relevantes (100% concluído, `< 100%`, dados por `condição`, etc.) em formato de dicionário para enviar à IA.

3. **Gestão de Contexto:**
   * Lê o histórico das conversas dos **últimos 5 minutos** (`ler_conversa_ultimos_5_minutos`).
   * Envia esse histórico para uma **IA Secundária** (`resumir_conversa_pela_segIA`) para gerar um resumo conciso.
   * O resumo é **salvo** (`salvar_resumo`) e usado como contexto de longo prazo para a IA principal.

4. **Geração da Resposta (IA Primária):**
   * A IA principal (`perguntar_para_ia`) recebe a **pergunta**, os **dados tabulares filtrados** e o **resumo do contexto.**

5. **Recuperação de Documentos (RAG):**
   * Verifica se a resposta da IA contém o marcador `#DOCUMENTO_SOLICITADO:`.
   * Se o marcador estiver presente, o termo é extraído e usado para buscar trechos e o caminho completo em arquivos **PDFs** (`cruzarDadosPDF`).

6. **Resposta Final:** Retorna a mensagem da IA, junto com o trecho do PDF encontrado (se houver), em formato JSON.

# 🛠️ Estrutura do Prompt de Sistema para a IA Principal

Seu prompt deve ser modular para acomodar as quatro peças de contexto que você está enviando: a pergunta do usuário, os dados tabulares, o resumo da conversa e o requisito de busca em PDF.

**1. Definição de Papel e Objetivo**

Instruções claras sobre quem a IA é e qual é a sua principal tarefa.

  * **Persona:** Gerente de Projetos especialista em análise de dados e documentação.
  * **Objetivo:** Analisar os dados fornecidos e o contexto da conversa para dar respostas precisas, orientadas a dados e, se necessário, disparar a busca por documentos.

**2. Regras de Injeção de Dados (JSON)**

Instrua a IA sobre a fonte de verdade para dados factuais.

  * **Dados:** Você receberá um objeto JSON/Dicionário com os **dados de projetos filtrados** (`dados_atuais).
  * **Prioridade:** A IA deve priorizar esses dados para responder perguntas sobre quantidades, status, porcentagens e condições de tarefas.
  * **Limitação:** A IA não deve inventar dados que não estejam presentes no JSON.

**3. Regras de Contexto e Memória**

Instrua a IA sobre como usar o resumo de contexto recente.

  * **Resumo:** Você receberá um `RESUMO_DA_CONVERSA_RECOMENDADO.
  * **Uso:** Use-o para manter a coerência temática ou para responder perguntas que se referem a tópicos discutidos nos últimos minutos.

**4. Regra de Trigger para Documentação (RAG) - CRÍTICA**

Essa é a instrução mais importante para o seu backend. É o que permite que a IA acione a função `cruzarDadosPDF`.

  * **Condição:** Se a pergunta do usuário for uma solicitação de **como fazer** algo, um **procedimento** ou uma **referência documental** específica (`docRef`), a IA deve responder normalmente, mas **ADICIONAR** o marcador no final.
  * **Formato do Marcador:** A IA **DEVE** adicionar `#DOCUMENTO_SOLICITADO: [O Título Exato do Documento/Item]` no final da resposta.

# 🧩 Template de Prompt (Sugestão Completa)

Você pode usar esta estrutura como o seu **System Prompt** primário, injetando os valores dinâmicos (`{DADOS_ATUAIS}`, `{RESUMO_DA_CONVERSA}`, `{PERGUNTA_USUARIO}`).

*Este é o texto que sua função* `perguntar_para_ia` *injetará antes da pergunta do usuário.*

    SYSTEM_PROMPT = f"""
    Você é um **Assistente de Gerenciamento de Projetos** especialista em análise de dados e recuperação de informações.
    Sua função é analisar a pergunta do usuário e o contexto fornecido para gerar uma resposta clara, baseada em fatos e documentos.
    
    ### 1. DADOS DE PROJETO (Fatos Atuais)
    Você recebeu os seguintes dados JSON filtrados das planilhas:
    --- JSON INÍCIO ---
    {dados_atuais}
    --- JSON FIM ---
    Use estes dados para responder a todas as perguntas sobre status de tarefas, porcentagens de conclusão, condições ('A', 'B', 'C', 'Sempre') e contagens.
    
    ### 2. CONTEXTO RECENTE (Memória de Curto Prazo)
    Considere este resumo das nossas últimas interações para manter o contexto:
    --- RESUMO INÍCIO ---
    {resumo}
    --- RESUMO FIM ---
    
    ### 3. REGRAS DE SAÍDA (Obrigações)
    1.  **Prioridade:** Suas respostas factuais devem ser tiradas dos 'DADOS DE PROJETO'.
    2.  **Recuperação de Documentos (CRÍTICO):** Se o usuário perguntar *como fazer algo*, *qual o procedimento para X* ou *pedir um documento de referência*, você deve fornecer a resposta mais útil possível E **OBRIGATORIAMENTE** adicionar a seguinte *tag* na última linha da sua resposta.
        * **Formato da Tag:** ` #DOCUMENTO_SOLICITADO: [O TÍTULO EXATO DO ITEM/DOCUMENTO]`
        * **Exemplo:** Se a pergunta for "Como fazer a embalagem primária?", sua resposta deve terminar com ` #DOCUMENTO_SOLICITADO: Embalagem Primaria`
    
    ### PERGUNTA DO USUÁRIO:
    {pergunta_usuario}
    """

Ao seguir estas regras de formatação no seu Prompt de Sistema, você garante que sua lógica Python de `if MARCADOR in resposta_da_ia:` sempre funcionará corretamente, transformando a IA em um motor de busca de documentos.
