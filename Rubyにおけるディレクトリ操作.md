

## Rubyにおけるディレクトリ操作の詳細解説

[Ruby.mdへ戻る](Ruby.md#2-ファイル操作)

Rubyでは、**Dirクラス**と**FileUtilsモジュール**を使用して、ディレクトリの作成、削除、移動、コピーなどの操作を行うことができます。以下に、主要なディレクトリ操作について詳しく解説します。

### 1. ディレクトリの作成

新しいディレクトリを作成するには、`Dir.mkdir`メソッドを使用します。

```ruby
# 単一のディレクトリを作成
Dir.mkdir('new_directory')

# 複数階層のディレクトリを一度に作成
require 'fileutils'
FileUtils.mkdir_p('parent_directory/child_directory')
```

`FileUtils.mkdir_p`を使用すると、指定したパスの中間ディレクトリが存在しない場合でも、一度にすべてのディレクトリを作成できます。

### 2. ディレクトリの削除

ディレクトリを削除するには、`Dir.rmdir`または`FileUtils.rm_r`を使用します。

```ruby
# 空のディレクトリを削除
Dir.rmdir('empty_directory')

# 中身があるディレクトリを削除
require 'fileutils'
FileUtils.rm_r('non_empty_directory')
```

`Dir.rmdir`は空のディレクトリのみ削除可能です。中身があるディレクトリを削除する場合は、`FileUtils.rm_r`を使用します。

### 3. ディレクトリの移動

ディレクトリを移動または名前を変更するには、`FileUtils.mv`を使用します。

```ruby
require 'fileutils'
# ディレクトリの移動
FileUtils.mv('source_directory', 'destination_directory')

# ディレクトリの名前変更
FileUtils.mv('old_directory_name', 'new_directory_name')
```

`FileUtils.mv`は、ディレクトリの移動と名前変更の両方に使用できます。

### 4. ディレクトリのコピー

ディレクトリをコピーするには、`FileUtils.cp_r`を使用します。

```ruby
require 'fileutils'
FileUtils.cp_r('source_directory', 'destination_directory')
```

`FileUtils.cp_r`は、ディレクトリ内の全ファイルとサブディレクトリを再帰的にコピーします。

### 5. ディレクトリの存在確認

指定したディレクトリが存在するか確認するには、`Dir.exist?`を使用します。

```ruby
if Dir.exist?('some_directory')
  puts 'ディレクトリは存在します。'
else
  puts 'ディレクトリは存在しません。'
end
```

`Dir.exist?`は、指定したパスがディレクトリとして存在する場合に`true`を返します。

### 6. ディレクトリの内容一覧取得

ディレクトリ内のファイルやサブディレクトリの一覧を取得するには、`Dir.entries`や`Dir.glob`を使用します。

```ruby
# すべてのエントリを取得
entries = Dir.entries('some_directory')
entries.each do |entry|
  puts entry
end

# 特定のパターンにマッチするファイルを取得
ruby_files = Dir.glob('some_directory/*.rb')
ruby_files.each do |file|
  puts file
end
```

`Dir.entries`は、指定したディレクトリ内のすべてのエントリ（`.`や`..`も含む）を配列で返します。`Dir.glob`は、ワイルドカードを使用して特定のパターンにマッチするファイルやディレクトリを取得できます。

### 7. カレントディレクトリの取得と変更

現在の作業ディレクトリ（カレントディレクトリ）を取得・変更するには、`Dir.pwd`と`Dir.chdir`を使用します。

```ruby
# カレントディレクトリの取得
current_directory = Dir.pwd
puts "現在のディレクトリ: #{current_directory}"

# カレントディレクトリの変更
Dir.chdir('new_working_directory')
puts "変更後のディレクトリ: #{Dir.pwd}"
```

`Dir.pwd`は現在のディレクトリを返し、`Dir.chdir`は指定したディレクトリに移動します。

これらのメソッドを活用することで、Rubyでのディレクトリ操作が効率的に行えます。

### 参考

- [【完全網羅】Rubyのディレクトリ・ファイル操作まとめ（dir, file）](https://www.sejuku.net/blog/14367)
  Rubyにおけるディレクトリやファイル操作の基本から応用までを網羅的に解説しています。

- [【Ruby入門】ディレクトリ（フォルダ）の作成、削除](https://style.potepan.com/articles/7357.html)
  Rubyでのディレクトリの作成や削除方法について、具体的なコード例とともに説明しています。

- [Rubyでのディレクトリ・ファイル操作](https://zenn.dev/kunosu/articles/7e01a31644575d0d50c2)
  Rubyを使ったディレクトリやファイルの操作方法を詳しく解説しています。

- [ファイル操作 - Ruby入門](https://ruby.open-code.club/%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E6%93%8D%E4%BD%9C/index.html)
  Rubyにおけるファイルやディレクトリの操作方法を基礎から学べる入門記事です。

- [Rubyでマスター！ディレクトリのファイル一覧を取得する5つの方法](https://jp-seemore.com/web/9218/)
  Rubyを使ってディレクトリ内のファイル一覧を取得する方法を5つ紹介しています。 