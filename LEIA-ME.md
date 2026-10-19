# Envio dos resultados para um repositório privado

A implementação está pronta para configurar. O envio remoto só funciona depois dos passos abaixo. O HTML continua permitindo baixar um JSON sem o serviço.

## 1. Criar o destino no GitHub

Crie `jorgealberto188-creator/resultados-radiologia` com visibilidade **Private** e um README inicial. Não use o repositório público do site. O serviço recusa gravar em um repositório público.

Em GitHub → Settings da conta → Developer settings → Personal access tokens → Fine-grained tokens, crie um token com acesso apenas ao repositório privado de resultados e permissão **Contents: Read and write**. Defina uma validade adequada. Não cole o token no HTML, no GitHub ou nesta conversa.

## 2. Publicar o serviço em Cloudflare Workers

Requer uma conta Cloudflare e Node.js instalado. Abra um terminal nesta pasta:

```bash
npx wrangler login
npx wrangler deploy
npx wrangler secret put GITHUB_TOKEN
npx wrangler secret put CLASS_CODE
```

Os dois últimos comandos pedem os valores em particular. `CLASS_CODE` é um código de acesso da turma, diferente do token GitHub. Use um código difícil de adivinhar e troque-o entre turmas. O código identifica acesso à turma, não autentica a identidade do aluno.

Se escolher outro nome para o repositório, ajuste `GITHUB_REPOSITORY` em `wrangler.toml` e publique novamente. `ALLOWED_ORIGIN` é a origem do site, sem o caminho do repositório.

## 3. Conectar o HTML

No `index.html`, encontre:

```js
var ENVIO_CONFIG={url:''};
```

Substitua somente a URL pelo endereço publicado do Worker, incluindo `/respostas`:

```js
var ENVIO_CONFIG={url:'https://SEU-WORKER.SEU-SUBDOMINIO.workers.dev/respostas'};
```

Envie o `index.html` atualizado ao repositório do site. O aluno clica em **Finalizar e enviar**, informa o código da turma e confirma o envio. Um protocolo aparece apenas após confirmação do servidor. Sem configuração ou em caso de falha, o JSON continua disponível para download.

## 4. Consultar depois

Entre no repositório privado `resultados-radiologia` com sua conta GitHub e abra a pasta `respostas`. Cada submissão contém as 20 respostas objetivas, notas recalculadas no serviço e as 15 respostas de autorreflexão com feedback. Respostas incompletas ficam identificadas como parciais. Não é coletado nome; a sessão recebe um identificador aleatório. O aluno pode guardar o protocolo para localizar o envio.

Repetir o mesmo envio não cria outra cópia; respostas alteradas criam outro registro. O botão de apagar no teste só apaga dados da página, não registros já enviados. Exclusão no GitHub exige considerar também o histórico de commits. Defina acesso e prazo de retenção antes de usar com a turma.

## Limites e segurança

- O token fica exclusivamente nos secrets do Worker. O serviço não oferece uma rota pública de leitura dos resultados.
- O servidor valida a estrutura, ignora notas fornecidas pelo navegador e recalcula as notas a partir do catálogo. As respostas continuam sendo autodeclaradas, sem comprovação de autoria ou execução.
- CORS restringe o uso pelo navegador, mas não substitui autenticação. O código de turma é compartilhado e pode ser repassado. Para uso amplo, adicione autenticação individual e limitação de requisições no provedor antes de distribuir o link.
- Não use esses registros para diagnosticar personalidade, selecionar candidatos ou determinar aptidão profissional. São exercícios educativos.
- Alterou as perguntas? Atualize o catálogo do serviço e publique uma versão compatível antes de receber novos envios.

Referências: [Secrets do Worker](https://developers.cloudflare.com/workers/configuration/secrets/) · [API de arquivos do GitHub](https://docs.github.com/en/rest/repos/contents).
