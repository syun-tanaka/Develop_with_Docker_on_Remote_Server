# Develop_with_Docker_on_Remote_Server

本リポジトリは リモートサーバー 上の Docker Container で開発を行うためのリポジトリです
devcontainer の設定をしていますので、VS Code と Docker、Git さえあれば各種開発用設定が行われた Python の開発環境が構築され、即時開発が可能です

## 内容

- [Remote-SSH / Remote-Containers](https://code.visualstudio.com/docs/remote/ssh#_open-a-folder-on-a-remote-ssh-host-in-a-container)
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

- VS Code インストール
- リモートとの公開鍵認証を設定

### リモートでの事前準備

- Docker or Rootless-Docker インストール
  - ***Rootless-Docker の場合：`export DOCKER_HOST=...` を `.bashrc` の先頭（ `# If not running interactively, don't do anything` より前）に設定すること***
  - 必要であれば[NVIDIA Docker って今どうなってるの？ (20.09 版)](https://medium.com/nvidiajapan/nvidia-docker-%E3%81%A3%E3%81%A6%E4%BB%8A%E3%81%A9%E3%81%86%E3%81%AA%E3%81%A3%E3%81%A6%E3%82%8B%E3%81%AE-20-09-%E7%89%88-558fae883f44)を参考に NVIDIA Docker 環境を設定
- リモートサーバー がプロキシ環境下に存在する場合、[プロキシーサーバー利用の設定](https://matsuand.github.io/docs.docker.jp.onthefly/network/proxy/#configure-the-docker-client)を参考にプロキシの設定を行う
- 本リポジトリをクローン
  - 必要であれば 本リポジトリの下に開発対象のリポジトリをクローン

### VSCode での事前準備

1. ローカルの VSCode
    1. 拡張機能のインストール
        - [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
        - [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)
    1. Remote-SSH を用いて リモートサーバーへ接続
        1. VSCodeの左端の `Remote Explorer` アイコンをクリック
        1. `SSH TARGETS` の中から目的のサーバーを選択
        1. （VSCode サーバーのインストールが行われるので しばらく待機）

1. リモートサーバー上の VSCode
    1. VSCode で本リポジトリのクローン先を開き、以下を変更する
        - `.devcontainer/Dockerfile`
          - `FROM` [nvcr.io/nvidia/pytorch:22.03-py3](https://docs.nvidia.com/deeplearning/frameworks/pytorch-release-notes/rel_22-03.html#rel_22-03)
            - python3.8, CUDA, PyTorch, TensorBoard, jupyter-lab に加えて [DALI](https://developer.nvidia.com/dali), [RAPIDS](https://rapids.ai/), [TensorRT](https://pytorch.org/TensorRT/) などが含まれている docker image. サイズが大きいので GPU が必要なければ別の軽量な image に変更すべき

          - `For root/non-root user` の一方をコメントアウト
            - non-root user を使用する場合：
            マウントするディレクトリの uid に合わせて `USER_UID` を変更

        - `.devcontainer/devcontainer.json`

          - `name`: 任意の名前
          - `workspaceMount`
            - `source=` clone したリポジトリの 絶対 path
          - `mounts`
            - `source=` リモートの `.ssh`, `.gitconfig` の 絶対 path
            - `target={HOMEDIR}/.ssh (.gitconfig)`, ただし `HOMEDIR` は `Dockerfile` で設定したもの
            - 必要であればデータディレクトリもマウントする
          - `runArgs`
            - [メモリ、CPU、GPU に対する実行時オプション](https://docs.docker.jp/v19.03/config/container/resource_constraints.html)を参考に設定

        - `postCreateCommand.sh`
            - 必要なパッケージをインストール

        - 自作パッケージの開発を行う場合は以下も変更
          - main.py
          - logging.conf
            - `hoge` を使用するモジュール名に合わせる
          - `README.md`
          - `LICENSE`

    1. Remote-Containers を用いて コンテナへ接続
        1. VSCodeの左下の緑色のアイコンをクリック
        1. 「Reopen in Container」をクリック
        1. （docker image の build や container の生成が行われるので しばらく待機）

1. コンテナ上の VSCode
    1. 拡張機能のインストール
        - オススメの拡張機能はインストールされているので その他開発に必要なものをお好みで
    1. Project Manager を用いて プロジェクトを保存
        1. VSCodeの左端の `PROJECT MANAGER` アイコンをクリック
        1. 任意の名前でプロジェクトを save

## 開発手順

### コンテナへの接続方法

1. ローカルで VSCode を起動
1. VSCodeの左端の `PROJECT MANAGER` アイコンをクリック
1. 目的のプロジェクトを選択

### Dockerfile, devcontainer.json を変更した場合

1. VSCodeでコンテナに接続した状態で 左下の緑色のアイコンをクリック
1. 「Rebuild Container」をクリック
1. （docker image の build や container の生成が行われるので しばらく待機）

## 参考

本リポジトリは [python_repository_simple](https://github.com/yamap55/python_repository_simple) (yamap55: Released under the [CC0-1.0 license](https://creativecommons.org/publicdomain/zero/1.0/deed.ja))を参考に作成しています。
