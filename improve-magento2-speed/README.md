# IMPROVE MAGENTO 2 SPEED

### Cache

Tenha certeza de que as páginas estão sendo cacheadas corretamente.
https://devdocs.magento.com/cloud/cdn/trouble-fastly.html
Verifique se o cache dá `MISS` no primeiro request e nos demais pra mesma página `HIT`, se não der `HIT` pode ser que tenha algum bloco xml na página com a tag `cachable="false"`, ai ele impede o magento de gerar o cache dessa página, um comando útil para encontrar possíveis situações dessas é o:
```
grep -rnw 'vendor/' -e 'cacheable="false"'
```
Instale o varnish, o varnish é uma camada que antecede a aplicação do magento, por isso consegue ter uma performance superior ao FPC nativo do magento.
Dê uma olhada nesse tutorial de como instalar e configurar o varnish: https://github.com/lima195/varnish-magento-2

### Imagens

Em imagens temos 4 pontos importantes:

- 1 - Tamanho
Tamanho é documento quando se trata de tamanho da imagens, o https://gtmetrix.com/ consegue gerar as imagens com tamanhos corretos e reduzidos para fazer a substituição no site.

- 2 - Lazyload
O Lazyload atrasa o carramento da imagem para a visão do usuário, existem alguns módulos disponíveis para isso.

- 3 - Webp
O webp é um formato optimizado para web, porém, não são todos os navegadores que rodam, por isso não pdoemos simplesmente usar o webp, precisamos converter as imagens para webp e usar a tag <picture> para decidir qual imagem será exibida de acordo com o navegador ou tamanho da tela do usuário.
```
<picture>
  <source srcset="img/awesomeWebPImage.webp" type="image/webp">
  <source srcset="img/creakyOldJPEG.jpg" type="image/jpeg"> 
  <img src="img/creakyOldJPEG.jpg" alt="Alt Text!">
</picture>
```


- 4 - Height e Width
Definir o Height e Width é importante para que o navegador aloque espaço da imagem na tela, reduzindo a alteração de layout conforme o site é carregado, isso é um dos critérios mais pesados do Pagespeed.

# HTTPS v2

Certifique-se de que o nginx com ssl tenha essa o v2:
```bash
listen 443 ssl http2;
```

# Remoção de módulos desnecessários

É interessante remover os módulos desnecessários via composer para que o magento carregue menos arquivos incluindo css e javascripts, abaixo vou listar alguns que removi na versão 2.3.5-p1, e vai da necessidade do lojista voltar com alguns desses módulos, conforme for removendo, é importante rodar o di:compile para verificar se tem alguma dependência quebrada.
```json
"replace":  {
 "magento/module-dhl":  "*",
 "magento/module-fedex":  "*",
 "magento/module-ups":  "*",
 "magento/module-usps":  "*",
 "magento/module-braintree":  "*",
 "magento/module-authorizenet":  "*",
 "magento/module-marketplace":  "*",
 "magento/module-multishipping":  "*",
 "magento/module-send-friend":  "*",
 "magento/module-release-notification":  "*",
 "magento/module-new-relic-reporting":  "*",
 "magento/module-sample-data":  "*",
 "magento/module-marketplace":  "*",
  
 "magento/module-analytics":  "*",
 "magento/module-catalog-analytics":  "*",
 "magento/module-customer-analytics":  "*",
 "magento/module-review-analytics":  "*",
 "magento/module-sales-analytics":  "*",
 "magento/module-wishlist-analytics":  "*",
  
 "magento/module-bundle-graph-ql":  "*",
 "magento/module-catalog-url-rewrite-graph-ql":  "*",
 "magento/module-cms-graph-ql":  "*",
 "magento/module-cms-url-rewrite-graph-ql":  "*",
 "magento/module-configurable-product-graph-ql":  "*",
 "magento/module-downloadable-graph-ql":  "*",
 "magento/module-grouped-product-graph-ql":  "*",
 "magento/module-quote-graph-ql":  "*",
 "magento/module-store-graph-ql":  "*",
 "magento/module-swatches-graph-ql":  "*",
 "magento/module-tax-graph-ql":  "*",
 "magento/module-url-rewrite-graph-ql":  "*",
 "magento/module-weee-graph-ql":  "*",
 "magento/module-catalog-customer-graph-ql":  "*",
 "magento/module-customer-downloadable-graph-ql":  "*",
 "magento/module-advanced-pricing-import-export":  "*",
 "magento/module-bundle-import-export":  "*",
 "magento/module-configurable-import-export":  "*",
 "magento/module-customer-import-export":  "*",
 "magento/module-downloadable-import-export":  "*",
 "magento/module-grouped-import-export":  "*",
 "magento/module-tax-import-export":  "*",
 "dotmailer/dotmailer-magento2-extension":  "*",
 "dotmailer/dotmailer-magento2-extension-chat":  "*",
 "klarna/module-kp":  "*",
 "klarna/module-ordermanagement":  "*",
 "klarna/module-core":  "*",
 "amzn/amazon-pay-sdk-php":  "*",
 "amzn/amazon-pay-and-login-with-amazon-core-module":  "*",
 "amzn/login-with-amazon-module":  "*",
 "amzn/amazon-pay-module":  "*",
 "vertex/module-tax":  "*",
 "vertex/sdk":  "*",
 "vertex/module-address-validation":  "*",
 "yireo/magento2-replace-inventory":  "*",
 "magento/module-graph-ql-cache":  "*",
 "magento/module-authorizenet-graph-ql":  "*",
 "magento/module-braintree-graph-ql":  "*",
 "magento/module-directory-graph-ql":  "*",
 "magento/module-paypal-graph-ql":  "*",
 "magento/module-related-product-graph-ql":  "*",
 "magento/module-send-friend-graph-ql":  "*",
 "magento/module-wishlist-graph-ql":  "*",
 "magento/module-inventory-export-stock":  "*",
 "magento/module-inventory-reservation-cli":  "*",
 "magento/google-shopping-ads":  "*",
 "magento/module-inventory-advanced-checkout":  "*",
 "magento/module-inventory-requisition-list":  "*",
 "magento/module-admin-analytics":  "*",
 "magento/module-adobe-ims":  "*",
 "magento/module-adobe-ims-api":  "*",
 "magento/module-adobe-stock-admin-ui":  "*",
 "magento/module-adobe-stock-asset":  "*",
 "magento/module-adobe-stock-asset-api":  "*",
 "magento/module-adobe-stock-client":  "*",
 "magento/module-adobe-stock-client-api":  "*",
 "magento/module-adobe-stock-image":  "*",
 "magento/module-adobe-stock-image-admin-ui":  "*",
 "magento/module-adobe-stock-image-api":  "*",
 "magento/module-catalog-cms-graph-ql":  "*"
 }
```

# CSS

O pagespeed quer que css não utilizado na página não seja exibido, no magento então é necessário separar do css principal todos os estilos que não vão para todas as páginas e passar a chamar via xml os css combinados para determinadas páginas, ex: categoria, checkout, minha conta, cms...
```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <css src="Namespace_YourModule::css/mycustom.css" after="-" />
    </head>
</page>
```

# Javascript

1 - Atualmente o pagespeed tem focado em desperdício de arquivos, poucos kb podem ser dolorosos para o navegador compilar, o bundling do magento que juntava vários javascripts em arquivos de no máximo x kb a fim de performar acabou se tornando uma má prática e o pagespeed vem desincentivando essa prática, por isso, o recomendado seria essas configurações:
```bash
bin/magento config:set dev/js/minify_files 0
bin/magento config:set dev/js/enable_js_bundling 0
bin/magento config:set dev/js/merge_files 0
```

Também é recomendado habilitar a opção de assinatura, pois como vamos trabalhar com o cache mais pesado, é interessante os arquivos estáticos serem atualizados conforme deploy e essa opção cria o timestamp na url, então a cada deploy é gerado uma url 'diferente' pro arquivo de css e js, fazendo com que o cache do navegador seja renovado.
```bash
bin/magento config:set dev/static/sign 1
```


2 - Instalação e configuração do baler e terser:
https://github.com/lima195/improve-magento2-speed/blob/master/baler_terser/README.md
(Eu criei um docker que já tem instalado o npm pronto pra rodar com o baler e o terser)
https://github.com/lima195/docker-lemp/tree/magento234baler
Lembrando que é na branch magento234baler
É necessário que faça o git clone do baler e instale/configure o módulo do magento_baler no modo production, então terá acesso aos comandos:

```bash
bin/baler @arg
bin/terser @theme
```


3 - Feito tudo isso, e removendo via composer os módulos do magento afim de carregar menos javascripts chega o momento de ter que fazer isso de forma manual.


# Preload fonts
https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/css-topics/using-fonts.html#font-head-type



# HTML

Essa é a tarefa mais difícil, o pagespeed recomenda que cada página tenha no máximo 1500 elementos html presentes na página, é um trabalho bastante manual e sem macetes, mas uma das coisas que ajuda bastante é combinar javascript e css na medida do possível, lembrando que ao combinar um css ou js e ele não ser utilizado na página, o google vai descontar nota de qualquer maneira, então esse seria o item menos prioritário da lista.
