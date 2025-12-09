# 🤖 Project Manager AI Chatbot

Este projeto é um sistema de gerenciamento de tarefas e projetos potencializado por um Chatbot de Inteligência Artificial. Ele é capaz de ler e analisar dados de planilhas (como Excel/CSV), cruzar informações com documentos de referência (PDFs) e manter o contexto da conversa, tudo em tempo real.

# 🎯 Objetivo Principal

Centralizar a consulta e a análise de grandes volumes de dados de projetos (como status de tarefas, porcentagens de conclusão e documentos de referência) através de uma interface de chatbot amigável e inteligente.

# 🚀 Tecnologias Utilizadas
| Categoria |	Tecnologia |	Função no Projeto |
| :--- | :--- | :--- |
| Backend |	Flask |	Micro-framework Python que hospeda a API REST. |
| Análise de Dados |	Pandas |	Leitura, tratamento, concatenação (pd.concat) e filtragem dos dados de planilhas (.xlsx, .csv). |
| IA/LLM |	Chatbots (IA Primária e Secundária) |	Geração de respostas, análise de dados e resumo de histórico de conversas. |
| Banco de Dados |	MySQL |	Armazenamento de dados persistentes (Usuários e Tarefas). |
| Recuperação de Info |	Lógica de Busca em PDF |	Função `cruzarDadosPDF` para encontrar trechos de referência em documentos. |
| Segurança | Flask-Bcrypt |	Hashing de senhas de usuários. |
| Autenticação |	Flask-CSRF |	Proteção contra ataques CSRF na API. |

# 💾 Arquitetura do Processamento de Mensagens

O endpoint principal (/enviar) é o núcleo do sistema, realizando uma complexa sequência de operações a cada mensagem do usuário.
Fluxo de Requisição (Chatbot)

1. **Recepção de Dados:** Recebe a pergunta do usuário e uma lista de arquivos (lista_arquivos) a serem analisados.

2. **Tratamento de Dados** (`Pandas`):
   * Lê e padroniza as colunas de até **2 planilhas.**
   * **Concatena** os DataFrames e os classifica.
   * **Filtra** e prepara os dados relevantes (100% concluído, `< 100%`, dados por `condição`, etc.) em formato de dicionário para enviar à IA.

3. **Gestão de Contexto:**
   * Lê o histórico das conversas dos **últimos 5 minutos** (`ler_conversa_ultimos_5_minutos`).
   * Envia esse histórico para uma **IA Secundária** (`resumir_conversa_pela_segIA`) para gerar um resumo conciso.
   * O resumo é **salvo** (`salvar_resumo`) e usado como contexto de longo prazo para a IA principal.

4. **Geração da Resposta (IA Primária):** A IA principal (`perguntar_para_ia`) recebe a **pergunta**, os **dados tabulares filtrados** e o **resumo do contexto.**

5. **Recuperação de Documentos (RAG):**
   * Verifica se a resposta da IA contém o marcador `#DOCUMENTO_SOLICITADO:`.
   * Se o marcador estiver presente, o termo é extraído e usado para buscar trechos e o caminho completo em arquivos **PDFs** (`cruzarDadosPDF`).

6. **Resposta Final:** Retorna a mensagem da IA, junto com o trecho do PDF encontrado (se houver), em formato JSON.
