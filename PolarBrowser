<!DOCTYPE html>
<!--Authers Francis Kirchman, Raphael Schanz, Iliya Vasilev-->



<html>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<body>

<h1>Heartscope <br>
</h1>


<table id = "table">
    <tr>
        <td>Sensortype</td>
        <td>User</td>
        <td>Sensor ID</td>
    </tr>
</table>

<style>
    table {
        font-family: arial, sans-serif;
        border-collapse: collapse;
        width: 50%;
    }
    td, th {
        border: 1px solid black;
        text-align: left;
        padding: 8px;
    }
</style>>

<button id = 'connect_Verity' 
    onclick = "connectVerity(options, 'Verity')"
    >Connect Verity
</button>

<button id = 'connect_H10' 
    onclick = "connectDevice(options, 'H10')"
    >Connect H10
</button>

<button type = "button"
    onclick = "enableSDK()"
    >Enable SDK mode(?only for Verity?)
</button>

<button type = "button"
    onclick = "startPPG()"
    >Start PPG(Verity)
</button>

<button type = "button"
    onclick = "startECG()"
    >Start ECG(H10)
</button>

<button onclick="download_csv_file()"> Download CSV </button>

<div id="chart"></div>
<div id="chart0"></div>
<div id="chart1"></div>
<div id="chart2"></div>
<div id="chart3"></div>
<div id="chart4"></div>

<p id="bpm">bpm: </p>

</body>

<script>
    Control_Types=[
        "",
        "Settings request",
        "Start measurement",
        "Stop measurement"
    ];
    Stream_Types=["ECG","PPG","ACC",,,,,,,"SDK"];
    Device_Errors=[
        "Success",
        "Not supported",
        "Stream not known",
        "Stream not supported",
        "Invalid length",
        "Invalid parameter", 
        "Already doing it!", 
        "Invalid Resolution", 
        "Invalid sample rate",
        "Invalid range",
        "Invalid MTU", 
        "Invalid channels",
        "Invalid state",
        "Sorry, charging"
    ];
    PMD_SERVICE = "fb005c80-02e7-f387-1cad-8acd2d8df0c8"; 
    PMD_CONTROL = "fb005c81-02e7-f387-1cad-8acd2d8df0c8"; 
    PMD_DATA = "fb005c82-02e7-f387-1cad-8acd2d8df0c8";
    const output = {
        Verity:['SDK','PPG','ACC_Sense'],
        H10:  ['ECG','ACC_H10']
    };

    var dataTime
    var deviceNameSplit
    var data
    var samplerate = 28
    var bpmValue = 0;

    var layout = {
        xaxis: {
        range: [0,365]
        },

        yaxis: {
        range: [-750, 1100]
        }
    }

    var csvFileData = [];
    var csv = 'Date,Time,User,Amplitude,BPM\n';  

    var arrayLength = 90
    var newArray = []

    const findDevice = function(element, Device) {
        if(element.device.id == Device.id){return true} else{return false}
    }

    class ConnectedDevices {
        constructor() {
            this.connectedDevices = [];
        }
        register(DeviceObj) {
            this.connectedDevices.push(DeviceObj)
        }
        remove(Device) {
            let index = this.find(Device)
            this.connectedDevices.splice(index, 1)
        }
        find(Device){
            return this.connectedDevices.findIndex(findDevice, Device);
        }
    }

    const instancedDevices = new ConnectedDevices();

    CustomException.prototype = Object.create(Error. prototype);
    var dataTime
    var deviceNameSplit
    var data
    var samplerate = 28
    var globalECGdata = [];
    var globalECGdata1 = [];
    //may not be used
    var cnt = 0;

    class Device{
        constructor(type, user) {
            this.type = type
            this.output = output[type]
            this.user = user
        }
        getDevice(){
            return this;
        }
    }


    var count = 0;
    var deviceIDs = [];

    var H10_0 = "";
    var H10_1 = "";
    //quick fix, should be done in connectDevice
    function connectVerity(abc, abcd) {

        console.log('Requesting Bluetooth Device...');
        navigator.bluetooth.requestDevice({filters: [{services: ['heart_rate']}]})
        .then(device => {
            console.log('Connecting to GATT Server...');
            return device.gatt.connect();
        })
        .then(server => {
            console.log('Getting Service...');
            return server.getPrimaryService('heart_rate');
        })
        .then(service => {
            console.log('Getting Characteristic...');
            return service.getCharacteristic('heart_rate_measurement');
        })
        .then(characteristic => {
            myCharacteristic = characteristic;
            return myCharacteristic.startNotifications().then(_ => {
            console.log('> Notifications started');
            myCharacteristic.addEventListener('characteristicvaluechanged',
                handleNotifications);
            });
        })
        .catch(error => {
            console.log('Argh! ' + error);
        });
    }

    function handleNotifications(event) {
        bpmValue = event.target.value;
        bpmValue = bpmValue.getUint16(0);
        console.log(bpmValue);
        document.getElementById("bpm").innerHTML = bpmValue;
    }

    async function connectDevice(options, name) {
        try {
            user = await addUser()
            if(user == null) { throw new CustomException('No input from user') }

            deviceObj = new Device(name, user)

            console.log('Requesting ' + name + ' Sensor..');
            deviceObj.device = await navigator.bluetooth.requestDevice(options)
            await deviceObj.device.addEventListener('gattserverdisconnected', onDisconnected);

            deviceIDs[count] = deviceObj.device.id;
            console.log("DEVICE ID FIRST CONNECT:" + deviceIDs[count]);
            count++;

            //bug in find
            if(instancedDevices.find(deviceObj.device) == -1){
                instancedDevices.register(deviceObj);
            } else { throw new CustomException('Already connected') }

            insertTable(deviceObj);

            console.log(deviceObj.device)

            console.log('Connecting ' + name + ' Sensor..');
            deviceObj.server = await deviceObj.device.gatt.connect()
            console.log(deviceObj.server)
            
            console.log('Getting ' + Object.keys({PMD_SERVICE})[0] + ' Service..');
            deviceObj.service = await deviceObj.server.getPrimaryService(PMD_SERVICE)
            console.log(deviceObj.service)

            console.log('Getting ' + Object.keys({PMD_CONTROL})[0] + ' Characteristic..');
            deviceObj.character = await deviceObj.service.getCharacteristic(PMD_CONTROL)
            console.log(deviceObj.character)

            console.log('addEventHandler to PMD_CONTROL for ' + name + '..');
            setEventHandler(deviceObj.character, handlePMD_CONTROL)

            console.log('Getting ' + Object.keys({PMD_DATA})[0] + ' Characteristic..')
            deviceObj.data = await deviceObj.service.getCharacteristic(PMD_DATA)
            console.log(deviceObj.data)

            console.log('addEventHandler to PMD_DATA for ' + name + '..');
            setEventHandler(deviceObj.data, handlePMD_DATA)


                if(name == "Verity"){
                    ButtonColour("connect_Verity", "Lime", true, "Disconnect Verity");
                }

                if(name == "H10"){
                    ButtonColour("connect_H10", "Lime", true, "Connect another device");
                }
        } catch(error) {
            console.log('Argh! ' + error);
            if(error != 'Error: No input from user') { 
            };
        }
    }

    function CustomException(message) {
        const error = new Error(message)
        error.code = "NO_USER_INPUT"
        return error;
    }

    var H10_0 = "0";
    var H10_1 = "0";
    var H10_2 = "0";
    var H10_3 = "0";
    var H10_4 = "0";

    async function setEventHandler(character, handlerFunc) {
        await character.startNotifications()
        .then(_ => {
            console.log('EventHandler ' + handlerFunc.name + ' added');
            character.addEventListener('characteristicvaluechanged', handlerFunc);
        })
    }
    //PMD_CONTROL response
    function handlePMD_CONTROL(event) {
        deviceName = event.currentTarget.service.device.name;
        response = new Uint8Array(event.currentTarget.value.buffer);
        console.log(deviceName + " (timeStamp = ",event.timeStamp + ") " +
        Control_Types[response[1]] + Stream_Types[response[2]] + 
        Device_Errors[response[3]]);
    }



    function handlePMD_DATA(event) {
        dataTime = Date.now()

        deviceNameSplit = event.currentTarget.service.device.name.split(' ')[1]
        data = event.target.value.buffer
        var DataType = Number(new Uint8Array(data.slice(0,1)));

        if (event.currentTarget.service.device.id == deviceIDs[0]){
            
        samples = new Uint8Array(data.slice(10,))
        npoints=samples.byteLength/3;
        ECGdata=[];

        for(offset=0; offset<samples.byteLength; offset+=3) {
            //i = iteration
            i=offset/3;
            ECGdata[i]=WordstoSignedInteger(samples.slice(offset,offset+2),8);
        }

        globalECGdata = globalECGdata.concat(ECGdata);

        let date = new Date();
        let entry = [];
        entry = [String(date.getFullYear() + '.' + date.getMonth() + '.' + date.getDate()), 
                String(date.getHours() + ':' + date.getMinutes()),
                String(event.currentTarget.service.user),
                String(Math.max(...ECGdata)),
                String(bpmValue)];
        csvFileData.push(entry);
        }

        else if (event.currentTarget.service.device.id == deviceIDs[1]){
            
        samples = new Uint8Array(data.slice(10,))
        npoints=samples.byteLength/3;
        ECGdata=[];

        for(offset=0; offset<samples.byteLength; offset+=3) {
            //i = iteration
            i=offset/3;
            ECGdata[i]=WordstoSignedInteger(samples.slice(offset,offset+2),8);
        }
        globalECGdata1 = globalECGdata1.concat(ECGdata);



        Plotly.newPlot('chart0', [{ y: ECGdata,mode: 'lines',line: {color: '#80CAF6'}}],layout);

            console.log("UpdateGraph2");

            var interval = setInterval(function() {
                var data_update = {
                y: [globalECGdata1]
                }

                Plotly.update('chart0', data_update, layout)

                if(globalECGdata1.length > 365) {
                    globalECGdata1.splice(0, 365)
                }
                //if(++cnt === 10) clearInterval(interval);

                }, 1); 
    }


        }

        function download_csv_file() {

        //merge the data with CSV
        csvFileData.forEach(function(row) {
                csv += row.join(',');
                csv += "\n";
        });

        var hiddenElement = document.createElement('a');
        hiddenElement.href = 'data:text/csv;charset=utf-8,' + encodeURI(csv);
        hiddenElement.target = '_blank';

        //provide the name for the CSV file to be downloaded
        hiddenElement.download = 'bpmAndStuff.csv';
        hiddenElement.click();
        }





        //drawGraph(ECGdata)
        
        //updateGraphs(ECGdata
    //not used 
    async function enableSDK() {
        const enableSdkFlag = [0x02,0x09];
        instancedDevices.connectedDevices.forEach(async function (item, index) {
            console.log(item, index);
            if(item.type = "Verity") {
                await item.character.writeValue(new Uint8Array(enableSdkFlag))
                .then(_ => {
                console.log('SDK enabled. ');
                })
                .catch(error => { console.error(error); });
            }
        });
    }
    //not used
    async function startPPG() {
        const startPpgFlag = [0x02,0x01];
        const paramsPPG =  [0x00, 0x01, samplerate, 0x00, 0x01, 0x01, 0x16, 0x00, 0x04, 0x01, 0x04]
        BytesToSend = startPpgFlag.concat(paramsPPG)

        instancedDevices.connectedDevices.forEach(async function (item, index) {
            console.log(item, index);
            if(item.type = "Verity") {
                await item.character.writeValue(new Uint8Array(BytesToSend))
                .then(_ => {
                    console.log('PPG enabled. ');
                })
                .catch(error => { console.error(error); });
            }
        });
        console.log("updateGraphs")
        //updateGraphs();
        console.log("updateGraphs2")
        updateGraphs2();
    }

    async function startECG() {
        const startEcgFlag = [0x02,0x00];
        const paramsECG =  [0x00, 0x01, 0x82, 0x00, 0x01, 0x01, 0x0E, 0x00]
        BytesToSend = startEcgFlag.concat(paramsECG)

        instancedDevices.connectedDevices.forEach(async function (item, index) {
            console.log(item, index);
            if(item.type = "H10") {
                await item.character.writeValue(new Uint8Array(BytesToSend))
                .then(_ => {
                    console.log('ECG enabled. ');
                })
                .catch(error => { console.error(error); });
            }
        });
        updateGraphs();
    }

    function onDisconnected(event) {
        instancedDevices.remove(event.currentTarget)
        let index = instancedDevices.find(event.currentTarget)
        deleteTable(index)
        console.log('Device disconnected! ID: ' + event.currentTarget.id);
    }

    function insertTable(Device) {
        var tableRow = document.getElementById("table");
        var row = tableRow.insertRow(-1);
        var cell1 = row.insertCell(0);
        var cell2 = row.insertCell(1);
        var cell3 = row.insertCell(2);
        cell1.innerHTML = Device.type;
        cell2.innerHTML = Device.user;
        cell3.innerHTML = Device.device.id;
    }

    function deleteTable(rowIndex) {
        var table = document.getElementById("table");
        var rowCount = table.rows.length;
        table.deleteRow(rowIndex);
    }

    function addUser() {
        user = window.prompt("Username:")
        return user;
    }

    function ButtonColour(buttonID, colour, state, newText) {
        id = document.getElementById(buttonID);
        id.style.backgroundColor=colour;
        id.disabled=!state;
        id.firstChild.data = newText;
    }

    let options = {
     filters: [
    {
    // Filtering devices with company indentifier, showing only devices made by Polar
     manufacturerData: [{ companyIdentifier: 0x006b }]
     },
     {
     services: ["heart_rate"]
     }
     ],
     acceptAllDevices: false,
     optionalServices: [
     "0000180a-0000-1000-8000-00805f9b34fb",
     "0000180f-0000-1000-8000-00805f9b34fb",
     "fb005c80-02e7-f387-1cad-8acd2d8df0c8",
     "fb005c81-02e7-f387-1cad-8acd2d8df0c8", 
     "fb005c82-02e7-f387-1cad-8acd2d8df0c8"
     ]
     }
    
    function drawGraph(ECGdata) {
        console.log("ECGData: ");
        console.log(ECGdata);
        Plotly.newPlot('chart', [{ y: ECGdata,mode: 'lines',line: {color: '#80CAF6'}}],layout);
    }



    //initial plotly call for H-10 visualisation 
    function updateGraphs(ECGdata) {
    Plotly.newPlot('chart', [{ y: ECGdata,mode: 'lines',line: {color: 'red'}}],layout);

    console.log("UpdateGraph");

    var interval = setInterval(function() {
        var data_update = {
        y: [globalECGdata]
        }

        Plotly.update('chart', data_update, layout)

        if(globalECGdata.length > 365) {
            globalECGdata.splice(0, 365)
        }

        }, 1); 
    }


    //draws a second graph for a second H-10 
    function updateGraphs2(ECGdata) {
    Plotly.newPlot('chart0', [{ y: ECGdata,mode: 'lines',line: {color: '#80CAF6'}}],layout);

    console.log("UpdateGraph2");

    var interval = setInterval(function() {
        var data_update = {
        y: [globalECGdata1]
        }

        Plotly.update('chart0', data_update, layout)

        if(globalECGdata1.length > 365) {
            globalECGdata1.splice(0, 365)
        }
        //if(++cnt === 10) clearInterval(interval);

        }, 10); 
    }

    //transforms received data into ECG data for visualisation 
    function WordstoSignedInteger(words,BitsPerWord) {
        val=0;
        word_val=2**BitsPerWord;

        for(i=0;i<words.length;i+=1) {
            val+=words[i]*word_val**i;
        }

        bits=words.length*BitsPerWord;

        if(val>2**(bits-1)) {
            val=val-2**bits;
        }
        return val;
    }

    </script>

</html>
