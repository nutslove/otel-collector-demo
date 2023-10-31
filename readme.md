## 概要
- Spring PetClinic Demo Applicationを使って、Opentelemetry Collectorで様々なバックエンド(New Relic, Jaeger/VictoriaMetrics/Lokiなど)にtelemetry data(metrics,logs,traces)を送信するデモ

## 準備するもの
- New Relicのアカウント/ライセンスキー（100GBまで無料）
- AWSアカウントとAmazon Linux2インスタンス2個

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
- Node Exporterをインストールする


#### ■ Opentelemetry Collector用EC2側
- `sudo amazon-linux-extras install docker`でDockerをインストールする
- Docker Composeをインストールする
  ~~~
  sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  docker-compose version
  ~~~
- Docker ComposeでOpentelemetry Collector、Jaager、Prometheus、Loki、Grafanaを起動する
  - `docker-compose -d -f docker-compose.yml`
> [!NOTE]  
> New Relicを使う場合は以下のようにライセンスキーを環境変数として設定する
    ~~~
    export NEW_RELIC_LICENSE_KEY=<YOUR_KEY_HERE>
    ~~~
