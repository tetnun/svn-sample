# Subversion Template

## 1. 環境

### 1.1. 環境

必要なコマンド

* **docker**, **docker-compose**

### 1.2. ファイル構成

```
.gitignore            ...... gitignore file
docker-compose.yml    ...... docker-compose file
# 下記は、後で作成
passwd                ...... SVN Basic認証用 file
                             htpasswd で作成します
svn                   ...... Docker Host に Mount された
                             Container上のRepogitory
                             docker実行時に作成されます
sample-repo           ...... 「3. Subversion コマンド例」
                             の中で作成されます
```

### 1.3. 設定

**docker-compose.yml** の設定内容

適宜、読み替え・置き換えてください。

| 項目 | 値 |
| --- | --- |
| Subversion Basic認証 | |
| └ User | hoge |
| └ Password | hoge00 |
| └ Host | localhost (\*1)
| └ HTTP Port | 12080 (\*2)|
| └ SVN Port | 13960 (\*3) |
| Repository | sample-repo (\*4) |

\*1. Host は、下記の通りです。

dockerと同じホスト上で svn コマンドを使用する場合

    localhost

dockerと異なるホスト上(例. eample.com)で svn コマンドを使用する場合
**localhost** 部分を読み替えてください

    eample.com
    


\*2. HTTP でのRepository Root URL は、下記の通りです。

    http://localhost:12080/svn
    
\*3. SVN でのRepository Root URL は、下記の通りです。

    svn://localhost:13960
    
\*4. HTTP Repository は、下記の通りです。

    http://localhost:12080/svn/sample-repo
    
## 2. 構築手順

### 2.1. docker起動 (初回)

空ファイル passwd ファイル作成して
docker起動します

```
$ touch passwd
$ docker-compose up -d
...(略)...
Starting my-svn ... done
```

起動確認

```
$ docker-compose ps
 Name    Command   State                            Ports                         
----------------------------------------------------------------------------------
my-svn   /init     Up      0.0.0.0:13960->3960/tcp, 443/tcp, 0.0.0.0:12080->80/tcp
```

### 2.2. SVN Basic認証用 file作成

```
$ docker-compose exec my-svn htpasswd -bc /etc/subversion/passwd hoge hoge00
```

ファイル内容確認 (1行追加されていることを確認する)

```
$ docker-compose exec my-svn cat /etc/subversion/passwd
hoge:$apr1$f5Nsy1MD$Fj.WOTTdv4g85asQJ4QIk.
```

ブラウザーで http アクセスできることを確認する
設定したBasic認証情報でアクセスすること

```
http://localhost:12080/svn
```

### 2.3. レポジトリ作成

レポジトリ **sample-repo** を作成し
所有者を **apache** のユーザ・グループにする


```
$ docker-compose exec my-svn svnadmin create /home/svn/sample-repo
$ docker-compose exec my-svn chown -R apache:apache /home/svn/sample-repo
```

## 3. Subversion コマンド例

### 3.1. チェックアウト - co

チェックアウトします。

【初回の例】

```
$ svn co --username hoge --password hoge00 http://localhost:12080/svn/sample-repo
Checked out revision 0.
$ ls -1a sample-repo/
.
..
.svn
```

### 3.2. ブランチ作成 - mkdir

レポジトリ **sample-repo** に移動して作業

ここでは、 ブランチ **trunk**, **branches**, **tags** を作成します

```
$ cd sample-repo
$ svn mkdir trunk branches tags
A         trunk
A         branches
A         tags
```

### 3.3. コミット - commit

作成した ブランチをコミットします

```
$ svn commit -m "First commit" --username hoge --password hoge00 --no-auth-cache
Adding         branches
Adding         tags
Adding         trunk
Committing transaction...
Committed revision 1.
```