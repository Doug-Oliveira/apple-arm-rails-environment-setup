# macm1-setup

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
## Openssl Setup
PS: Tentar instalar a versão mais antiga do openssl conforme tutorial abaixo, 
    caso não de certo, pode seguir com o resto do setup por que ao instalar o postgresql@10,
    ele por default já instala o openssl@3.0.

- Instalar openssl@1.0 via axbrew
link de referencia => https://www.davidseek.com/ruby-on-m1/
 
- criar arquivo executavel no diretorio de sua preferencia, com o nome openssl@1.0.rb e colar conteudo abaixo:
```
# Extraced from pre- https://github.com/Homebrew/homebrew-core/pull/46876
class OpensslAT10 < Formula
  desc "SSL/TLS cryptography library"
  homepage "https://openssl.org/"
  url "https://www.openssl.org/source/openssl-1.0.2u.tar.gz"
  mirror "https://www.mirrorservice.org/sites/ftp.openssl.org/source/openssl-1.0.2u.tar.gz"
  sha256 "ecd0c6ffb493dd06707d38b14bb4d8c2288bb7033735606569d8f90f89669d16"

  keg_only :provided_by_macos,
    "Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries"

  # Add darwin64-arm64-cc & debug-darwin64-arm64-cc build targets.
  patch :DATA

  def install
    # OpenSSL will prefer the PERL environment variable if set over $PATH
    # which can cause some odd edge cases & isn't intended. Unset for safety,
    # along with perl modules in PERL5LIB.
    ENV.delete("PERL")
    ENV.delete("PERL5LIB")

    ENV.deparallelize
    args = %W[
      --prefix=#{prefix}
      --openssldir=#{openssldir}
      no-ssl2
      no-ssl3
      no-zlib
      shared
      enable-cms
      darwin64-#{Hardware::CPU.arch}-cc
      enable-ec_nistp_64_gcc_128
    ]
    system "perl", "./Configure", *args
    system "make", "depend"
    system "make"
    system "make", "test"
    system "make", "install", "MANDIR=#{man}", "MANSUFFIX=ssl"
  end

  def openssldir
    etc/"openssl"
  end

  def post_install
    keychains = %w[
      /System/Library/Keychains/SystemRootCertificates.keychain
    ]

    certs_list = `security find-certificate -a -p #{keychains.join(" ")}`
    certs = certs_list.scan(
      /-----BEGIN CERTIFICATE-----.*?-----END CERTIFICATE-----/m,
    )

    valid_certs = certs.select do |cert|
      IO.popen("#{bin}/openssl x509 -inform pem -checkend 0 -noout", "w") do |openssl_io|
        openssl_io.write(cert)
        openssl_io.close_write
      end

      $CHILD_STATUS.success?
    end

    openssldir.mkpath
    (openssldir/"cert.pem").atomic_write(valid_certs.join("\n") << "\n")
  end

  def caveats; <<~EOS
    A CA file has been bootstrapped using certificates from the SystemRoots
    keychain. To add additional certificates (e.g. the certificates added in
    the System keychain), place .pem files in
      #{openssldir}/certs

    and run
      #{opt_bin}/c_rehash
  EOS
  end

  test do
    # Make sure the necessary .cnf file exists, otherwise OpenSSL gets moody.
    assert_predicate HOMEBREW_PREFIX/"etc/openssl/openssl.cnf", :exist?,
            "OpenSSL requires the .cnf file for some functionality"

    # Check OpenSSL itself functions as expected.
    (testpath/"testfile.txt").write("This is a test file")
    expected_checksum = "e2d0fe1585a63ec6009c8016ff8dda8b17719a637405a4e23c0ff81339148249"
    system "#{bin}/openssl", "dgst", "-sha256", "-out", "checksum.txt", "testfile.txt"
    open("checksum.txt") do |f|
      checksum = f.read(100).split("=").last.strip
      assert_equal checksum, expected_checksum
    end
  end
end
```

- executar comando abaixo para instalar o openssl a partir do arquivo criado.

```
axbrew install ~/LocalPath/openssl@1.0.rb
```

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
- Instalar versao antiga do ruby 💎
  
  PS: caso não tenha conseguido instalar o opessl@1.0, não precisa passar a flag ```RUBY_CONFIGURE_OPTS='--with-openssl-dir=/usr/local/homebrew/opt/openssl@1.0'```
```
RUBY_CFLAGS="-Wno-error=implicit-function-declaration" RUBY_CONFIGURE_OPTS='--with-readline-dir=/usr/local/homebrew/opt/readline' RUBY_CONFIGURE_OPTS='--with-openssl-dir=/usr/local/homebrew/opt/openssl@1.0' arch -x86_64 rbenv install 2.1.3
```
```
axbrew install rbenv
```
```
rbenv global 2.1.3
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
```
axbrew install nvm
```
----
The End! 👏👏👏





