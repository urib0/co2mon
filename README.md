# co2mon

Hec-EyeにつながるCO2モニタ

## ビルド

```sh
docker buildx create --use         # 最初の1回のみ
docker buildx inspect --bootstrap  # 最初の1回のみ
./docker_build.sh
```

## DockerイメージをRaspberryPiに転送する

```sh
docker image save co2mon | ssh cm01.local docker image load
```

**ビルドして転送**

```sh
./docker_build.sh && (docker image save co2mon | pv | ssh cm01.local docker image load)
```

## コンテナの起動

```sh
docker run -d --privileged --rm -v /var/local/co2mon:/var/local/co2mon --name co2mon co2mon /sbin/init
```

**ビルドして転送して起動**

```
./docker_build.sh && (docker image save co2mon | pv | ssh cm01.local docker image load) && ssh cm01.local 'docker stop co2mon; docker run -d --privileged --rm -v /var/local/co2mon:/var/local/co2mon --name co2mon co2mon /sbin/init'
```

または,

```sh
./build_send_run.sh
```

## コンテナのシェルを開く

```sh
docker exec -it co2mon /bin/bash --login
```

## USBコネクタ

- 左右はコネクタ側から見たときの左右
- 指す場所はこれで仮決定として、udevの設定がよくわかるまでは `GPS=/dev/ttyUSB0` `CO2=/dev/ttyACM0` としておく（コンテナ内でもこのような名前で認識されているので）

### Pi3

### Pi4

**GPS(上段左側)**

```
/dev/serial/by-path/platform-fd500000.pcie-pci-0000\:01\:00.0-usb-0\:1.3\:1.0-port0
```

**CO2センサ(上段右側)**

```
/dev/serial/by-path/platform-fd500000.pcie-pci-0000\:01\:00.0-usb-0\:1.1\:1.0
```

## メモ

### USBシリアルデバイスの自動判別(案)

- `/dev/serial/by-path` 以下にすべてのシリアルデバイスが列挙される
- `tail -n 10`でファイルに10行読み出す
- 3秒経ったらtailのプロセスをkillしてwaitする
- 読みだしたファイルの中身をみてデバイスの種類を判別する
