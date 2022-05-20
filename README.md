# Develop_with_Docker_on_Remote_Server

本リポジトリは リモートサーバー 上の Docker Container で開発を行うためのテンプレートリポジトリです
devcontainer の設定をしていますので、VS Code と Docker、Git さえあれば各種開発用設定が行われた Python の開発環境が構築され、即時開発が可能です
GitHub のリポジトリページの「Use this template」を押下して使用してください

## 内容

- [devcontainer](https://code.visualstudio.com/docs/remote/containers)
- linter, formatter
  - [flake8](https://flake8.pycqa.org/en/latest/)
  - [black](https://black.readthedocs.io/en/stable/)
  - [isort](https://pycqa.github.io/isort/)
  - [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance), [pyright](https://github.com/microsoft/pyright)
  - [hadolint](https://github.com/hadolint/hadolint)
- [pytest](https://docs.pytest.org/en/stable/)
- [logging](https://docs.python.org/ja/3/howto/logging.html)

## 設定方法

### ローカルでの事前準備

- Docker CLI インストール
- VS Code インストール
- VSCode の拡張機能のインストール
  - Remote-Containers
    - https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers
  - Sync-Rsync
    - https://marketplace.visualstudio.com/items?itemName=vscode-ext.sync-rsync
- リモートとの公開鍵認証を設定
- リモート PC がプロキシ環境下に存在する場合、[プロキシーサーバー利用の設定](https://matsuand.github.io/docs.docker.jp.onthefly/network/proxy/#configure-the-docker-client)を参考に**ローカルマシンで**プロキシの設定を行う

### リモートでの事前準備

- Docker or Rootless-Docker インストール
  - 必要であれば[NVIDIA Docker って今どうなってるの？ (20.09 版)](https://medium.com/nvidiajapan/nvidia-docker-%E3%81%A3%E3%81%A6%E4%BB%8A%E3%81%A9%E3%81%86%E3%81%AA%E3%81%A3%E3%81%A6%E3%82%8B%E3%81%AE-20-09-%E7%89%88-558fae883f44)を参考に NVIDIA Docker 環境を設定
- 本リポジトリの clone
- 以下を変更

  - `.devcontainer/Dockerfile`

    - `For root/non-root user`の一方をコメントアウト
    - (Normal) Docker の場合:
      マウントするディレクトリの uid に合わせて`USER_UID`を変更

  - `.devcontainer/devcontainer.json`

    - `name`: 任意の名前
    - `workspaceMount`
      - `source=` リモートに clone したリポジトリの path
    - `mounts`
      - `source=` リモートの`.ssh`, `.gitconfig` の path
      - `target={HOMEDIR}/.ssh (.gitconfig)`, ただし`HOMEDIR`は`Dockerfile`で設定したもの
      - 必要であればデータディレクトリもマウントする
    - `runArgs`
      - [メモリ、CPU、GPU に対する実行時オプション](https://docs.docker.jp/v19.03/config/container/resource_constraints.html)を参考に設定

  - `.vscode/settings.json`
    - `docker.host`:
      - (Normal) Docker の場合:
        リモートのユーザー名とホスト名を用いて`ssh://{User}@{HostName}`
      - Rootless Docker の場合:
        任意のポート番号を用いて`tcp://localhost:{port}`
    - `sync-rsync.sites`
      - `name`: 任意の名前
      - `remotePath`: `{User}@{HostName}:/path/to/local_vscode_setting`

- 必要であれば以下も変更する
  - main.py
  - logging.conf
    - `hoge` を使用するモジュール名に合わせる
  - `README.md`
  - `LICENSE`

## 開発手順

1. Rootless Docker の場合

   1. リモートで`echo $DOCKER_HOST`を実行し、`unix://`を除いた`/path/to/docker.sock`をメモ
   1. ローカルで以下を実行
      ```bash
      $ ssh -fNL localhost:{port}:{/path/to/docker.sock} {User}@{HostName}
      ```
      ただし`port`は`docker.host`に設定したポート番号、`/path/to/docker.sock`は先程メモした path

1. (初回のみ) リモートの`local_vscode_setting/`をローカルにコピー
   - 必要であればディレクトリ名を変更してもよい
1. ローカルの`local_vscode_setting/`内で VSCode を起動
1. 左下の緑色のアイコン -> 「Remote-Containersa: Reopen in Container」をクリック
1. しばらく待つ
   - 初回の場合、コンテナー image の取得や作成が行われる
1. 起動したらコンテナでの開発可能

### `local_vscode_setting/`内のファイルを変更した場合 (Container の Rebuild)

1. 左下の緑色のアイコン -> 「Remote-Containersa: Reopen Locally」をクリック
1. (ローカルで) cmd + shift + P -> 「Sync-Rsync: Sync Remote to Local」を検索してクリック
1. (ローカルで) 左下の緑色のアイコン -> 「Remote-Containersa: Reopen in Container」をクリック
1. 左下の緑色のアイコン -> 「Remote-Containersa: Rebuild Container」をクリック

## ユニットテスト実行

```
pytest
```

## 参考

本リポジトリは [python_repository_simple](https://github.com/yamap55/python_repository_simple) (yamap55: Released under the [CC0-1.0 license](https://creativecommons.org/publicdomain/zero/1.0/deed.ja))を参考に作成しています。
