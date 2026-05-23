# Dossier de Implantação (Deploy) no Render 🚀

Este guia detalha como configurar e realizar o deploy do projeto **Kanban** na plataforma [Render](https://render.com/). 

O projeto foi atualizado para suportar variáveis de ambiente dinâmicas, garantindo segurança na autenticação e persistência de dados.

---

## 🛠️ Alterações de Padronização Realizadas no Projeto

Para tornar a aplicação compatível com o ambiente de nuvem do Render, os seguintes pontos foram parametrizados no arquivo `server.js`:

1. **Porta Dinâmica (`PORT`)**:
   - Alterado de um valor fixo (`3000`) para `process.env.PORT || 3000`. O Render define automaticamente a porta que a aplicação deve escutar na variável `PORT`.
2. **Chave Secreta JWT (`JWT_SECRET`)**:
   - O segredo de assinatura do JWT foi externalizado para `process.env.JWT_SECRET`, com fallback seguro. Isso impede que chaves privadas fiquem expostas no código fonte público.
3. **Diretório de Dados Persistente (`DATA_DIR`)**:
   - Como os servidores do Render usam um sistema de arquivos efêmero (arquivos salvos localmente são perdidos a cada reinicialização ou novo deploy), adicionamos suporte a um diretório de dados dinâmico (`process.env.DATA_DIR || __dirname`).
   - Os arquivos `users.json`, `tasks.json` e `revoked_tokens.json` serão salvos nesse diretório montado em um disco permanente.

---

## 📋 Passo a Passo para Deploy no Render

Siga estas instruções passo a passo para colocar sua aplicação no ar com persistência de dados:

### 1. Criar o Web Service no Render
1. Acesse o seu painel do [Render](https://dashboard.render.com/) e faça login.
2. Clique no botão **New +** (canto superior direito) e selecione **Web Service**.
3. Conecte sua conta do GitHub (caso ainda não tenha conectado).
4. Localize e selecione o repositório `allanlerma/kanban2305`.

### 2. Configurar os Parâmetros Básicos
Na tela de criação do Web Service, preencha os campos abaixo:
* **Name**: `kanban-2305` (ou o nome de sua preferência)
* **Region**: Escolha a mais próxima (ex: *Ohio (us-east-2)* or *Frankfurt (eu-central-1)*)
* **Branch**: `main`
* **Runtime**: `Node`
* **Build Command**: `npm install`
* **Start Command**: `node server.js` (ou `npm start`)
* **Instance Type**: `Free` (ou o plano que desejar)

### 3. Configurar as Variáveis de Ambiente (Environment Variables)
1. Role a tela até a seção **Advanced** ou clique na aba **Env Groups / Environment Variables**.
2. Adicione as seguintes variáveis:
   - **`JWT_SECRET`**: Digite uma chave forte e aleatória (ex: um hash gerado ou uma string longa aleatória).
   - **`DATA_DIR`**: `/var/data` (caminho onde o disco persistente será montado).
   - *Nota: A variável `PORT` é gerenciada automaticamente pelo Render, você não precisa adicioná-la.*

### 4. Configurar o Disco Persistente (Persistent Disk) ⚠️ *Muito Importante*
Como a aplicação utiliza arquivos JSON locais para salvar os usuários e tarefas, é necessário configurar um disco persistente para evitar perda de dados.
1. Ainda na seção **Advanced**, role até a área **Disks**.
2. Clique em **Add Disk**.
3. Preencha as configurações do disco:
   - **Name**: `kanban-data`
   - **Mount Path**: `/var/data` (deve ser exatamente o mesmo valor definido na variável `DATA_DIR`)
   - **Size**: `1 GiB` (suficiente para arquivos JSON e plano gratuito)
4. Clique em **Create Web Service**.

---

## ⚡ Verificação pós-Deploy
Assim que o Render terminar o build e colocar o serviço online:
1. Acesse a URL gerada pelo Render (ex: `https://kanban-2305.onrender.com`).
2. Tente cadastrar um novo usuário e fazer login.
3. Crie algumas tarefas no Kanban.
4. Para testar a persistência: faça um deploy manual (clicando em *Clear Build Cache & Deploy*) ou reinicie o serviço. Acesse a URL novamente; suas tarefas e usuários criados devem continuar salvos!
