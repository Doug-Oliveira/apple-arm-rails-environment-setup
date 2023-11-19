# Mac Arm setup

- Add code bellow to .zshrc

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

- Install brew compatible with x86 version
Reference post ->
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
- Install command line tols for xCode 
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

- Check bundler version of the project.

```
gem install bundler -v 1.13.0
```
-----
## Install Redis
```
axbrew install redis
axbrew services start redis
```
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
## Rubyracer (older rails projects)
```
axbrew install v8-315
```
```
gem install therubyracer -v '0.12.2' -- --with-v8-dir=/usr/local/homebrew/opt/v8@3.15
```
- Therubyracer gem adds a JavascriptRuntime to the rails project, which is mainly used to precompile assets. however, it currently depends on lib v8 to work, but its installation has been discontinued because it depends on phyton 2.

A workaround for this problem was to comment out therubyracer in the project's Gemfile and add JavascriptRuntime via node.

Reference post -> https://www.rubyonmac.dev/how-to-install-therubyracer-on-m1-m2-apple-silicon-mac
```
axbrew install nvm
nvm install version
```
```
npm install
```
----
The End! 👏👏👏





