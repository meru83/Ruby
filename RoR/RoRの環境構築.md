Dockerを使用してRuby on Rails（以下、RoR）の開発環境を構築する手順を詳しく解説します。この方法により、ローカル環境に直接RubyやRailsをインストールすることなく、コンテナ内で開発環境を整えることができます。

### 1. Dockerのインストール

まず、Dockerをインストールします。以下の手順で進めてください。

- **公式サイトからダウンロード**: [Docker公式サイト](https://www.docker.com/)にアクセスし、使用しているOSに対応したDocker Desktopをダウンロードします。

- **インストール手順**:
  - **Windowsの場合**: ダウンロードしたインストーラーを実行し、画面の指示に従ってインストールします。 citeturn0search6
  - **macOSの場合**: ダウンロードした`Docker.dmg`ファイルを開き、アプリケーションフォルダにドラッグ＆ドロップしてインストールします。
  - **Linuxの場合**: ディストリビューションに応じた手順でインストールします。

### 2. Dockerの起動と確認

インストール後、Docker Desktopを起動します。ターミナル（またはコマンドプロンプト）を開き、以下のコマンドでDockerが正しくインストールされているか確認します。

```bash
docker --version
```

バージョン情報が表示されれば、インストールは成功です。

### 3. プロジェクトディレクトリの作成

RoRプロジェクト用のディレクトリを作成し、そこに移動します。

```bash
mkdir myapp
cd myapp
```

### 4. 必要なファイルの作成

以下のファイルをプロジェクトディレクトリ内に作成します。

- **Dockerfile**: RoR環境を定義するファイル
- **docker-compose.yml**: 複数のコンテナを管理するための設定ファイル
- **Gemfile**: Rubyの依存関係を管理するファイル
- **Gemfile.lock**: Gemのバージョンを固定するファイル（空でOK）

**Dockerfile**:

```dockerfile
FROM ruby:3.1

# 必要なパッケージのインストール
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs

# 作業ディレクトリの設定
WORKDIR /myapp

# GemfileとGemfile.lockをコピー
COPY Gemfile Gemfile.lock ./

# ユーザーの作成
RUN groupadd -g 1000 rails && \
    useradd -u 1000 -g rails -m rails
# OR
# railsユーザーとグループの作成
# RUN groupadd --system --gid 1000 rails && \
#     useradd --uid 1000 --gid 1000 --create-home --shell /bin/bash rails


# 作業ディレクトリの所有者を変更
RUN chown -R rails:rails /myapp
# OR
# アプリケーションディレクトリの所有権をrailsユーザーに変更
# RUN chown -R rails:rails /rails/db /rails/log /rails/storage /rails/tmp


# ユーザーを切り替え
USER rails

# Bundlerのインストール
RUN gem install bundler && bundle install

# アプリケーションのソースコードをコピー
COPY . ./

# ポートの公開
EXPOSE 3000

# サーバーの起動コマンド
CMD ["rails", "server", "-b", "0.0.0.0"]
```

**docker-compose.yml**:

```yaml
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
  web:
    build: .
    user: "rails"
    command: ["rails", "server", "-b", "0.0.0.0"]
    volumes:
      - .:/myapp:cached
    ports:
      - "3000:3000"
    depends_on:
      - db
volumes:
  db-data:
```

**Gemfile**:

```ruby
source 'https://rubygems.org'
gem 'rails', '~> 7.0.0'
gem 'pg', '~> 1.2'
```

### 5. Railsアプリケーションの作成

以下のコマンドでRailsアプリケーションを作成します。

```bash
# 以下のどちらかのコマンドで成功した
docker-compose run web rails new . --force --database=postgresql

docker-compose run web rails new . --force --no-deps --database=postgresql
```

このコマンドにより、既存のファイルを上書きして新しいRailsアプリケーションが作成されます。

`docker-compose run web rails new . --force --no-deps` について、各部分の意味を詳しく説明いたします。

1. **`docker-compose run`**:
   `docker-compose` は、複数の Docker コンテナを一括で管理・起動するためのツールです。`run` コマンドは、指定したサービスのコンテナを一時的に起動し、特定のコマンドを実行する際に使用します。

2. **`web`**:
   これは、`docker-compose.yml` ファイル内で定義されたサービス名です。通常、Rails アプリケーションのコンテナを指します。このサービスを基にコンテナを起動し、指定されたコマンドを実行します。

3. **`rails new .`**:
   `rails new` は、新しい Rails アプリケーションを作成するコマンドです。`.`（ドット）は、現在のディレクトリにアプリケーションを作成することを指定しています。

4. **`--force`**:
   既存のファイルやディレクトリが存在する場合でも、強制的に上書きして新しいアプリケーションを作成します。これにより、既存のファイルがあってもエラーを避けてプロジェクトを初期化できます。

5. **`--no-deps`**:
   依存する他のサービス（例えば、データベースサービスなど）を起動せずに、指定したサービス（ここでは `web`）のみを起動してコマンドを実行します。`rails new` コマンドの実行時にはデータベース接続は不要なため、このオプションを指定します。

6. **`--database=postgresql`**: 
新しい ``Rails`` アプリケーションで使用するデータベースとして ``PostgreSQL`` を指定します。デフォルトでは ``SQLite`` が使用されますが、このオプションを指定することで、`config/database.yml` ファイルなどが ``PostgreSQL`` 用に設定されます。

まとめると、このコマンドは `docker-compose.yml` で定義された `web` サービスのコンテナを一時的に起動し、その中で `rails new .` コマンドを実行して、現在のディレクトリに新しい Rails アプリケーションを強制的に作成します。この際、他の依存サービスは起動せず、データベースとして PostgreSQL を使用する設定が行われます。

### 6. データベース設定の更新

`config/database.yml`ファイルを開き、以下のように編集します。

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development
```

### 7. コンテナの起動

以下のコマンドでコンテナを起動します。

```bash
docker-compose up --build
```

データベースを作成し、サーバーを起動します。

```bash
docker-compose run web rake db:create
docker-compose up
```


### 8. アプリケーションへのアクセス

ブラウザで`http://localhost:3000`にアクセスし、Railsのウェルカムページが表示されれば成功です。

以上の手順で、Dockerを使用したRoRの開発環境が構築できます。この方法により、環境の差異による問題を避け、一貫した開発環境 

---

### 9. おまけ

Rubyから直接データベースに接続し、データの操作を行うことも可能です。以下に、代表的なデータベースであるMySQLとPostgreSQLへの接続方法を説明します。

**1. MySQLへの接続**

MySQLに接続するためには、`mysql2`というgemを使用します。以下の手順で接続を行います。

1. **`mysql2` gemのインストール**

   ```bash
   gem install mysql2
   ```

2. **接続スクリプトの作成**

   ```ruby
   require 'mysql2'

   client = Mysql2::Client.new(
     host: 'localhost',
     username: 'your_username',
     password: 'your_password',
     database: 'your_database'
   )

   results = client.query('SELECT * FROM your_table')
   results.each do |row|
     puts row
   end
   ```

   上記のスクリプトでは、`Mysql2::Client.new`メソッドを使用してデータベースに接続し、`query`メソッドでSQLクエリを実行しています。

**2. PostgreSQLへの接続**

PostgreSQLに接続するためには、`pg`というgemを使用します。以下の手順で接続を行います。

1. **`pg` gemのインストール**

   ```bash
   gem install pg
   ```

2. **接続スクリプトの作成**

   ```ruby
   require 'pg'

   conn = PG.connect(
     dbname: 'your_database',
     user: 'your_username',
     password: 'your_password',
     host: 'localhost',
     port: 5432
   )

   result = conn.exec('SELECT * FROM your_table')
   result.each do |row|
     puts row
   end
   ```

   上記のスクリプトでは、`PG.connect`メソッドを使用してデータベースに接続し、`exec`メソッドでSQLクエリを実行しています。

**注意点**

- データベースに接続する際は、適切な認証情報（ユーザー名、パスワード）を使用してください。
- データベースサーバーが起動しており、ネットワーク経由で接続可能な状態であることを確認してください。
- セキュリティ上、認証情報をソースコード内に直接記述するのは避け、環境変数や設定ファイルを使用して管理することを推奨します。

**参考文献：**

- [Rubyでmysqlの操作 - Qiita](https://qiita.com/hukuro310/items/ec91beaecd11157e0640)
- [RubyでPostgreSQLを使うために知っておきたい最低限のこと（pg gem編） - Study Infra](https://study-infra.com/ruby-postgresql-pg-begginer/) 



### 10. **よくある問題と解決策**

- **PostgreSQLのバージョン不一致**:
    データベースの初期化エラーを解消するために、古いデータを削除し、新しいPostgreSQLバージョンに合わせて再初期化します。以下の手順を試してください。

    1. Docker ComposeでPostgreSQLのデータボリュームを確認し、削除します。

    1. 次に、Docker Composeを使用して再構築します。

    ```bash
    docker-compose down -v  # ボリュームを含むすべてのコンテナを停止し、削除します
    docker-compose up -d  # 新しいコンテナとボリュームを作成します
    ```

    このコマンドは、すべてのデータを再初期化し、新しいバージョンに適したデータベースを作成します。