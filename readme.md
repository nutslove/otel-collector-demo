## 概要
- Spring PetClinic Demo Applicationを使って、Opentelemetry Collectorで様々なバックエンド(New Relic, Jaeger/VictoriaMetrics/Lokiなど)にtelemetry data(metrics,logs,traces)を送信するデモ

## 準備するもの
- New Relicのアカウント/ライセンスキー（100GBまで無料）
- AWSアカウントとAmazon Linux2インスタンス３台
  - javaアプリサーバ
  - Otel-Collectorサーバ
  - バックエンドサーバ（Jaeger、Prometheus、Loki、Grafana、・・・）

## セットアップ
#### ■ サンプルアプリ用EC2側
- Javaをインストールする
  - https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/amazon-linux-install.html
    ~~~
    sudo yum install java-17-amazon-corretto-devel
    ~~~
- OpenTelemetry Java auto-intrumention agentをダウンロードする
  - https://github.com/open-telemetry/opentelemetry-java-instrumentation
- Fluent Bit(ログ連携用)をインストールする
  - https://docs.fluentbit.io/manual/installation/linux/amazon-linux#install-on-amazon-linux-2
  - Fluent BitのConfiguration
    - https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/classic-mode/configuration-file
- Node Exporterをインストールする
  ~~~
  wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
  tar xzvf node_exporter-1.6.1.linux-amd64.tar.gz
  mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
  ~~~
  - systemdに登録する
    - `vi /etc/systemd/system/node_exporter.service`
      ~~~
      [Unit]
      Description=Node Exporter

      [Service]
      ExecStart=/usr/local/bin/node_exporter

      [Install]
      WantedBy=multi-user.target
      ~~~
    - node_exporterを有効化/実行する
      ~~~
      sudo systemctl daemon-reload
      sudo systemctl enable node_exporter
      sudo systemctl start node_exporter
      ~~~
- サンプルJavaアプリを実行する
  - **javaagentを先に指定すること！**
  ~~~
  java -javaagent:<opentelemetry-javaagent.jarのフルパス> -Dotel.exporter.otlp.endpoint=http://<OpenTelemetry-CollectorのIP>:4318 -Dotel.exporter.otlp.protocol=http/protobuf -Dotel.service.name=petclinic -jar target/*.jar
  ~~~

#### ■ Opentelemetry Collector用EC2側
- Opentelemetry Collectorをインストールする
  ~~~
  sudo yum update
  sudo yum -y install wget systemctl
  wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.88.0/otelcol_0.88.0_linux_amd64.rpm
  sudo rpm -ivh otelcol_0.88.0_linux_amd64.rpm
  ~~~
- `systemctl status otelcol`でOtel-Collectorが動いていることを確認
> [!NOTE]  
> New Relicを使う場合は以下のようにライセンスキーを環境変数として設定する
    ~~~
    export NEW_RELIC_LICENSE_KEY=<YOUR_KEY_HERE>
    ~~~

#### ■ Backend用EC2側
- `yum install git`でgitをインストールする
- `sudo amazon-linux-extras install docker`でDockerをインストールする
- `systemctl enable docker && systemctl start docker`でDocker実行する
- Docker Composeをインストールする
  - https://gist.github.com/npearce/6f3c7826c7499587f00957fee62f8ee9
  ~~~
  sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  docker-compose version
  ~~~
- 

- Docker ComposeでOpentelemetry Collector、Jaager、Prometheus、Loki、Grafanaを起動する
  - `docker-compose up -d`
