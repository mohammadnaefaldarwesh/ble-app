<template>
  <div id="app">
    <div class="bottons">
      <button @click="scan" :disabled="scanDisabled">scan</button>
      <button @click="startNotification" :disabled="startDisabled">start</button>
      <button @click="stopNotification" :disabled="stopDisabled">stop</button>
      <button @click="disconnect" :disabled="disconnectDisabled">disconnect</button>
    </div>
    <div class="chart">
      <Chart :chartdata="dataSetsForChart" :options="chartOption" />
    </div>
  </div>
</template>

<script>
import Chart from './components/Chart';
export default {
  name: 'App',
  components: {
    Chart,
  },
  data() {
    return {
      scanDisabled: false,
      startDisabled: true,
      stopDisabled: true,
      disconnectDisabled: true,
      bluetoothDeviceDetected: null,
      characteristics: null,
      tempData: {
        labels: [],
        datasets: [
          {
            label: 'int8',
            backgroundColor: 'transparent',
            borderColor: 'rgba(0,255,0,0.7)',
            data: [],
          },
          {
            label: 'unit8-index(0)',
            backgroundColor: 'transparent',
            borderColor: 'rgba(0,204,255,0.7)',
            data: [],
          },
          {
            label: 'unit8-index(1)',
            backgroundColor: 'transparent',
            borderColor: 'rgba(153,102,255,0.5)',
            data: [],
          },
          {
            label: 'unit16',
            backgroundColor: 'transparent',
            borderColor: 'rgba(0,102,255,0.5)',
            data: [],
          },
        ],
      },
      dataSetsForChart: null,

      chartOption: {
        scales: {
          xAxes: [
            {
              gridLines: {
                display: false,
                drawBorder: false,
              },
              ticks: {
                autoSkip: true,
                minTicksLimit: 3,
                maxTicksLimit: 10,
                minRotation: 55,
              },
            },
          ],
          yAxes: [
            {
              ticks: {
                beginAtZero: true,
                stepSize: 20,
                // callback: function(value) {
                //   return value + '%';
                // },
              },
            },
          ],
        },
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          display: true,
          position: 'bottom',
        },
        elements: {
          point: {
            radius: 1.8,
            hoverRadius: 6,
            hoverBorderWidth: 10,
          },
        },
      },
    };
  },
  methods: {
    async readChValue(x) {
      let ch = await x.readValue();
      return ch;
    },
    isWebBluetoothAvailable() {
      if (!navigator.bluetooth) {
        console.log('Web Bluetooth is not available');
        return false;
      }
      return true;
    },
    async getDeviceInfo() {
      let option = {
        acceptAllDevices: true,
        // filters: [
        //   {
        //     name: 'EChel',
        //   },
        // ],
        optionalServices: [
          '0000fff0-0000-1000-8000-00805f9b34fb',
          'device_information',
          'generic_access',
        ],
      };
      try {
        let device = await navigator.bluetooth.requestDevice(option);
        this.bluetoothDeviceDetected = device;
        await this.bluetoothDeviceDetected.gatt.connect();
        console.log('connected');
        // console.log(this.bluetoothDeviceDetected);
        // test each one :
        // let service = await this.bluetoothDeviceDetected.gatt.getPrimaryService(
        //   '00001800-0000-1000-8000-00805f9b34fb',
        // );

        // let characteristic = await service.getCharacteristics();
        // let descriptors = await characteristic[3].getDescriptors();
        // let descriptorsValue = await descriptors[0].value();
        // console.log(descriptorsValue);
        let customService = await this.bluetoothDeviceDetected.gatt.getPrimaryService(
          '0000fff0-0000-1000-8000-00805f9b34fb',
        );
        console.log('we get the service');
        this.characteristics = await customService.getCharacteristic(
          '0000fff4-0000-1000-8000-00805f9b34fb',
        );
        console.log('we get the characteristics');
        this.characteristics.addEventListener(
          'characteristicvaluechanged',
          this.handelChangedValue,
        );
        console.log('add the event listener');
        console.log('now you cant start the notifications');
        this.startDisabled = false;
        this.scanDisabled = true;
        this.disconnectDisabled = false;
        // await this.characteristics.startNotifications();
        // console.log(this.characteristics);
        // let service = await this.bluetoothDeviceDetected.gatt.getPrimaryServices();
        // console.log({ services: service });
        // let characteristic = [];
        // await Promise.all(
        //   service.map(async serv => {
        //     characteristic.push(await serv.getCharacteristics());
        //   }),
        // );

        // console.log({ characteristics: characteristic });

        // this.gattCharacteristic = characteristic;
        // // let descriptors = [];
        // await Promise.all(
        //   this.gattCharacteristic.map(characteristic => {
        //     characteristic.map(async char => {
        //       char.addEventListener('characteristicvaluechanged', this.handelChangedValue);
        //       // let encoder = new TextEncoder('utf-8');
        //       // let userDescription = encoder.encode('hello...');
        //       // char.properties.write && char.writeValue(userDescription)
        //       //   ? console.log(this.readChValue(char))
        //       //   : null;

        //       char.properties.notify && (await char.startNotifications());
        //       // this.readChValue(char);
        //     });
        //   }),
        // );
        // this.gattCharacteristic[0][0].properties.indicate &&
        //   descriptors.push(await this.gattCharacteristic[0][0].getDescriptors());
        // // let desValue = await descriptors[0][0].readValue();

        // let descriptorValue = await descriptors[0][0].readValue();
        // // let encoder = new TextEncoder('utf-8');
        // // console.log('User Description: ' + encoder.encode(descriptorValue));
        // let decoder = new TextDecoder('utf-8');
        // console.log('descriptor: ' + decoder.decode(descriptorValue));
        // // await descriptors[0][0].writeValue(new Uint8Array([1]));
        // console.log(descriptorValue.getUint8(), descriptorValue.getUint16());
      } catch (err) {
        console.log('error: ' + err);
      }
    },
    getConnection(e) {
      e.stopPropagation();
      e.preventDefault();
      this.isWebBluetoothAvailable() && this.getDeviceInfo();
    },
    handelChangedValue(event) {
      let value = event.target.value;
      // let encoder = new TextEncoder('utf-8');
      // console.log('encoder: ' + encoder.encode(value));
      // let decoder = new TextDecoder('utf-8');
      // console.log('decoder: ' + decoder.decode(value));
      // console.log(value);
      console.log(
        value.getInt8(0),
        value.getInt8(1),
        value.getUint8(0),
        value.getUint8(1),
        value.getInt16(),
      );

      this.tempData.labels.push(Date.now());
      this.tempData.datasets[0].data.push(value.getInt8(1));
      this.tempData.datasets[1].data.push(value.getUint8(0));
      this.tempData.datasets[2].data.push(value.getUint8(1));
      this.tempData.datasets[3].data.push(value.getInt16());

      this.dataSetsForChart = {
        labels: this.tempData.labels,
        datasets: this.tempData.datasets,
      };
    },
    async scan(e) {
      try {
        await this.getConnection(e);
      } catch (err) {
        console.error(err);
      }
    },
    async startNotification() {
      try {
        await this.characteristics.startNotifications();
        this.startDisabled = true;
        this.stopDisabled = false;
      } catch (err) {
        console.error(err);
      }
    },
    async stopNotification() {
      try {
        await this.characteristics.stopNotifications();
        this.startDisabled = false;
        this.stopDisabled = true;
      } catch (err) {
        console.error(err);
      }
    },
    async disconnect() {
      try {
        await this.bluetoothDeviceDetected.gatt.disconnect();
        this.startDisabled = true;
        this.startDisabled = true;
        this.disconnectDisabled = true;
        this.scanDisabled = false;
        console.log('disconnected');
      } catch (err) {
        console.error(err);
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.bottons {
  display: flex;
  justify-content: space-between;
}
.chart {
  margin-top: 50px;
}
</style>
