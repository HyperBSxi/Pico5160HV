# Pico5160HV

## 特徴

* **MCU**: Raspberry Pi RP2350
* **ファームウェア**: Klipperで動作確認済み
* **電源入力**: 24V～55V 単一電源
* **通信**: USBまたはCAN BusでKlipperホストと接続可能
* **ファン出力**: PWM制御対応 12Vファン出力（100Vまでのサージ保護内蔵）
* **エンドストップ**: 4つのエンドストップまたはDIAG1入力に切り替え可能
* **CAN終端抵抗**: 有無を選択可能。2つのCANコネクタを搭載（中継用、電気的に接続済み）
* **低電圧保護**: 12Vおよび3.3Vの低電圧を検知するとMCUがシャットダウン
* **非搭載機能**: サーミスタ入力、ベッド出力、ホットエンド出力

## 物理仕様

* **基板サイズ**: 100mm × 100mm
* **固定穴**: M3、穴間距離 92mm × 92mm

---

## ファームウェアのコンパイル手順

### 1. BOOTモードで起動

電源投入後、USBコネクタ付近の `BOOTSEL` ボタンを押しながら `RESET` ボタンを押すことでBOOTモードに入ります。

### 2. デバイス認識の確認

ターミナルで以下を実行し、「Raspberry Pi RP2350 Boot」が表示されるか確認してください：

```bash
$ lsusb
```

表示されたUSB IDを控えておきます。

### 3. `make menuconfig` による構成

```bash
# klipperディレクトリに移動して構成を開始
$ make menuconfig
```

設定内容：

* `[*] Enable extra low-level configuration options` → チェックを入れる
* `Micro-controller Architecture` → **Raspberry Pi RP2040/RP235x**
* `Processor model` → **rp2350**
* `Bootloader offset` → **No bootloader**
* `Communication Interface` → **USB serial** または **CAN bus** を選択

  * CAN使用時：

    * `CAN RX gpio number`: `26`
    * `CAN TX gpio number`: `27`
    * `CAN bus speed`: 最大値は 1,000,000（1Mbps）
* `GPIO pins to set at micro-controller startup`:

  * USB接続：`gpio25`
  * CAN接続：`gpio25,gpio28`

設定後、`q` で終了 → `Y` で保存

### 4. コンパイルと書き込み

```bash
$ make flash FLASH_DEVICE=[ステップ2で確認したUSB ID]
```

### 5. Klipperでの認識確認

#### USB接続の場合

```bash
$ ls -l /dev/serial/by-id/
```

→ `usb-Klipper_rp2350_xxxxxxxxxxxxxxxx-if00` が表示されれば成功

#### CAN接続の場合

```bash
$ ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```

→ `Found canbus_uuid=xxxxxxxxxxxx` が表示されれば成功
※複数デバイスがある場合、Pico5160HVの電源を一時的に切って識別してください

---

## Klipperのconfig設定

1. `Pico5160HV_RevA.cfg` を `klipper.cfg` と同じディレクトリに配置
2. `klipper.cfg` の末尾に以下を追記：

```ini
[include Pico5160HV_RevA.cfg]
```

3. `Pico5160HV_RevA.cfg` の `[stepper]` および `[tmc5160]` セクションを環境に応じて編集
   ※`klipper.cfg` 側と重複するセクションは `klipper.cfg` 側をコメントアウト
