---
title: "Publicando seu Projeto Flutter na Web com Firebase Hosting"
date: 2025-08-12T10:37:27-03:00
draft: false
tags: ["Flutter", "Firebase", "Web", "Hosting", "Deploy"]
categories: ["Flutter"]
---

Levar um projeto Flutter para a web é uma excelente maneira de alcançar um público maior sem a necessidade de uma loja de aplicativos. Quando combinado com o Firebase Hosting, o processo se torna não apenas simples, mas também incrivelmente rápido e seguro, com um generoso plano gratuito.

Neste guia, vamos percorrer o passo a passo para publicar um projeto Flutter já existente na web usando o Firebase Hosting.

### Pré-requisitos

Antes de começar, garanta que você tenha:

1.  **Um projeto Flutter existente.**
2.  **O SDK do Flutter** instalado e configurado no seu PATH.
3.  **Uma conta Google** para acessar o Firebase.
4.  **Node.js e npm** instalados. Eles são necessários para instalar a ferramenta de linha de comando (CLI) do Firebase no próximo passo. O `npm` já vem incluído na instalação do Node.js. Se você não o tiver, baixe a versão **LTS** no site oficial [nodejs.org](https://nodejs.org/).

    **Importante:** Na página de download, escolha a opção **"Windows Installer (.msi)"**. Evite outras opções como Docker, que são para casos de uso mais avançados. O instalador cuidará de todo o processo de configuração automaticamente.

### Passo 1: Configurar o Ambiente Firebase (Apenas uma vez por máquina)

Este passo prepara seu computador para se comunicar com o Firebase. Você só precisa fazer isso **uma única vez**; não é necessário repetir para cada novo projeto.

1.  **Instale a CLI globalmente** via npm. Isso adiciona os comandos do Firebase ao seu sistema:
    ```bash
    npm install -g firebase-tools
    ```

2.  **Faça login** na sua conta Google. Isso autentica sua máquina:
    ```bash
    firebase login
    ```
    Este comando abrirá uma janela no seu navegador para autenticação. Uma vez logado, você permanecerá conectado para futuros projetos.

### Passo 2: Configurar o Projeto no Console do Firebase

Com as ferramentas prontas, o próximo passo é criar um projeto no Firebase que receberá nosso aplicativo.

1.  Acesse o [Console do Firebase](https://console.firebase.google.com/).
2.  Clique em **"Adicionar projeto"** e dê um nome a ele (ex: `meu-app-flutter-web`).
3.  Siga os passos de configuração. Não é necessário ativar o Google Analytics para este tutorial.

### Passo 3: Gerar a Build para a Web

Agora que o Firebase está pronto para receber os arquivos, vamos gerar a versão de produção do nosso app Flutter.

1.  Execute o comando abaixo no terminal, na raiz do seu projeto:
    ```bash
    flutter build web
    ```
    Este comando cria uma pasta `build/web` com todo o conteúdo estático (HTML, CSS, JS) que será publicado.

*Dica: Você pode testar localmente antes de publicar com `flutter run -d chrome`.*

### Passo 4: Inicializar o Firebase no Projeto Flutter

Com a build pronta e o projeto criado no console, vamos conectar o seu projeto local ao Firebase.

1.  Na pasta raiz do seu projeto Flutter, execute o comando de inicialização:
    ```bash
    firebase init
    ```

2.  Siga as instruções que aparecerão no terminal:
    * "Which Firebase features do you want to set up?"
        * **Atenção:** Selecione a opção **Hosting: Configure files for Firebase Hosting and (optionally) set up GitHub Action deploys**. Esta é a opção correta para o plano gratuito. Não escolha "App Hosting", pois ele exige um plano de faturamento.
    * "Please select an option:"
        * Escolha **Use an existing project** e selecione o projeto que você criou no passo anterior.
    * "What do you want to use as your public directory?"
        * **Esta é a parte mais importante!** Digite `build/web`.
    * "Configure as a single-page app (rewrite all urls to /index.html)?"
        * Digite **`y`** (Sim). Isso é crucial para a navegação do Flutter.
    * "Set up automatic builds and deploys with GitHub?"
        * Digite **`N`** (Não). A configuração com GitHub Actions, embora muito poderosa para automatizar o deploy, é um tópico mais avançado que não abordaremos neste tutorial.
    * "File build/web/index.html already exists. Overwrite? (y/N)"
        * **Digite `N` (Não)**. É fundamental manter o arquivo `index.html` original gerado pelo Flutter.

Ao final, dois novos arquivos terão sido criados no seu projeto. Eles são a base da configuração do Firebase para esta pasta:
* **`.firebaserc`**: Funciona como uma "etiqueta" que conecta esta pasta local ao seu projeto específico no console do Firebase.
* **`firebase.json`**: É o arquivo de configuração principal do Hosting. Ele define qual pasta será publicada (`build/web`) e outras regras, como o redirecionamento necessário para que aplicativos de página única (SPA), como os feitos em Flutter, funcionem corretamente.

### Passo 5: Fazer o Deploy!

Tudo está pronto. Com apenas um comando, seu site estará no ar.

```bash
firebase deploy
```

Após a conclusão, o terminal mostrará a **Hosting URL**. Esse é o link para o seu projeto Flutter, agora ao vivo na web!

Parabéns! Você publicou seu aplicativo Flutter na web de forma rápida e eficiente com o Firebase Hosting.

---
### Atualizando seu Aplicativo

Uma vez que a configuração inicial está feita, o processo para atualizar seu site é muito mais simples. Toda vez que você fizer uma alteração no seu código Flutter, basta repetir dois passos:

1.  **Gerar a nova build:**
    ```bash
    flutter build web
    ```
2.  **Fazer o deploy da nova versão:**
    ```bash
    firebase deploy
    ```

E pronto! O Firebase cuidará de atualizar os arquivos no servidor.