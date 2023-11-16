# mac Arm setup

- adicionar código abaixo ao .zshrc

```
export PATH=$HOME/bin:/usr/local/bin:$PATH

# for intel x86_64 brew
alias axbrew='arch -x86_64 /usr/local/homebrew/bin/brew'

# Homebrew
export PATH=$HOME/bin:/usr/local/homebrew/bin:$PATH
export PATH=$HOME/bin:/usr/local/homebrew/sbin:$PATH

# openssl
export PATH="/usr/local/homebrew/opt/openssl@1.0/bin:$PATH"
export LDFLAGS="-L/usr/local/homebrew/opt/openssl@1.0/lib"
export CPPFLAGS="-I/usr/local/homebrew/opt/openssl@1.0/include"
export PKG_CONFIG_PATH="/usr/local/homebrew/opt/openssl@1.0/lib/pkgconfig"
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=/usr/local/homebrew/opt/openssl@1.0"

# rbenv
export RBENV_ROOT=$(axbrew --prefix rbenv)
export PATH=$RBENV_ROOT/bin:$PATH
eval "$(arch -x86_64 rbenv init -)"
export PATH="/usr/local/homebrew/opt/postgresql@10/bin:$PATH"
```

- instalar brew compativel com a versao x86
link de referencia =>
https://medium.com/mkdir-awesome/how-to-install-x86-64-homebrew-packages-on-apple-m1-macbook-54ba295230f
------
## Rbenv install
```
axbrew install rbenv
```
## Openssl Setup

- Install openssl@1.0 via brew

```
axbrew install rbenv/tap/openssl@1.0
rbenv rehashq
```
- Check installation
```
echo $(axbrew --prefix openssl@1.0)
/usr/local/homebrew/opt/openssl@1.0
```
---
- Instalar command line tols for xCode 
https://developer.apple.com/download/all/
```
axbrew install readline
echo $(axbrew --prefix readline)
/usr/local/homebrew/opt/readline
```

```
axbrew install ruby-build
axbrew install libyaml
axbrew install gmp
axbrew install zlib
```
- Install Ruby 💎
  
```
rbenv global 2.1.3
```
```
RUBY_CFLAGS="-Wno-error=implicit-function-declaration" RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(axbrew --prefix openssl@1.0)" arch -x86_64 rbenv install 2.1.3
```

- Verificar qual a versao espeficifica do bundle do projeto BUNDLED WITH no Gemfile.lock neste caso iremos usar a 1.13.0

```
gem install bundler -v 1.13.0
```
-----
## Postgresql 🐘
```
axbrew install postgresql@10
```

```
echo 'export PATH="/usr/local/homebrew/opt/postgresql@10/bin:$PATH"' >> ~/.zshrc
```

```
axbrew services list
```

```
axbrew services start postgresql@10
```
```
axbrew install libpq
```

```
echo $(axbrew --prefix libpq)
```

```
ARCHFLAGS="-arch x86_64" sudo gem install pg -v 0.17.0 -- --with-pg-config=/usr/local/homebrew/opt/postgresql@10/bin/pg_config --with-pq-dir="/usr/local/homebrew/opt/libpq"
```
----
## Rubyracer
```
axbrew install v8-315
```
```
gem install therubyracer -v '0.12.2' -- --with-v8-dir=/usr/local/homebrew/opt/v8@3.15
```
- A gem therubyracer adiciona um JavascriptRuntime ao projeto rails, que é utilizado principalmente para fazer a pré-compilação de assets.
  porém atualmente ela depende da lib v8 para funcionar, porém a instalacao dela foi descontinuada por que ela depende do pyton 2.

Um workaround para este problema foi comentar o therubyracer no Gemfile do projeto.

Adicionando node no projeto com nvm, resolvemos o problema do projeto ficar sem JavascriptRuntime

referencia -> https://www.rubyonmac.dev/how-to-install-therubyracer-on-m1-m2-apple-silicon-mac
```
axbrew install nvm
nvm install version
```
```
npm install
```
----
The End! 👏👏👏





