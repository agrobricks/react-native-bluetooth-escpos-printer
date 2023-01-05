# react-native-bluetooch-escpos-printer

React-Native plugin for the bluetooth ESC/POS printers.

Any questions or bug please raise a issue.

##Still under developement

#May support Android /IOS

[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/agrobricks/react-native-bluetooth-pocket-printer/master/LICENSE) [![npm version](https://badge.fury.io/js/react-native-bluetooth-pocket-printer.svg)](https://www.npmjs.com/package/react-native-bluetooth-pocket-printer)

## Installation
### Step 1 ###
Install via NPM [Check In NPM](https://www.npmjs.com/package/react-native-bluetooth-pocket-printer)
```bash
npm install react-native-bluetooth-pocket-printer --save
```

Or install via github
```bash
npm install https://github.com/agrobricks/react-native-bluetooth-pocket-printer.git --save
```

### Step2 ###
Link the plugin to your RN project
```bash
react-native link react-native-bluetooth-pocket-printer
```

### Manual linking (Android) ###
Ensure your build files match the following requirements:

1. (React Native 0.59 and lower) Define the *`react-native-bluetooth-pocket-printer`* project in *`android/settings.gradle`*:

```
include ':react-native-bluetooth-pocket-printer'
project(':react-native-bluetooth-pocket-printer').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-bluetooth-pocket-printer/android')
```
2. (React Native 0.59 and lower) Add the *`react-native-bluetooth-pocket-printer`* as an dependency of your app in *`android/app/build.gradle`*:
```
...
dependencies {
  ...
  implementation project(':react-native-bluetooth-pocket-printer')
}
```

3. (React Native 0.59 and lower) Add *`import cn.jystudio.bluetooth.RNBluetoothEscposPrinterPackage;`* and *`new RNBluetoothEscposPrinterPackage()`* in your *`MainApplication.java`* :



### Step3 ###
Refers to your JS files
```javascript
    import {BluetoothManager,BluetoothEscposPrinter} from 'react-native-bluetooth-pocket-printer';
```

## Usage and APIs ##

### BluetoothManager ###
BluetoothManager is the module for Bluetooth service management, supports Bluetooth status check, enable/disable Bluetooth service, scan devices, connect/unpair devices.

* isBluetoothEnabled ==>
async function, checks whether Bluetooth service is enabled.
//TODO: consider to return the the devices information already bound and paired here..

```javascript
     BluetoothManager.isBluetoothEnabled().then((enabled)=> {
                alert(enabled) // enabled ==> true /false
            }, (err)=> {
                alert(err)
            });
```

* enableBluetooth ==> ``` diff + ANDROID ONLY ```
async function, enables the bluetooth service, returns the devices information already bound and paired.  ``` diff - IOS would just resovle with nil ```

```javascript
BluetoothManager.enableBluetooth().then((r)=>{
                var paired = [];
                if(r && r.length>0){
                    for(var i=0;i<r.length;i++){
                        try{
                            paired.push(JSON.parse(r[i])); // NEED TO PARSE THE DEVICE INFORMATION
                        }catch(e){
                            //ignore
                        }
                    }
                }
                console.log(JSON.stringify(paired))
            },(err)=>{
               alert(err)
           });
```

* disableBluetooth ==>  ``` diff + ANDROID ONLY ```
async function ,disables the bluetooth service. ``` diff - IOS would just resovle with nil ```

```javascript
BluetoothManager.disableBluetooth().then(()=>{
            // do something.
          },(err)=>{alert(err)});
```

* scanDevices ==>
async function , scans the bluetooth devices, returns devices found and paired after scan finish. Event [BluetoothManager.EVENT_DEVICE_ALREADY_PAIRED] would be emitted with devices bound; event [BluetoothManager.EVENT_DEVICE_FOUND] would be emitted (many time) as long as new devices found.

samples with events:
```javascript
 DeviceEventEmitter.addListener(
            BluetoothManager.EVENT_DEVICE_ALREADY_PAIRED, (rsp)=> {
                this._deviceAlreadPaired(rsp) // rsp.devices would returns the paired devices array in JSON string.
            });
        DeviceEventEmitter.addListener(
            BluetoothManager.EVENT_DEVICE_FOUND, (rsp)=> {
                this._deviceFoundEvent(rsp) // rsp.devices would returns the found device object in JSON string
            });
```

samples with scanDevices function
```javascript
BluetoothManager.scanDevices()
            .then((s)=> {
                var ss = JSON.parse(s);//JSON string
                this.setState({
                    pairedDs: this.state.pairedDs.cloneWithRows(ss.paired || []),
                    foundDs: this.state.foundDs.cloneWithRows(ss.found || []),
                    loading: false
                }, ()=> {
                    this.paired = ss.paired || [];
                    this.found = ss.found || [];
                });
            }, (er)=> {
                this.setState({
                    loading: false
                })
                alert('error' + JSON.stringify(er));
            });
```

* connect ==>
async function, connects the specified device, if not bound, bound dailog prompts.

```javascript

    BluetoothManager.connect(rowData.address) // the device address scanned.
     .then((s)=>{
       this.setState({
        loading:false,
        boundAddress:rowData.address
    })
    },(e)=>{
       this.setState({
         loading:false
     })
       alert(e);
    })

```

* unpair ==>
async function, disconnects and unpairs the specified devices

```javascript
     BluetoothManager.connect(rowData.address)
     .then((s)=>{
        //success here
     },
     (err)=>{
        //error here
     })
```

* Events of BluetoothManager module

| Name/KEY | DESCRIPTION |
|---|---|
| EVENT_DEVICE_ALREADY_PAIRED | Emits the devices array already paired |
| EVENT_DEVICE_DISCOVER_DONE | Emits when the scan done |
| EVENT_DEVICE_FOUND | Emits when device found during scan |
| EVENT_CONNECTION_LOST | Emits when device connection lost |
| EVENT_UNABLE_CONNECT | Emits when error occurs while trying to connect device |
| EVENT_CONNECTED | Emits when device connected |
| EVENT_BLUETOOTH_NOT_SUPPORT | Emits when device not support bluetooth(android only) |

### BluetoothEscposPrinter ###
  the printer for receipt printing, following ESC/POS command.

#### printerInit() ####
  init the printer.

#### printAndFeed(int feed) ####
  printer the buffer data and feed (feed lines).

#### printerLeftSpace(int sp) ####
  set the printer left spaces.

#### printerLineSpace(int sp) ####
  set the spaces between lines.

#### printerUnderLine(int line) ####
  set the underline of the text, @param line --  0-off,1-on,2-deeper

#### printerAlign(int align) ####
  set the printer alignment, constansts: BluetoothEscposPrinter.ALIGN.LEFT/BluetoothEscposPrinter.ALIGN.CENTER/BluetoothEscposPrinter.ALIGN.RIGHT.
  Does not work on printPic() method.

#### printText(String text, ReadableMap options) ####
  print text, options as following:
  * encoding => text encoding,default GBK.
  * codepage => codepage using, default 0.
  * widthtimes => text font mul times in width, default 0.
  * heigthTimes => text font mul times in height, default 0.
  * fonttype => text font type, default 0.

#### printColumn(ReadableArray columnWidths,ReadableArray columnAligns,ReadableArray columnTexts,ReadableMap options) ####
  print texts in column, Parameters as following:
  * columnWidths => int arrays, configs the width of each column, calculate by english character length. ex:the width of "abcdef" is 5 ,the width of "中文" is 4.
  * columnAligns => arrays, alignment of each column, values is the same of printerAlign().
  * columnTexts => arrays, the texts of each colunm to print.
  * options => text print config options, the same of printText() options.

#### setWidth(int width) ####
  sets the width of the printer.

#### printPic(String base64encodeStr,ReadableMap options) ####
  prints the image which is encoded by base64, without schema.
  * options: contains the params that may use in printing pic: "width": the pic width, basic on devices width(dots,58mm-384); "left": the left padding of the pic for the printing position adjustment.

#### setfTest() ####
  prints the self test.

#### rotate() ####
  sets the rotation of the line.

#### setBlob(int weight) ####
  sets blob of the line.

#### printQRCode(String content, int size, int correctionLevel) ####
  prints the qrcode.

#### printBarCode(String str,int nType, int nWidthX, int nHeight, int nHriFontType, int nHriFontPosition) ####
  prints the barcode.

### Demos of printing a receipt ###
```javascript
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await BluetoothEscposPrinter.setBlob(0);
await  BluetoothEscposPrinter.printText("广州俊烨\n\r",{
  encoding:'GBK',
  codepage:0,
  widthtimes:3,
  heigthtimes:3,
  fonttype:1
});
await BluetoothEscposPrinter.setBlob(0);
await  BluetoothEscposPrinter.printText("销售单\n\r",{
  encoding:'GBK',
  codepage:0,
  widthtimes:0,
  heigthtimes:0,
  fonttype:1
});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.LEFT);
await  BluetoothEscposPrinter.printText("客户：零售客户\n\r",{});
await  BluetoothEscposPrinter.printText("单号：xsd201909210000001\n\r",{});
await  BluetoothEscposPrinter.printText("日期："+(dateFormat(new Date(), "yyyy-mm-dd h:MM:ss"))+"\n\r",{});
await  BluetoothEscposPrinter.printText("销售员：18664896621\n\r",{});
await  BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
let columnWidths = [12,6,6,8];
await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["商品",'数量','单价','金额'],{});
await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["React-Native定制开发我是比较长的位置你稍微看看是不是这样?",'1','32000','32000'],{});
    await  BluetoothEscposPrinter.printText("\n\r",{});
  await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["React-Native定制开发我是比较长的位置你稍微看看是不是这样?",'1','32000','32000'],{});
await  BluetoothEscposPrinter.printText("\n\r",{});
await  BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
await BluetoothEscposPrinter.printColumn([12,8,12],
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["合计",'2','64000'],{});
await  BluetoothEscposPrinter.printText("\n\r",{});
await  BluetoothEscposPrinter.printText("折扣率：100%\n\r",{});
await  BluetoothEscposPrinter.printText("折扣后应收：64000.00\n\r",{});
await  BluetoothEscposPrinter.printText("会员卡支付：0.00\n\r",{});
await  BluetoothEscposPrinter.printText("积分抵扣：0.00\n\r",{});
await  BluetoothEscposPrinter.printText("支付金额：64000.00\n\r",{});
await  BluetoothEscposPrinter.printText("结算账户：现金账户\n\r",{});
await  BluetoothEscposPrinter.printText("备注：无\n\r",{});
await  BluetoothEscposPrinter.printText("快递单号：无\n\r",{});
await  BluetoothEscposPrinter.printText("打印时间："+(dateFormat(new Date(), "yyyy-mm-dd h:MM:ss"))+"\n\r",{});
await  BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
await  BluetoothEscposPrinter.printText("电话：\n\r",{});
await  BluetoothEscposPrinter.printText("地址:\n\r\n\r",{});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await  BluetoothEscposPrinter.printText("欢迎下次光临\n\r\n\r\n\r",{});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.LEFT);
```

### Demo for opening the drawer ###
```javascript
BluetoothEscposPrinter.opendDrawer(0, 250, 250);
```
