# Concourse Tutorial
# Concourseチュートリアル

Learn to use https://concourse.ci with this linear sequence of tutorials. Learn each concept that builds on the previous concept.

https://concourse.ci の使い方を一連のチュートリアルを通じて学びます。
前の章で学んだコンセプトを使って個別のコンセプトについて学びます。

## Sections
## セクション

* [01 - Hello World](#01---hello-world)
* [02 - Task inputs](#02---task-inputs)
* [03 - Task scripts](#03---task-scripts)
* [04 - Basic pipeline](#04---basic-pipeline)
* [05 - Tasks extracted into Resources](#05---tasks-extracted-into-resources)
* [06 - View job output in terminal](#06---view-job-output-in-terminal)
* [07 - Trigger jobs with fly](#07---trigger-jobs-with-fly)
* [08 - Triggering jobs with resources](#08---triggering-jobs-with-resources)
* [09 - Destroying pipelines](#09---destroying-pipelines)
* [10 - Using resource inputs in job tasks](#10---using-resource-inputs-in-job-tasks)
* [11 - Passing task outputs to another task](#11---passing-task-outputs-to-another-task)
* [12 - Publishing outputs](#12---publishing-outputs)
* [13 - Actual pipeline - passing resources between jobs](#13---actual-pipeline---passing-resources-between-jobs)
* [14 - Parameterized pipelines](#14---parameterized-pipelines)

The following sections are found in subfolders of this repository and the tutorial continues in their README:
以下のセクションはこのリポジトリのサブフォルダに見つかります。
READMEのなかでチュートリアルは続きます。

* [15 - Deploy application to Cloud](15_deploy_cloudfoundry_app/README.md)
* [16 - Run tests, then deploy application](16_run_tests_before_deploy/README.md)
* [20 - Versions and build numbers](20_versions_and_buildnumbers/README.md)
* [21 - Versioned artifacts](21_versioned_artifacts/README.md)
* [35 - Push Docker Image](35_push_docker_image/README.md)
* [36 - Pull Docker Image](36_pull_docker_image/README.md)

* [40 - Github release input](40_github_release_input/README.md)
* [42 - Run Concourse on BOSH lite AWS](42_run_concourse_on_bosh_lite_aws/README.md)
* [44 - BOSH IO](44_bosh_io/README.md)
* [45 - BOSH deploy](45_bosh_deploy/README.md)
* [46 - BOSH manifest spiff merge](46_bosh_manifest_spiff_merge/README.md)
* [47 - Spiff merge Redis to BOSH lite](47_spiff_merge_redis_to_bosh_lite/README.md)
* [51 - Dummy resource Docker image](51_dummy_resource_docker_image/README.md)
* [52 - Dummy resource](52_dummy_resource/README.md)
* [70 - Pivnet updates](70_pivnet_updates/README.md)

## Thanks

Thanks to Alex Suraci for inventing Concourse CI, and to Pivotal for sponsoring him and a team of developers to work on it for over a year (2014 onwards).

At Stark & Wayne we started this tutorial as we were learning Concourse in early 2015, and we've been using Concourse in production since mid-2015 internally and at nearly all client projects.

Thanks to everyone who has worked through this tutorial and found it useful. I love learning that you're enjoying the tutorial and enjoying Concourse.

Thanks for all the pull requests to help fix regressions with some Concourse versions that came out with "backwards incompatible change".

## Getting Started

### Mac & Linux

1. [Virtualbox](https://www.virtualbox.org/wiki/Downloads) をインストール
2. [BOSH CLI v2](https://bosh.io/docs/cli-v2.html#install) をインストール
3. VirtualboxとBOSHを使って単一のVMで構成されたconcourseを構築します。

Download the `concourse-lite` deployment manifest and then have `bosh` create a
Single VM server running concourse on Virtualbox.

`concourse-lite` デプロイメントマニフェストをダウンロードして `bosh`を用いて
Virtualbox上の単一のVMサーバを使ったconcourseを実行させます。

```
wget https://github.com/concourse/concourse/releases/download/v3.5.0/concourse-lite.yml
bosh create-env concourse-lite.yml
```

### Windows

1. Use the Vagrant box as a pre-compiled build of a Single VM instance of Concourse.
1. 事前にコンパイルされたconcourseを含む単一のVMをvagrant boxを用いて準備します。

```
git clone https://github.com/starkandwayne/concourse-tutorial.git
cd concourse-tutorial
vagrant box add concourse/lite --box-version $(cat VERSION)
vagrant up
```

### Test Setup

## テストをセットアップします。

Open http://192.168.100.4:8080/ in your browser:

[![initial](no_pipelines.png)](http://192.168.100.4:8080/)

Once the page loads in your browser, click to download the `fly` CLI appropriate for your operating system:

![cli](fly_cli.png)

Once downloaded, copy the `fly` binary into your path (`$PATH`), such as `/usr/local/bin` or `~/bin`. Don't forget to also make it executable. For example,

```
sudo mkdir -p /usr/local/bin
sudo mv ~/Downloads/fly /usr/local/bin
sudo chmod 0755 /usr/local/bin/fly
```

For Windows users, use [this article](https://stackoverflow.com/questions/23400030/windows-7-add-path)
to see where to add `fly` in to the `PATH`.

Target Concourse
----------------

In the spirit of declaring absolutely everything you do to get absolutely the same result every time, the `fly` CLI requires that you specify the target API for every `fly` request.

毎回同じ結果を得るために、あなたが実行すること全てを絶対に宣言するという精神に基づき、
the `fly` CLIは `fly`をもちいたリクエストを行うたびにターゲットAPIを特定する必要があります。

First, alias it with a name `tutorial` (this name is used by all the tutorial task scripts):

まず、ターゲットAPI?に対して`tutorial`という別名をつけます。

```
訳注)
つまりアクセス先のホストはここで名前をつける。
さらに、syncを行うことで、アクセス先のホスト上で動作するconcouseのバージョンと
flyのバージョンを一致させます。
```

```
fly --target tutorial login --concourse-url http://192.168.100.4:8080
fly --target tutorial sync
```

ここまでの作業でローカルファイルにConcourse APIのターゲットが保存されます。

```
cat ~/.flyrc
```

.flyrcの中身は、アクセス先のAPIや認証情報などが含まれたシンプルなYAMLファイルです。

```yaml
targets:
  tutorial:
    api: http://192.168.100.4:8080
    token:
      type: ""
      value: ""
```

`fly`コマンドを利用する際、このConcourse APIを `fly --target tutorial`とすることで指定します。
When we use the `fly` command we will target this Concourse API using `fly --target tutorial`.

> @alexsuraci: I promise you'll end up liking it more than having an implicit target state :) Makes reusing commands from shell history much less dangerous (rogue fly configure can be bad)


ここから先は個別のチュートリアルとなります。

Tutorials
---------

### 01 - Hello World

The central concept of Concourse is to run tasks. You can run them directly from the command line as below, or from within pipeline jobs (as per every other section of the tutorial).

Concourseの中心的なコンセプトはタスクを実行することにあります。
以下のようにコマンドラインから直接的にタスクを実行することができます。
またはパイプラインジョブの内部から実行します。

```
cd 01_task_hello_world
fly --target tutorial execute --config task_hello_world.yml
```

The output starts with
出力の冒頭部分は以下のようになります。

```
executing build 1
initializing
```

Concourseでは全てのタスクはコンテナーの内部で実行されます。
`task_hello_world.yml`の設定は`linux`プラットフォーム上で、`busybos`コンテナーイメージを用いてタスクが実行されることを表しています。
このコンテナ内部では`echo hello world`のコマンドが実行されます。

```yaml
---
platform: linux

image_resource:
  type: docker-image
  source: {repository: busybox}

run:
  path: echo
  args: [hello world]
```

上記の部分が実行された際、`busybox`のDockerイメージをダウンロードするメッセージが表示されます。
これは一度しか実行する必要はありませんが、毎回より新しい`busybox`イメージが存在しないかを確認します。
最終的に、処理は継続されて`echo hello world`コマンドが成功裏に呼び出されます。

```
running echo hello world
hello world
succeeded
```

試しに`image_resource:`と`run:`を変更して、別のタスクを実行しましょう。

```yaml
---
platform: linux

image_resource:
  type: docker-image
  source: {repository: ubuntu, tag: "14.04"}

run:
  path: uname
  args: [-a]
```

忙しい人のための利便性のために、先ほど説明した変更内容を反映した
`task_ubuntu_uname.yml`ファイルを提供します。

```
fly --target tutorial execute --config task_ubuntu_uname.yml
```

出力は以下のようになります。

```
executing build 2
initializing
running uname -a
Linux 96f1e0d6-ffdb-40e1-5e8b-5ae9a3b36a58 4.2.0-42-generic #49~14.04.1-Ubuntu SMP Wed Jun 29 20:22:11 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
succeeded
```

ベース`image`(または[タスクを設定する際](http://concourse.ci/running-tasks.html)の`image_resource`)を選択できるのは、実行するのに必要な前提条件を持つことを許容しているからである。

タスクの中で毎回前提条件として必要になるものを導入する代わりに、`image`を用いて前提条件を
充足させておくことでタスクをより高速に実行することを可能にします。

[Go to Top](#concourse-tutorial)

### 02 - Task inputs

前のセクションでは、タスクコンテナに対しての入力は利用する`image`のみでした。
Dockerイメージのようなベースイメージは、比較的静的かつ大きく、作成するのに時間がかかります。
そこで、Concourseでは`inputs`をファイルやフォルダを処理のためにパスすることをサポートしています。


以下のように入力の存在しないタスクを考えてみます。

```
---
platform: linux

image_resource:
  type: docker-image
  source: {repository: busybox}

run:
  path: ls
  args: ['-al']

```

これを実行します。

```
cd ../02_task_inputs
fly --target tutorial execute --config no_inputs.yml
```

タスクでは `ls -al` が実行され、コンテナ内部の空っぽの作業フォルダのなかが表示されます。

```
running ls -al
total 8
drwxr-xr-x    2 root     root          4096 Feb 27 07:23 .
drwxr-xr-x    3 root     root          4096 Feb 27 07:23 ..
```

例としてのタスク`inputs_required.yml`では、一つの入力を追加してみました。


```
---
platform: linux

image_resource:
  type: docker-image
  source: {repository: busybox}

inputs:
- name: some-important-input

run:
  path: ls
  args: ['-alR']
```

上記ファイルのこの部分でinputを要求しています。

```yaml
inputs:
- name: some-important-input
```

When we try to execute the task:

タスクを実行しようとすると、

```
fly --target tutorial execute --config inputs_required.yml
```

以下のように失敗します。

```
error: missing required input `some-important-input`
```

一般的に`fly execute`を実行しようとした場合、ローカルフォルダ(`.`)をパスしたいと考えると思います。
以下のように`-i name=path`オプションで、要求される`inputs`にたいしての設定を行えます。


```
fly --target tutorial execute --config inputs_required.yml --input some-important-input=.
```


ここで、`fly execute`コマンドでは`.`ディレクトリをコンテナへの入力としてアップロードします。
そして、`soma-important-input`のパスで利用可能になります。

```
running ls -alR
.:
total 8
drwxr-xr-x    3 root     root          4096 Feb 27 07:27 .
drwxr-xr-x    3 root     root          4096 Feb 27 07:27 ..
drwxr-xr-x    1 501      20              64 Feb 27 07:27 some-important-input

./some-important-input:
total 12
drwxr-xr-x    1 501      20              64 Feb 27 07:27 .
drwxr-xr-x    3 root     root          4096 Feb 27 07:27 ..
-rw-r--r--    1 501      20             112 Feb 27 07:30 input_parent_dir.yml
-rw-r--r--    1 501      20             118 Feb 27 07:27 inputs_required.yml
-rw-r--r--    1 501      20              79 Feb 27 07:18 no_inputs.yml
```


別のディレクトリをパスしたい場合は、相対パスを指定します。

```
fly --target tutorial execute --config inputs_required.yml --input some-important-input=../01_task_hello_world
```

`fly execute --input ` オプションは外すことができます。
これは要求されている入力とカレントディレクトリの名前が一致する場合です。
以下のコマンドは先ほど提示したコマンドと同じ結果を返します。

```
fly --target tutorial execute --config input_parent_dir.yml
```

[Go to Top](#concourse-tutorial)

### 03 - Task scripts

The `inputs` feature of a task allows us to pass in two types of inputs:

タスクの`inputs`機能によって、二種類の入力をコンテナにパスできることがわかりました。

1. 処理、テスト、コンパイルに必要なものや・依存関係のあるもの
1. 複雑な振る舞いを実現するために実行されるタスクスクリプト

* requirements/dependencies to be processed/tested/compiled
* task scripts to be executed to perform complex behavior

A common pattern is for Concourse tasks to `run:` complex shell scripts rather than directly invoking commands as we did above (we ran `uname` command with arguments `-a`).

`run:`をつかって複雑な処理を実行させるほうが、(わたしが`uname`コマンドを`-a`オプションをつけて実行して見せたように)直接、複雑なコマンドを実行するよりは典型的なConcouseタスクになります。


Let's refactor `01_task_hello_world/task_ubuntu_uname.yml` into a new task `03_task_scripts/task_show_uname.yml` with a separated task script `03_task_scripts/task_show_uname.sh`

`01_task_hello_world/task_ubuntu_uname.yml`をリファクタして、
`03_task_scripts/task_show_uname.yml`を作りましょう。
このスクリプトにはセットとして、`03_task_scripts/task_show_uname.sh`が付いてきます。

その上で以下のように`task_show_uname.yml`を実行します。

```
cd ../03_task_scripts
fly --target tutorial execute --config task_show_uname.yml
```

The former specifies the latter as its task script:

ymlファイルは以下のようにして.shファイルをymlファイル内に含まれるタスクスクリプトとして実行します。

```yaml
run:
  path: ./03_task_scripts/task_show_uname.sh
```

_Where does the `./03_task_scripts/task_show_uname.sh` file come from?_

`./03_task_scripts/task_show_uname.sh`ファイルは一体どこからきたのでしょうか。

From section 2 we learned that we could pass `inputs` into the task. The task configuration `03_task_scripts/task_show_uname.yml` specifies one input:

セクション2で我々は`inputs`をタスクにパスできることを学びました。
`03_task_scripts/task_show_unam.yml`では一つの入力を指定しています。

```
inputs:
- name: 03_task_scripts
```

ここで03_task_scriputsをinputsとして指定しています。
これによって、`03_task_scripts`はカレントディレクトリと一致するので、
`fly execute --inpute 03_task_scripts=.`のような形で、inputsを指定する必要はありません。

Since input `03_task_scripts` matches the current directory `03_task_scripts` we did not need to specify `fly execute --input 03_task_scripts=.`.


現在のディレクトリがConcourseのタスクコンテナにアップロードされ、`03_task_scripts`ディレクトリに
配置されます。

The current directory was uploaded to the Concourse task container and placed inside the `03_task_scripts` directory.

したがって、`task_show_uname.sh`ファイルをConcourseタスクコンテナの内部で`03_task_scripts/task_show_uname.sh`として指定の上実行することができるようになります。

Therefore its file `task_show_uname.sh` is available within the Concourse task container at `03_task_scripts/task_show_uname.sh`.

追加でさらに必要にるのは、`task_show_uname.sh`です。
これは、実行可能なスクリプトとなっています。

The only further requirement is that `task_show_uname.sh` is an executable script.

[Go to Top](#concourse-tutorial)

### 04 - Basic pipeline


`fly execute`で実行されるタスクは全体の1%です。残りの99%のタスクはパイプラインの中で実行されます。

1% of tasks that Concourse runs are via `fly execute`. 99% of tasks that Concourse runs are within "pipelines".

```
cd ../04_basic_pipeline
fly --target tutorial set-pipeline --config pipeline.yml --pipeline helloworld
```

上記コマンドではConcourseのパイプライン(またはその変更内容)が表示され、
確認を促されます。

It will display the concourse pipeline (or any changes) and request confirmation:

```yaml
jobs:
  job job-hello-world has been added:
    name: job-hello-world
    public: true
    plan:
    - task: hello-world
      config:
        platform: linux
        image_resource:
          type: docker-image
          source: {repository: busybox}
        run:
          path: echo
          args:
          - hello world
          dir: ""
```

設定ファイルに変更があると、
`fly set-pipeline`(またはそのエイリアスである`fly sp`)を実行するたびに
設定の変更を適用することを促されます。

You will be prompted to apply any configuration changes each time you run `fly set-pipeline` (or its alias `fly sp`)

```
apply configuration? [yN]:
```


`y`を押しましょう。
Press `y`.

すると以下のように表示されます。

You should see:

```
pipeline created!
you can view your pipeline here: http://192.168.100.4:8080/pipelines/helloworld

the pipeline is currently paused. to unpause, either:
  - run the unpause-pipeline command
  - click play next to the pipeline in the web ui
```

As suggested, un-pause a pipeline from the `fly` CLI:

メッセージ中に記載されている通り、`fly` CLIを使ってパイプラインの実行を再開します。

```
fly --target tutorial unpause-pipeline --pipeline helloworld
```


次は、メッセージ中で促された通り、web UIにアクセスします。

Next, as suggested, visit the web UI http://192.168.100.4:8080/pipelines/helloworld.

最初のパイプラインはなんということはないものです。
`job-hello-world`という単一のジョブは入力ジョブも出力ジョブも持っていません。
また、ジョブ単独としても、なんの入力も持っていません。

これは最も基本的なパイプラインです。
まだ実行されたことがないので、グレーカラーのままです。

Your first pipeline is unimpressive - a single job `job-hello-world` with no inputs from the left and no outputs to its right, no jobs feeding into it, nor jobs feeding from it. It is the most basic pipeline. The job is gray colour because it has never been run before.

`job-hello-world`をクリックして大きな`+`をクリックするとジョブが実行されます。

Click on `job-hello-world` and then click on the large `+` in the top right corner. Your job will run.

![job](http://cl.ly/image/3i2e0k0v3O2l/02-job-hello-world.gif)

最上部の"Home"アイコンをクリックすると、パイプラインの状態が表示されます。
`job-hello-world`ジョブは現在グリーンです。
これは、ジョブの実行が成功したことを表します。

Clicking the top-left "Home" icon will show the status of our pipeline. The job `job-hello-world` is now green. This means that the last time the job ran it completed successfully.

[Go to Top](#concourse-tutorial)

### 05 - Tasks extracted into Resources

`pipeline.yml`内でジョブのタスクを定義することでパイプラインを繰り返し実行することが容易になりました。
`pipeline.yml`を編集して、`fly set-pipeline`を実行することでパイプライン全体を即時アップデートすることができます。

It is very fast to iterate on a job's tasks by configuring them in the `pipeline.yml` as above. You edit the `pipeline.yml`, run `fly set-pipeline`, and the entire pipeline is updated atomically.

しかし、セクション3で述べた通り、タスクが複雑化すると、`run:`コマンドがタスクスクリプトに切り出されます。
さらに、タスクそのものも`yml`タスクファイルに出力されるようになります。

But, as per section 3, if a task becomes complex then its `run:` command can be extracted into a task script, and the task itself can be extracted into a `yml` task file.

セクション3では、`fly execute`コマンドを用いてローカルのコンピュータからタスクスクリプトとタスクファイルをアップロードしていました。

In section 3 we uploaded the task file and task script from our local computer with the `fly execute` command.

セクション3と異なり、パイプラインではConcourse外部のどこかにタスクファイルとスクリプトを保管しておく
必要があります。

Unlike section 3, with a pipeline we now need to store the task file and task script somewhere outside of Concourse.

データを保管/回収するためのサービスをConcourseは提供しません。
gitリポジトリもなければblogストアもありません。ビルドナンバーもないです。
全ての入力と出力は外部から提供される必要があります。
Concourseではそれらを"Resources"と呼んでいます。
例えばリソースとしては、`git`、`s3`、`semver`といったものがgitリポジトリ、blobストア、ビルドナンバー
に対応するものとして存在します。

Concourse offers no services for storing/retrieving your data. No git repositories. No blobstores. No build numbers. Every input and output must be provided externally. Concourse calls them "Resources". Example resources are `git`, `s3` and `semver` respectively.

後述の"利用可能な concourse resourceのリスト"のセクションには、利用可能な built-in resourceと
コミュニティresourceの見つけ方が記載されています。
例えばslackにメッセージを送ったり、バージョン番号を0.5.6から1.0.0に付け替えるためのもの、
Pivotal Trackerにチケットを作成したりといったものも含みます。


See the section "Available concourse resources" below for the list of available built-in resources and how to find community resources. Send messages to Slack. Bump a version number from 0.5.6 to 1.0.0. Create a ticket on Pivotal Tracker. It is all possible with Concourse resources.

`git`リソースはタスクファイルとタスクスクリプトを保管するための最も一般的なリソースです。

The most common resource to store our task files and task scripts is the `git` resource.

このチュートリアルのソースリポジトリはGitリポジトリです。
そして、大量のタスクファイルとタスクスクリプトを含んでいます。
例えば、最初に実行した`01_task_hello_world/task_hello_world.yml`などです。

This tutorial's source repository is a Git repo, and it contains many task files (and their task scripts). For example, the original `01_task_hello_world/task_hello_world.yml`.

次のパイプラインでは、タスクファイルを読み込んでそれを実行するような形になっています。
ここでは、先ほどの`helloworld`パイプラインをアップデートするようになっています。

The following pipeline will load this task file and run it. We will update the previous `helloworld` pipeline:

```
cd ../05_pipeline_task_hello_world
fly set-pipeline --target tutorial --config pipeline.yml --pipeline helloworld
```

このコマンドを実行すると2つのパイプラインの差分が表示されます。
`y`をタイプして成功したら設定がアップデートされた胸のメッセージが表示されます。

The output will show the delta between the two pipelines and request confirmation. Type `y`. If successful, it will show:

```
apply configuration? [yN]: y
configuration updated
```

The [`helloworld` pipeline](http://192.168.100.4:8080/pipelines/helloworld) now shows an input resource `resource-tutorial` feeding into the job `job-hello-world`.

[`helloworld` pipeline](http://192.168.100.4:8080/pipelines/helloworld) では、
この時点で、`resource-turorial`が`job-hello-world`ジョブへの入力リソースとして流入していることが
表示されます。

![pipeline-task-hello-world](http://cl.ly/image/271z3T322l25/03-resource-job.gif)

This tutorial verbosely prefixes `resource-` to resource names, and `job-` to job names, to help you identify one versus the other whilst learning. Eventually you will know one from the other and can remove the extraneous text.

このチュートリアルでは、`resource-`という接頭語をリソース名につけて冗長になっています。
そして`job-`という接頭語をジョブにはつけています。
これによって個々の要素を区別して学ぶことを容易にしています。
最終的には、あなたは個々の要素を区別できるようになると思うので、その時に余分なテキストを除去しましょう。

After manually triggering the job via the UI, the output will look like:

UIからジョブを手動で実行した後に、出力はこのようになります。


![job-task-from-task](http://cl.ly/image/0Q3m223v2l3M/job-task-from-task.png)

The in-progress or newly-completed `job-hello-world` job UI has three sections:

* Preparation for running the job - collecting inputs and dependencies
* `resource-tutorial` resource is fetched
* `hello-world` task is executed

The latter two are "steps" in the job's [build plan](http://concourse.ci/build-plans.html). A build plan is a sequence of steps to execute. These steps may fetch down or update Resources, or execute Tasks.

The first build plan step fetches down (note the down arrow to the left) a `git` repository for these training materials and tutorials. The pipeline named this resource `resource-tutorial`.

The `pipeline.yml` documents this single resource:

```yaml
resources:
- name: resource-tutorial
  type: git
  source:
    uri: https://github.com/starkandwayne/concourse-tutorial.git
```

The resource name `resource-tutorial` is then used in the build plan for the job:

```yaml
jobs:
- name: job-hello-world
  public: true
  plan:
  - get: resource-tutorial
```

Any fetched resource can now be an input to any task in the job build plan. As discussed in section 3 & section 4, task inputs can be used as task scripts.

The second step runs a user-defined task. The pipeline named the task `hello-world`. The task itself is not described in the pipeline. Instead it is described in a file `01_task_hello_world/task_hello_world.yml` from the `resource-tutorial` input.

The completed job looks like:

```yaml
jobs:
- name: job-hello-world
  public: true
  plan:
  - get: resource-tutorial
  - task: hello-world
    file: resource-tutorial/01_task_hello_world/task_hello_world.yml
```

The task `{task: hello-world, file: resource-tutorial/...}` has access to all fetched resources (and later, to the outputs from tasks).

The name of resources, and later the name of task outputs, determines the name used to access them by other tasks (and later, by updated resources).

So, `hello-world` can access anything from `resource-tutorial` (this tutorial's `git` repository) under the `resource-tutorial/` path. Since the relative path of `task_hello_world.yml` task file inside this repo is `01_task_hello_world/task_hello_world.yml`, the `task: hello-world` references it by joining the two: `file: resource-tutorial/01_task_hello_world/task_hello_world.yml`


There are benefits and downsides to abstracting tasks into YAML files outside of the pipeline.

One benefit is that the behavior of the task can be kept in sync with the primary input resource (for example, a software project with tasks for running tests, building binaries, etc).

Another benefit of extracting inline tasks into task files is that `pipeline.yml` files can get long and it can be hard to read and comprehend all the YAML. When extracting tasks into separate files we can give them long names so that readers can understand the purpose and expectation of the tasks at a glance.

One downside is that the `pipeline.yml` no longer explains exactly what commands will be invoked. Comprehension of pipeline behavior is potentially reduced.

Also, when we extract inline tasks into files, `fly set-pipeline` is no longer the only step to updating a pipeline.

From now onwards, any change to your pipeline might require you to do one or both:

* `fly set-pipeline` to update Concourse on a change to the job build plan and/or input/output resources
* `git commit` and `git push` your primary resource that contains the task files and task scripts

If the new behavior you intended is not showing up in the pipeline then you may have skipped one of the two steps above.

[Go to Top](#concourse-tutorial)

### 06 - View job output in terminal

The `job-hello-world` had terminal output from its resource fetch of a git repo and of the `hello-world` task running.

In addition to the Concourse web ui you can also view this output from the terminal with `fly`:

```
fly --target tutorial watch --job helloworld/job-hello-world
```

The output will be similar to:

```
using version of resource found in cache
initializing
running echo hello world
hello world
succeeded
```

The `--build NUM` option allows you to see the output of a specific build number, rather than the latest build output.


You can see the results of recent builds across all pipelines with `fly builds`:

```
fly --target tutorial builds
```

The output will look like:

```
id  pipeline/job                build  status     start                     end                       duration
9   helloworld/job-hello-world  2      succeeded  2017-01-09@10:42:52-0800  2017-01-09@10:42:56-0800  4s
8   helloworld/job-hello-world  1      succeeded  2017-01-09@10:42:26-0800  2017-01-09@10:42:28-0800  2s
7   one-off                     n/a    succeeded  2017-01-09@10:41:25-0800  2017-01-09@10:41:33-0800  8s
6   one-off                     n/a    succeeded  2017-01-09@10:41:14-0800  2017-01-09@10:41:17-0800  3s
5   one-off                     n/a    succeeded  2017-01-09@10:41:08-0800  2017-01-09@10:41:11-0800  3s
4   one-off                     n/a    succeeded  2017-01-09@10:40:57-0800  2017-01-09@10:41:00-0800  3s
3   one-off                     n/a    succeeded  2017-01-09@10:40:40-0800  2017-01-09@10:40:43-0800  3s
2   one-off                     n/a    succeeded  2017-01-09@10:40:00-0800  2017-01-09@10:40:28-0800  28s
1   one-off                     n/a    succeeded  2017-01-09@10:39:41-0800  2017-01-09@10:39:51-0800  10s
```

[Go to Top](#concourse-tutorial)

### 07 - Trigger jobs with `fly`

There are three ways for a job to be triggered:

* Clicking the `+` button on the web UI of a job (as we did in previous sections)
* Input resource triggering a job (see section 8 below)
* `fly --target target trigger-job --job pipeline/jobname` command


```
fly --target tutorial trigger-job --job helloworld/job-hello-world
```

Whilst the job is running, and after it has completed, you can then watch the output in your terminal using `fly watch`:

```
fly --target tutorial watch --job helloworld/job-hello-world
```

[Go to Top](#concourse-tutorial)

### 08 - Triggering jobs with resources

The primary way that Concourse jobs will be triggered to run will be by resources changing. A `git` repo has a new commit? Run a job to test it. A GitHub project cuts a new release? Run a job to pull down its attached files and do something with them.

Triggering resources are defined the same as non-triggering resources, such as the `resource-tutorial` defined earlier.

The difference is in the job build plan where triggering is desired.

By default, including `get: my-resource` in a build plan does not trigger its job.

To mark a fetched resource as a trigger add `trigger: true`

```yaml
jobs:
- name: job-demo
  plan:
  - get: resource-tutorial
    trigger: true
```

In the above example the `job-demo` job would trigger anytime the remote `resource-tutorial` had a new version. For a `git` resource this would be new git commits.

The `time` resource has express purpose of triggering jobs.

If you want a job to trigger every few minutes then there is the [`time` resource](https://github.com/concourse/time-resource#readme).

```yaml
resources:
- name: my-timer
  type: time
  source:
    interval: 2m
```

Now upgrade the `helloworld` pipeline with the `time` trigger.

```
cd ../08_triggers
fly set-pipeline --target tutorial --config pipeline.yml --pipeline helloworld
```

This adds a new resource named `my-timer` which triggers `job-hello-world` approximately every 2 minutes.

Visit the pipeline dashboard http://192.168.100.4:8080/pipelines/helloworld and wait a few minutes and eventually the job will start running automatically.

![my-timer](http://cl.ly/3M3A2n1e2i2s/download/Image%202016-02-28%20at%2010.39.37%20am.png)

The dashboard UI makes non-triggering resources distinct with a hyphenated line connecting them into the job. Triggering resources have a full line.

Why does `time` resource configured with `interval: 2m` trigger "approximately" every 2 minutes?

> "resources are checked every minute, but there's a shorter (10sec) interval for determining when a build should run; time resource is to just ensure a build runs on some rough periodicity; we use it to e.g. continuously run integration/acceptance tests to weed out flakiness" - alex

The net result is that a timer of `2m` will trigger every 2 to 3 minutes.

[Go to Top](#concourse-tutorial)

### 09 - Destroying pipelines

The current `helloworld` pipeline will now keep triggering every 2-3 minutes for ever. If you want to destroy a pipeline - and lose all its build history - then may the power be granted to you.

You can delete the `helloworld` pipeline:

```
fly destroy-pipeline --target tutorial --pipeline helloworld
```

[Go to Top](#concourse-tutorial)

### 10 - Using resource inputs in job tasks

Note, the topic of running unit tests in your pipeline will be covered in more detail in future sections.

Consider a simple application that has unit tests. In order to run those tests inside a pipeline we need:

* a task `image` that contains any dependencies
* an input `resource` containing the task script that knows how to run the tests
* an input `resource` containing the application source code

For the example Go application [simple-go-web-app](https://github.com/cloudfoundry-community/simple-go-web-app), the task image needs to include the Go programming language. We will use the `golang:1.6-alpine` image from:

https://hub.docker.com/_/golang/ (see https://imagelayers.io/?images=golang:1.6-alpine for size of layers)

The task file `task_run_tests.yml` includes:

```yaml
image_resource:
  type: docker-image
  source: {repository: golang, tag: 1.6-alpine}

inputs:
- name: resource-tutorial
- name: resource-app
  path: gopath/src/github.com/cloudfoundry-community/simple-go-web-app
```

The `resource-app` resource will place the inbound files for the input into an alternate path. By default we have seen that inputs store their contents in a folder of the same name. The reason for using an alternate path in this example is specific to building & testing Go language applications and is outside the scope of the section.

To run this task within a pipeline:

```
cd ../10_job_inputs
fly set-pipeline --target tutorial --config pipeline.yml --pipeline simple-app --non-interactive
fly unpause-pipeline --target tutorial --pipeline simple-app
```

View the pipeline UI http://192.168.100.4:8080/pipelines/simple-app and notice that the job automatically starts.

The job will pause on the first run at `web-app-tests` task because it is downloading the `golang:1.6-alpine` image for the first time.

The `web-app-tests` output below corresponds to the Go language test output (in case you've not seen it before):

```
ok  	github.com/cloudfoundry-community/simple-go-web-app	0.003s
```

[Go to Top](#concourse-tutorial)

### 11 - Passing task outputs to another task

In section 10 our task `web-app-tests` consumed an input resource and ran a script that ran some unit tests. The task did not create anything new. Some tasks will want to create something that is then passed to another task for further processing (this section); and some tasks will create something that is pushed back out to the external world (next section).

So far our pipelines' tasks' inputs have only come from resources using `get: resource-tutorial` build plan steps.

A task's `inputs` can also come from the `outputs` of previous tasks. All a task needs to do is declare that it publishes `outputs`, and subsequent steps can consume those as `inputs` by the same name.

A task file declares it will publish outputs with the `outputs` section:

```
outputs:
- name: some-files
```

If a task included the above `outputs` section then its `run:` command would be responsible for putting interesting files in the `some-files` directory.

Subsequent tasks (discussed in this section) or resources (discussed in the next section) could reference these interesting files within the `some-files/` directory.

```
cd ../11_task_outputs_to_inputs
fly set-pipeline --target tutorial --config pipeline.yml --pipeline pass-files --non-interactive
fly unpause-pipeline --target tutorial --pipeline pass-files
```

Open http://192.168.100.4:8080/teams/main/pipelines/pass-files in your browser and trigger `job-pass-files`.

In this pipeline's `job-pass-files` there are two task steps `create-some-files` and `show-some-files`:

![pass-files](http://cl.ly/1j32242g0227/download/Image%202016-02-28%20at%205.14.12%20pm.png)

The former creates 4 files into its own `some-files/` directory. The latter gets a copy of these files placed in its own task container filesystem at the path `some-files/`.

The pipeline build plan only shows that two tasks are to be run in a specific order. It does not indicate that `show-files/` is an output of one task and used as an input into the next task.

```yaml
jobs:
- name: job-pass-files
  public: true
  plan:
  - get: resource-tutorial
  - task: create-some-files
    file: resource-tutorial/11_task_outputs_to_inputs/create_some_files.yml
  - task: show-some-files
    file: resource-tutorial/11_task_outputs_to_inputs/show_files.yml
```

Note, task `create-some-files` build output includes the following error:

```
mkdir: can't create directory 'some-files': File exists
```

This is a demonstration that if a task includes `outputs` then those output directories are pre-created and do not need creating.

[Go to Top](#concourse-tutorial)

### 12 - Publishing outputs

So far we have used the `git` resource to fetch down a git repository, and used `git` & `time` resources as triggers. The [`git` resource](https://github.com/concourse/git-resource) can also be used to push a modified git repository to a remote endpoint (possibly different than where the git repo was originally cloned from).

```
cd ../12_publishing_outputs
fly set-pipeline --target tutorial --config pipeline.yml --pipeline publishing-outputs --non-interactive
fly unpause-pipeline --target tutorial --pipeline publishing-outputs
```

Pipeline dashboard http://192.168.100.4:8080/pipelines/publishing-outputs shows that the input resource is erroring (see orange in key):

![broken-resource](http://cl.ly/330n473y3X1s/download/Image%202016-02-28%20at%206.33.26%20pm.png)

The `pipeline.yml` does not yet have a git repo nor its write-access private key credentials.

[Create a Github Gist](https://gist.github.com/) with a single file `bumpme`, and press "Create public gist":

![gist](http://cl.ly/3P1m1m272B2h/download/Image%202016-02-28%20at%206.35.10%20pm.png)

Copy the "SSH" git URL:

![ssh](http://cl.ly/2m303j1r3E3b/download/Image%202016-02-28%20at%206.36.52%20pm.png)

And paste it into the `pipeline.yml` file:

```
---
resources:
- name: resource-gist
  type: git
  source:
    uri: git@gist.github.com:0c2e172346cb8b0197a9.git
    branch: master
    private_key: |
      -----BEGIN RSA PRIVATE KEY-----
      MIIEpQIBAAKCAQEAuvUl9YU...
      ...
      HBstYQubAQy4oAEHu8osRhH...
      -----END RSA PRIVATE KEY-----
```

Also paste in your `~/.ssh/id_rsa` private key (or which ever you have registered with github) into the `private_key` section.
Be careful, If you don't include the pipe symbol after `private_key: ` you will get an error like `Private keys with passphrases are not supported.`

Update the pipeline:

```
fly set-pipeline --target tutorial --config pipeline.yml --pipeline publishing-outputs --non-interactive
```

Revisit the dashboard UI and the orange resource will change to black if it can successfully fetch the new `git@gist.github.com:XXXX.git` repo.

After running the `job-bump-date` job, refresh your gist:

![bumped](http://cl.ly/2w0E3U0y0735/download/Image%202016-02-28%20at%208.47.54%20pm.png)

This pipeline is an example of updating a resource. It has pushed up new git commits to the git repo (your github gist).

_Where did the new commit come from?_

The `bump-timestamp-file.yml` task file describes a single output `updated-gist`:

```yaml
outputs:
  - name: updated-gist
```

The `bump-timestamp-file` task runs the following `bump-timestamp-file.sh` script:

```bash
git clone resource-gist updated-gist

cd updated-gist
echo $(date) > bumpme

git config --global user.email "nobody@concourse.ci"
git config --global user.name "Concourse"

git add .
git commit -m "Bumped date"
```

First, it copied the input resource `resource-gist` into the output resource `updated-gist` (using `git clone` as the preferred `git` way to do this). The modifications are subsequently made to the `updated-gist` directory, including a `git commit`. It is this `updated-gist` and its additional `git commit` that is subsequently pushed back to the gist by the pipeline step:

```yaml
- put: resource-gist
  params: {repository: updated-gist}
```

The `updated-gist` output from the `bump-timestamp-file` task becomes the `updated-gist` input to the `resource-gist` resource (see the [`git` resource](https://github.com/concourse/git-resource) for additional configuration) because their names match.

The `bump-timestamp-file.sh` script needed the `git` CLI tool. It could have installed it each time via `apt-get install git` or similar, but this would have made the task very slow. Instead `bump-timestamp-file.yml` task file uses a different base image `concourse/concourse-ci` that contains `git` CLI (and many other pre-installed packages). The contents of this Docker image are described at https://github.com/concourse/concourse/blob/master/ci/dockerfiles/bosh-cli/Dockerfile

[Go to Top](#concourse-tutorial)

### 13 - Actual pipeline - passing resources between jobs

Finally, it is time to make an actual pipeline - one job passing results to another job upon success.

In all previous sections our pipelines have only had a single job. For all their wonderfulness, they haven't yet felt like actual pipelines. Jobs passing results between jobs. This is where Concourse shines even brighter.

Update the `publishing-outputs` pipeline with a second job `job-show-date` which will run whenever the first job successfully completes:

```yaml
- name: job-show-date
  plan:
  - get: resource-tutorial
  - get: resource-gist
    passed: [job-bump-date]
    trigger: true
  - task: show-date
    config:
      platform: linux
      image_resource:
        type: docker-image
        source: {repository: busybox}
      inputs:
        - name: resource-gist
      run:
        path: cat
        args: [resource-gist/bumpme]
```

Update the pipeline:

```
cd ../13_pipeline_jobs
fly set-pipeline --target tutorial --config pipeline.yml --pipeline publishing-outputs --non-interactive
```

The dashboard UI displays the additional job and its trigger/non-trigger resources. Importantly, it shows our first pipeline:

![pipeline](http://cl.ly/3y0i0Z2v2b2F/download/Image%202016-02-28%20at%209.09.14%20pm.png)

The latest `resource-gist` commit fetched down in `job-show-date` will be the exact commit used in the last successful `job-bump-date` job. If you manually created a new git commit in your gist and manually ran the `job-show-date` job it would continue to use the previous commit it used, and ignore your new commit. *This is the power of pipelines.*

![trigger](http://cl.ly/3x1t0T450g1h/download/pipeline-flow.gif)

[Go to Top](#concourse-tutorial)

### 14 - Parameterized pipelines

In the preceding sections you were asked to add private credentials and personal git URLs into the `pipeline.yml` files. This would make it difficult to share your `pipeline.yml` with anyone who had access to the repository. Not everyone needs nor should have access to the shared secrets.

Concourse pipelines can include `{{parameter}}` parameters for any value in the pipeline YAML file.

Parameters are all mandatory:

```
cd ../14_parameters
fly set-pipeline --target tutorial --config pipeline.yml --pipeline publishing-outputs --non-interactive
```

The error output will be like:

```
failed to evaluate variables into template: 2 error(s) occurred:

* unbound variable in template: 'gist-url'
* unbound variable in template: 'github-private-key'
```

Somewhere secret on laptop create a `credentials.yml` file with keys `gist-url` and `github-private-key`. The values come from your previous `pipeline.yml` files:

```
gist-url: git@gist.github.com:xxxxxxx.git
github-private-key: |
  -----BEGIN RSA PRIVATE KEY-----
  MIIEpQIBAAKCAQEAuvUl9YUlDHWBMVcuu0FH9u2gSi83PkL4o9TS+F185qDTlfUY
  fGLxDo/bn8ws8B88oNbRKBZR6yig9anIB4Hym2mSwuMOUAg5qsA9zm5ArXQBGoAr
  ...
  iSHcGbKdWqpObR7oau2LIR6UtLvevUXNu80XNy+jaXltqo7MSSBYJjbnLTmdUFwp
  HBstYQubAQy4oAEHu8osRhH1VX8AR/atewdHHTm48DN74M/FX3/HeJo=
  -----END RSA PRIVATE KEY-----
```

To pass in your `credentials.yml` file use the `--load-vars-from` or `-l` options:

```
fly set-pipeline --target tutorial --config pipeline.yml --pipeline publishing-outputs --non-interactive --load-vars-from ../credentials.yml
```

[Go to Top](#concourse-tutorial)

## Continuing the tutorial

This tutorial now leaves this README and goes out to the dozens of subfolders. Each has its own README.

Why are there numerical gaps between the section numbers? Because renumbering is hard and so I just left gaps. If we had a cool way to renumber the sections then perhaps wouldn't need the gaps. Sorry :)


### Available concourse resources

In addition to the `git` resource and the `time` resource there are many ways to interact with the external world: getting objects/data or putting objects/data.

Two ideas for finding them:

* https://github.com/concourse?query=resource - the resources bundled with every concourse installation
* https://github.com/search?q=concourse+resource - additional resources shared by other concourse users

Some of the bundled resources include:

-	[bosh-deployment-resource](https://github.com/concourse/bosh-deployment-resource) - deploy bosh releases as part of your pipeline
-	[semver-resource](https://github.com/concourse/semver-resource) - automated semantic version bumping
-	[bosh-io-release-resource](https://github.com/concourse/bosh-io-release-resource) - Tracks the versions of a release on bosh.io
-	[s3-resource](https://github.com/concourse/s3-resource) - Concourse resource for interacting with AWS S3
-	[git-resource](https://github.com/concourse/git-resource) - Tracks the commits in a git repository.
-	[bosh-io-stemcell-resource](https://github.com/concourse/bosh-io-stemcell-resource) - Tracks the versions of a stemcell on bosh.io.
-	[vagrant-cloud-resource](https://github.com/concourse/vagrant-cloud-resource) - manages boxes in vagrant cloud, by provider
-	[docker-image-resource](https://github.com/concourse/docker-image-resource) - a resource for docker images
-	[archive-resource](https://github.com/concourse/archive-resource) - downloads and extracts an archive (currently tgz) from a uri
-	[github-release-resource](https://github.com/concourse/github-release-resource) - a resource for github releases
-	[tracker-resource](https://github.com/concourse/tracker-resource) - pivotal tracker output resource
-	[time-resource](https://github.com/concourse/time-resource) - a resource for triggering on an interval
-	[cf-resource](https://github.com/concourse/cf-resource) - Concourse resource for interacting with Cloud Foundry

To find out which resources are available on your target Concourse you can ask the API endpoint `/api/v1/workers`:

```
$ curl -s http://192.168.100.4:8080/api/v1/workers | jq -r ".[0].resource_types[].type" | sort
archive
bosh-deployment
bosh-io-release
bosh-io-stemcell
cf
docker-image
git
github-release
pool
s3
semver
time
tracker
vagrant-cloud
```

[Go to Top](#concourse-tutorial)
