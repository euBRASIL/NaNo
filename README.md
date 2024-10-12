# NanoCrawler

Uma interface de usuário leve da web para visualizar informações em tempo real sobre seu nó Nano e explorar a rede Nano.

## O que é Nano?

O objetivo do Nano é se tornar "uma moeda global com transações instantâneas e taxas zero em uma rede segura e descentralizada". Mais informações estão disponíveis no [repositório Nano](https://github.com/nanocurrency/raiblocks) oficial.

## Instalação

Primeiro, clone este repositório no servidor onde você pretende hospedar o site. Não precisa ser o mesmo servidor do nó Nano, mas certamente pode ser se você quiser.

Execute `yarn` para instalar dependências.

Há 2 arquivos de configuração que você precisa atualizar.

### Configuração do servidor de API

O servidor é responsável por fazer proxy de solicitações entre o site e seu nó Nano. Você nunca deve expor seu RPC do nó Nano ao público, e é por isso que temos um servidor que expõe apenas determinados endpoints. Ele também faz um pequeno processamento na resposta bruta do Nano RPC e armazena em cache as respostas com o Redis.

Há uma configuração padrão completa disponível na pasta de exemplos. Copie `server-config.json` para a raiz do projeto. Atualize todos os valores para se adequarem ao seu ambiente.

**O suporte ao Redis é opcional, mas recomendado.** Se desejar ignorá-lo, você pode excluir com segurança a entrada de configuração.

### Configuração do cliente

O front-end da web precisa saber onde o servidor da API pode ser alcançado. Copie `client-config.json` dos exemplos para a pasta `src` e atualize o arquivo de configuração para se adequar ao seu ambiente.

O [servidor websocket](https://github.com/meltingice/nanovault-ws) é opcional, mas você pode usar o servidor websocket hospedado que está definido como padrão na configuração. Dependendo do status de sincronização do seu nó, você pode receber blocos do servidor websocket antes que seu nó os confirme, e é por isso que hospedar um você mesmo é o ideal. Remova a entrada de configuração para desabilitar o websocket completamente.

## Desenvolvimento

Para executar o NanoCrawler no modo de desenvolvimento, basta executar `yarn start`. Isso iniciará o servidor de API e o servidor de desenvolvimento webpack para o front-end. Isso não inicia nenhum dos trabalhos recorrentes, mas você pode executá-los manualmente se precisar dos dados que eles fornecem (veja abaixo).

## Hospedagem de produção

Depois que a configuração for definida, você pode compilar o projeto com `yarn deploy`.

Isso compilará e produzirá todos os arquivos estáticos do site no diretório `html`. Isso é preferível ao uso de `yarn build` porque o processo de compilação começa excluindo o diretório de compilação, o que pode fazer com que o site seja interrompido para qualquer visitante durante o processo de compilação. Sempre que você alterar a configuração do cliente ou extrair quaisquer alterações do git, precisará reconstruir o projeto. A partir daqui, você pode hospedar os arquivos estáticos do site em qualquer lugar. Pode ser em um servidor doméstico, [Heroku](https://github.com/mars/create-react-app-buildpack), um droplet DigitalOcean, etc. Se você não estiver no Heroku, recomendo fortemente hospedar os arquivos estáticos com Nginx.

Uma coisa importante a ser observada é que todo o roteamento do site é feito do lado do cliente. Isso significa que você precisa fazer uma de duas coisas: configurar seu servidor web para sempre servir `index.html` independentemente do caminho da URL ou alternar para o roteamento baseado em hash.

**Exemplo de configuração do Nginx**

```nginx
server {
root /path/to/build_dir;

index index.html;
server_name nano.yourdomain.com;

# Esta é a parte importante que retornará ao carregamento de index.html
location / {
try_files $uri $uri/ /index.html =404;
}
}
```

**Mudando para roteador baseado em hash**

Embora eu recomende fortemente hospedar por meio de um servidor web adequado, como último recurso, você pode mudar para roteamento baseado em hash. Abra `src/index.js` e altere `BrowserRouter` para `HashRouter`. Execute `yarn build` para obter uma versão atualizada do site. Agora, em vez de `/explorer`, a URL será `/#explorer`.

### Hospedando o servidor

Os componentes do lado do servidor são divididos em vários processos para separar o servidor de API de alguns trabalhos recorrentes de longa duração. Eles são compostos por:

- `server.api.js` - o servidor de API
- `server.peers.js` - tarefa recorrente para descobrir e buscar dados de outros monitores Nano
- `server.top-accounts.js` - tarefa recorrente para construir a lista de contas principais
- `server.tps.js` - tarefa recorrente para registrar a contagem de blocos atual para calcular blocos/seg

Existem várias opções para hospedar um servidor NodeJS. Se você tem experiência com uma opção, sinta-se à vontade para usá-la. Todos os scripts podem ser executados diretamente com `node` e todos eles usam o mesmo `server-config.json`.

Eu uso e recomendo [PM2](https://www.npmjs.com/package/pm2) para gerenciar servidores NodeJS. Há um arquivo `ecosystem.config.js` incluído, então tudo o que você precisa fazer é executar `pm2 start ecosystem.config.js --env production` para iniciar todos os processos. O servidor API iniciará no modo cluster com 4 processos por padrão. Sinta-se à vontade para ajustar isso no arquivo `ecosystem.config.js`.

## Localização

O NanoCrawler pretende estar disponível em tantos idiomas quanto possível. Se você quiser contribuir com traduções, por favor
