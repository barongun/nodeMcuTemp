# nodeMcuTemp


```lua
temperature = 0
humidity = 0
flashId = node.flashid()
wifi_SSID = "yourWIFIID"
wifi_password = "yourWIFIPASSWORD"

wifi.setmode(wifi.STATION)
wifi.sta.config(wifi_SSID,wifi_password)
print(wifi.sta.getip())

function get_sensor_Data()
    dht=require("dht")
    status,temp,humi,temp_decimial,humi_decimial = dht.read(2)
    if( status == dht.OK ) then
        temperature = temp.."."..(math.abs(temp_decimial)/100)
        humidity = humi.."."..(math.abs(humi_decimial)/100)
        if(temp == 0 and temp_decimial<0) then
            temperature = "-"..temperature
        end
        --print("Temperature: "..temperature.." deg C")
        --print("Humidity: "..humidity.."%")
    elseif( status == dht.ERROR_CHECKSUM ) then          
        --print( "DHT Checksum error" )
        temperature=-1 --TEST
    elseif( status == dht.ERROR_TIMEOUT ) then
        --print( "DHT Time out" )
        temperature=-2
    end
    dht=nil
    package.loaded["dht"]=nil
end
function send_sensor_Data()
    time = rtctime.get()
    url = "https://<<FirebasDomain>>.firebaseio.com/users/"..flashId.."/tData.json"
    header = "Accept: application/json\r\nConnection: keep-alive\r\nAccept: */*\r\nContent-Type: application/json\r\nCache-Control: no-cache\r\n"
    body = "{\"temp\" : "..temperature..", \"humi\" : "..humidity.."}"
    
    http.post(url,header,body,
      function(code, data)
        if (code < 0) then
          print("HTTP request failed")
        else
          print(code, data)
        end
      end)
end

function loop() 
    if wifi.sta.status() == 5 then
        get_sensor_Data()
        send_sensor_Data()
    else 
        print("conn...")
    end    
end

tmr.alarm(0, 15000, 1, function() loop() end)
```
