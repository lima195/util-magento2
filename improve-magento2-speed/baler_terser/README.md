### 1 - Instalando o nvm:

```bash
sudo apt install build-essential

touch ~/.bash_profile
curl -o- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

Se por acaso não reconhecer o nvm no zsh, é só editar o arquivo `~/.zshrc` e inserir dentro `source ~/.nvm/nvm.sh` ou no `~/.profile`. Ou tente com o bash normal mesmo...

confere se está instalado:

```bash
nvm -v
```

liste as opçoes de node:

```bash
nvm list-remote
```

instale a que quiser >= 10:

```bash
nvm install v14.16.0
```

confere a versão que está rodando:

```bash
node -v
```

instalando o nodejs:

```bash
curl -fsSL https://deb.nodesource.com/setup_15.x | sudo -E bash -
sudo apt install nodejs
```

verificando a versão do nodejs:

```bash
nodejs -v
```

a esse ponto temos:

```bash
➜   nodejs -v
v15.13.0

➜   npm -v
6.14.11

➜   node -v
v14.16.0
```

### 2 - Baler

Clonando a pasta do baler para dentro do magento

```bash
git clone https://github.com/magento/baler.git  
```

vá para dentro da pasta do baler (`./magento_root/baler`) e rode

```bash
npm install  
```

talvez seja necessário rodar

```bash
npm audit fix --force
```

vá para dentro da pasta `./magento_root/baler/bin` e rode

```bash
./baler
```

Caso receba um erro como: 'Error: Cannot find module '../dist/cli/initializeCLI', apenas rode na pasta bin mesmo:

```bash
npm run build  
```

Testando o funcionamento do baler

```bash
./baler --help
```

Pega o nome do tema, por exemplo:
```bash
cat ../../vendor/Alothemes/gecko9/registration.php
# vai retornar um Alothemes/gecko9
# é só olhar o registration.php e tirar o `frontend/`
```

Compilar os assets
```bash
./baler build --theme Alothemes/gecko9
```
ou
```bash
./baler build
```

Instalar o módulo do baler no magento
```bash
composer config repositories.magento-baler vcs git@github.com:magento/m2-baler.git
composer require magento/module-baler:dev-master
bin/magento module:enable Magento_Baler
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento c:c
```

Atualize as configurações

```bash
bin/magento config:set dev/js/minify_files 0
bin/magento config:set dev/js/enable_js_bundling 0
bin/magento config:set dev/js/merge_files 0
bin/magento config:set dev/js/enable_baler_js_bundling 1  
```

---

### 3 - Terser

```bash
npm install -g terser;
THEME_FOLDER=('pub/static/frontend/Alothemes/gecko9/');
find ${THEME_FOLDER[@]} \( -name '*.js' -not -name '*.min.js' -not -name 'core-bundle.js' -not -name 'requirejs-bundle-config.js' \) -exec terser \{} -c -m reserved=['$','jQuery','define','require','exports'] -o \{} \;
```
ooou
```bash
npm install -g terser;

find pub/static/frontend/Alothemes/gecko9/ \( -name '*.js' -not -name '*.min.js' -not -name 'core-bundle.js' -not -name 'requirejs-bundle-config.js' \) -exec terser \{} -c -m reserved=['$','jQuery','define','require','exports'] -o \{} \;
```

---

### 4 - Fluxo de deploy

```bash
## 0 - config <-- pode ser feito só uma vez... não precisa estar em todos os deploys
bin/magento config:set dev/js/minify_files 0
bin/magento config:set dev/js/enable_js_bundling 0
bin/magento config:set dev/js/merge_files 0
bin/magento config:set dev/js/enable_baler_js_bundling 1  

## 1 - deploy magento
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento deploy:mode:set production

## 2 - baler
./baler/bin/baler build;

## 3 - terser <-- Lembrando que aqui depende do tema.
find pub/static/frontend/Alothemes/gecko9/ \( -name '*.js' -not -name '*.min.js' -not -name 'core-bundle.js' -not -name 'requirejs-bundle-config.js' \) -exec terser \{} -c -m reserved=['$','jQuery','define','require','exports'] -o \{} \;
```

---

### 5 - script bash pro deploy:
> assets.js
```bash
#!/bin/bash
[ -z "$1" ] && echo "Please specify a theme (ex. Magento/luma)" && exit

#baler
echo ".:Baler - Bundling and minify files:.";
#npm --prefix ./baler/ install;
#npm --prefix ./baler/ audit fix --force;
npm --prefix ./baler/ run build;
./baler/bin/baler build;

# terser
echo ".:Terser - minify all JavaScript files Baler did not already minify:.";
find pub/static/frontend/"$@"/ \( -name '*.js' -not -name '*.min.js' -not -name 'core-bundle.js' -not -name 'requirejs-bundle-config.js' \) -exec terser \{} -c -m reserved=['$','jQuery','define','require','exports'] -o \{} \;
```

uso:
```bash
- ./assets.sh Alothemes/gecko9
```

---

### Possíveis problemas:

- Não encontrar o comando terser, seja no uso normal ou via pipeline, o motivo é que o npm foi instalado via nvm e precisa ser incluído o código no ~/.bash, ~/.zshrc, ~/.profile ou algum outro na pasta do usuário


Referências:

https://nemanja.io/optimize-magento-2-store-using-baler-method/
https://www.integer-net.com/magento-2-javascript-bundling-baler-devtools/
