# Cromática: CMS de novelas e filmes (Sanity + React)

O site foi migrado para **React + TypeScript + Vite**, seguindo a arquitetura moderna de componentes e páginas (`src/components/layout`, `src/components/ui`, `src/pages/home/components`, etc.).

- **Sanity** guarda só os dados (título, gênero, episódios, sinopse, capa, link, ordem, publicado). O painel (Sanity Studio) é onde o cliente trabalha.
- **YouTube ou Vimeo** hospedam os vídeos. O site aceita os dois tipos de link.
- **Site (React + Vite)** busca a lista publicada no Sanity a cada visita. O player modal só entra na página depois do clique em "Assistir".
- **Cloudflare Pages / Vercel** hospeda o site compilado (`dist`).

## Como rodar localmente

Na pasta `site/`:
```bash
npm install
npm run dev
```

Para gerar a versão de produção:
```bash
npm run build
```

Dois tipos de conteúdo foram criados: **Novela** (seção "Novelas") e **Filme ou clipe** (seção "Filmes e clipes").

## O que tem na pasta

```
studio/schemaTypes/   index.js, novela.js, filme.js   (copiar para o seu Studio)
site/index.html       o site, com ganchos data-cms nas duas seções
site/js/cms-config.js onde você coloca o projectId e o dataset
site/js/cms.js        busca no Sanity, cache, renderização e player
site/assets/          fotos da equipe e vídeo da capa
```

Falta copiar do seu repositório atual o arquivo `politica-de-privacidade.html` (o rodapé aponta para ele).

## Passo a passo

### 1. Criar o projeto Sanity
1. Crie uma conta em sanity.io e rode, na pasta onde ficará o painel:
   `npm create sanity@latest -- --project-id SEU_ID --dataset production --template clean --typescript false`
2. Anote o **Project ID** e o nome do dataset (`production`). O dataset precisa estar como **público** (é o padrão), porque o site lê sem senha.
3. Em **Manage > API > CORS origins**, adicione o domínio do site e o endereço de teste (por exemplo `http://localhost:5173`). **Não** marque "Allow credentials".

### 2. Colocar os tipos no painel
1. Copie `studio/schemaTypes/index.js`, `novela.js` e `filme.js` para a pasta `schemaTypes/` do Studio, substituindo o `index.js` gerado.
2. Rode `npm run dev`, abra o Studio e cadastre 3 novelas e 3 filmes de teste, deixando um de cada com **Publicado** desmarcado.

### 3. Ligar o site
1. Abra `site/js/cms-config.js` e preencha o `projectId` (e o `dataset`, se mudou).
2. Teste localmente com um servidor simples, por exemplo `python -m http.server 5173` dentro da pasta `site`.

Enquanto o `projectId` estiver vazio, o site funciona em **modo demonstração**: mostra os 6 filmes e as 6 novelas de teste que já estão no `index.html`. Quando o Sanity responde, o aviso "(Conteúdo de teste)" some sozinho.

### 4. Painel para o cliente
- **Hospedado pelo Sanity (mais simples):** na pasta do Studio, rode `npx sanity deploy` e escolha um nome. O painel fica em `nome.sanity.studio`.
- **No Cloudflare Pages:** rode `npm run build` no Studio e publique a pasta `dist` em um projeto separado.

Convide o cliente em **Manage > Members** com o papel **Editor**. Esse papel cria, edita e publica conteúdo, mas não mexe nas configurações do projeto.

### 5. Publicar o site
No Cloudflare Pages, publique a pasta `site` (sem comando de build; diretório de saída: `site`). O `projectId` não é segredo, então pode ficar no arquivo de configuração. Alterações feitas no painel aparecem no site sem novo deploy; como a API usa CDN, pode levar alguns segundos.

## Como o site se comporta

| Situação | Filmes e clipes | Novelas |
|---|---|---|
| Sanity responde com itens | mostra os itens | mostra os itens |
| Sanity responde sem itens publicados | esconde a seção | esconde a seção |
| Sanity fora do ar, com visita anterior | usa a última lista salva no navegador | usa a última lista salva |
| Sanity fora do ar, sem visita anterior | mostra os 6 filmes escritos no HTML | esconde a seção |

O conteúdo vindo do Sanity é inserido como texto (nunca como HTML), e só links `https` de YouTube e Vimeo abrem no player. Links de vídeo "Ocultos" do Vimeo (com código no final, como `vimeo.com/123456789/abcdef1234`) funcionam. Se o link não puder ser incorporado, o botão abre o vídeo em outra aba.

## Checklist de testes
- Publicado aparece; com Publicado desmarcado não aparece.
- Editar título, sinopse e ordem no painel reflete no site.
- Despublicar ou excluir remove o item.
- Link do YouTube, do Vimeo e do Vimeo oculto abrem no player; Esc fecha e o som para.
- Novela sem capa mostra o fundo padrão; com capa mostra a foto em preto e branco.
- Layout com 1, 3 e 6 ou mais itens, e com títulos longos.
- Celular: um toque abre a ficha da novela; "Assistir" abre o player.
- Desligar a internet depois de uma visita: as seções continuam com a última lista.
- Vimeo: o embed só no domínio do cliente funciona se o plano do Vimeo permitir esse recurso.

## Guia rápido para o cliente
**Adicionar uma novela**
1. Entre no painel e clique em **Novela > Criar**.
2. Preencha título, gênero, episódios, situação e sinopse (até 220 caracteres).
3. Envie uma capa vertical (opcional) e cole o link do YouTube ou do Vimeo (opcional).
4. Escolha a ordem (o menor número aparece primeiro), marque **Publicado** e clique em **Publicar**.

**Adicionar um filme ou clipe:** o mesmo caminho, em **Filme ou clipe > Criar**, com título, descrição e link do vídeo.

**Esconder um item:** abra o item, desmarque **Publicado** e publique de novo.

## Pontos de atenção
- **Vimeo:** o plano gratuito tem limites de armazenamento e upload, e restringir o embed ao seu domínio costuma exigir plano pago. Para esconder o vídeo das buscas, use "Oculto no Vimeo". Confirme no plano atual antes de prometer custo zero.
- **Sanity:** o plano gratuito atende bem este uso. Confira os limites vigentes no site deles.
- **Próximos passos opcionais:** categorias de novela, página própria com episódios, capa automática do Vimeo quando não houver imagem.
